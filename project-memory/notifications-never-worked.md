---
name: notifications-never-worked
description: "CONFIRMED IN PROD 2026-08-30: the client notification system has delivered ZERO notifications in its entire life. Three stacked defects. The banner on client home is dead code."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-30T12:42:21.454Z
---

**Verified against the live database, not reasoned about.** `plan_assigned` has
exactly **one** row and `is_demo = true` on its owner — seed data. `plan_cancelled`
has **zero**. Every notification that ever reached a real person was written
either by `service_role` (the invite route) or from inside a `SECURITY DEFINER`
function. **No client-side cross-user write has ever succeeded.**

With 25 active clients, every plan assigned since launch should have raised a
banner on that client's home screen. None ever did. Visible only by absence,
which is why nobody reported it. **Not a regression — it has never worked.**

## Three defects, stacked. Each one alone is enough.

1. **RLS denies the insert.** `notifications_own` (`velto_schema_refactored.sql:579`)
   is `FOR ALL USING (user_id = auth.uid())` with **no `WITH CHECK`**, so Postgres
   reuses USING for INSERT: *you may only notify yourself*. The trainer inserts
   `user_id = clientId` → 42501. Exact same bug class as
   [[rls-write-side-holes]], in a table that fix did not cover.
2. **`plan_cancelled` is not in the enum** (`velto_schema_refactored.sql:38`), so
   that insert is a `22P02` independent of RLS. And `NotificationType`
   (`notifications-api.ts:17`) *declares* it — TypeScript asserting a false fact
   about the schema is why it survived review, and why `NotificationBanner` has a
   branch that can never render.
3. **The reader only asks for one type** — `notifications-api.ts:48` does
   `.eq('type', 'plan_assigned')`. So `plan_completed` rows that the client's own
   auto-complete DOES successfully write are never shown to anyone.

## Why it was invisible for months

All four writers end in `.then(() => {})` (`progress-api.ts:1037, 1114, 1164,
1211`). supabase-js resolves with `{ data, error }` rather than rejecting, so the
error lands in a discarded object: **fails loudly at the database, silently in the
app.** `route.ts:760-762` already warns about exactly this in a comment — the
knowledge was in the repo, one feature over.

## ⛔ DELETED, not fixed — 2026-08-30 (`d09fba5`)

Owner's call once he saw what was actually there: *"momentan să nu avem niciun fel
de notificare... nu sunt ceva foarte important pentru lansare."* It was never a
feature — one dismissible banner, one page, one type, in an app with no email and
no push. Fixing it would have told a client something already visible one card
lower on the same screen.

Deleted: the banner, the reader, the feature module, all four broken writers.
**Kept:** the table, its rows, and the invite route's `service_role` write, which
works. A note sits where the `plan_assigned` insert was, giving the fix shape so
nobody re-adds the broken version.

Two things fell out with it: `planTitle` was threaded through three functions and
a store purely to fill a message body, and every client on the home screen was
polling the database every 30 seconds for a notification that could not arrive.

The real question, still unanswered and now in `launch-readiness-v1.md` §6: **what
should the client be told about, and through which channel?** A banner only finds
someone already looking. `roadmap-features-v1.md` feature 4 (messaging + feed)
cannot be built without answering it.

## 🔴 Fourth defect, found 2026-08-30 while running the seat-wall test in prod

**A different mechanism from the three above — not RLS, not the enum.** The
notification written by `accept_invite()` when a client claims the reusable link
is an `INSERT ... SELECT ... FROM profiles p WHERE p.id = v_uid`. If that
`profiles` row does not exist yet, the SELECT yields zero rows, the INSERT writes
nothing, and **no exception is raised** — the surrounding
`EXCEPTION WHEN OTHERS THEN NULL` is never even reached. The quietest possible
failure: not a swallowed error, a zero-row insert.

And it always is missing on the link path. A brand-new account runs
`signUp` → `signIn` → **claim** → `setProfileRole()`, so the profiles row is
written from the browser roughly 200 ms AFTER the claim.

Evidence, all four claims that have ever happened on the test trainer:

| Client | profiles row vs claim | notification |
|---|---|---|
| `+testc5` (link, 2026-08-30) | **+189 ms after** | ❌ |
| `+c3` (link, 2026-08-29) | **+197 ms after** | ❌ |
| `+c1` (link, 2026-08-29) | **+245 ms after** | ❌ |
| Iunia (email invite, 2026-08-29) | **−44 s before** | ✅ |

Iunia is the only one whose profile pre-existed the claim, and hers is the only
notification row in the table — timestamped to the millisecond of her claim. The
email path differs because `/api/clients/invite` creates the profiles row
**server-side** when the trainer types the address; the link path has no server
step.

⚠️ The owner's first reading was "email path notifies, link path doesn't". The
correlation is real but the cause is not the branch — the `CASE` that picks the
title works fine on both. Discriminating test, run 2026-08-30:
`activate_waiting_client()` uses the SAME `INSERT ... SELECT FROM profiles`
pattern for a client who arrived **via the link**, but reads the TRAINER's
profile, which always exists — and the notification **was written**. So the
variable is the row's existence, not the entry path.

**Fix:** `INSERT ... VALUES` with a scalar subquery for the name
(`COALESCE((SELECT display_name FROM profiles WHERE id = v_uid), 'Un nou client')`)
so the row is always written. One line, but it must be known — an
`INSERT ... SELECT` reads like it cannot fail, which is exactly why it survived.

⚠️ **The same window would be FATAL, not silent, on the single-use invite path:**
`invites.accepted_by` has an FK to `profiles`, so `accept_invite`'s
`UPDATE invites SET accepted_by = v_uid` would raise and roll back the entire
claim. It does not happen only because the email route creates the profile
server-side first. That ordering dependency is invisible from the code.

Full session context: [[test3-seat-wall-results]].

## Fix shape when they return (agreed, not built)

**SECURITY DEFINER RPC**, not a policy. Move the inserts inside
`assign_plan_to_client` / `unassign_plan_from_client` (`trainer_plan_rpcs.sql`) —
the precedent exists three times (`add_client_flow.sql:350`,
`accept_invite_seat_cap.sql:592` and `:747`). A permissive INSERT policy would let
a trainer write arbitrary `type`/`title`/`body` into a client's feed.

**Order matters:** add the enum value BEFORE moving the insert, or cancel turns
from a silent no-op into a hard failure inside the RPC. Then widen the reader,
then fix `NotificationType` to match the enum exactly.

## The pattern to hunt, beyond this

Writes that cannot fail loudly. Also found: the non-atomic `Promise.all` pairs in
`progress-api.ts:1088` and `:1139` (they manufacture the torn state that the dead
"Aprobă planul" button exists to repair), `use-client-detail.ts` never refetching
after a mutation, and `markNotificationAsRead` returning ok regardless of rows.
See [[backlog-trainer-writes-silent-noop]].

**UPDATE 2026-09-04 — the feature is gone.** `src/features/notifications/` was
deleted on 2026-08-30 in `d09fba5` ("delete a feature that never delivered a
single notification"). Nothing above is wrong as history, but do not go looking
for the code: there is none. For public copy this makes "Velto nu trimite
notificări" simply true, not a caveat — see the content rules in the
`velto-social` skill.
