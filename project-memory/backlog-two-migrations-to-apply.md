---
name: backlog-two-migrations-to-apply
description: "The two written-but-unapplied SQL migrations, the code each needs first, and the DEPLOY ORDER they disagree on. Started 2026-08-28, paused mid-flight."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T11:55:07.651Z
---

Owner's call 2026-08-28: **do both, so they don't rot.** Work started and was
paused the same evening; the working tree was reverted to match deployed `main`
(`93611ef`).

## ✅ BOTH MIGRATIONS ARE LIVE — 2026-08-29. Only the UI is left.

Steps 1-3 of the four-step sequence are done and verified in production:

1. Route change pushed to `main` (`93611ef..ee61fb2`, Vercel green).
2. `trainer_private_notes.sql` applied — old column gone, new table with RLS on,
   4 trainer-only policies, 0 notes to migrate, note written+re-read in the app.
   See [[column-revoke-is-a-noop]].
3. `accept_invite_seat_cap.sql` applied — `consented_at` exists and **25 active
   links backfilled**, `activate_waiting_client` exists, `accept_invite` now
   reads the cap, the freeze-consent trigger is on, and the profiles policy sees
   consent. The seat cap is no longer decorative on the reusable-link path.

**REMAINING: step 4, the waiting-client UI** — the only code work left. Details
in section 2 below. Nothing is broken while it is missing, because all 5 trainers
are `billing_exempt` (limit = 2147483647), so no one can reach the waiting state.

**Both migrations were audited before running and both needed fixes first.**
Worth repeating the pattern next time: `trainer_private_notes.sql` was missing a
pre-flight guard for the composite FK's backing index (a miss would have killed
the whole transaction with a bare 42830) and its deploy-order section was
backwards. `accept_invite_seat_cap.sql` **was not wrapped in a transaction at
all** — an interrupted run between `DROP FUNCTION accept_invite` and its
`CREATE` would have killed invite claiming platform-wide, and
`set-password/page.tsx:109-112` swallows that error into a `console.warn`, so it
would have failed silently. A read-only pre-flight query per migration (20 checks
for the second) turned every "the file assumes X" into a live yes/no before
pasting. Do this for every future migration.

## STEP 1 CODE — how it was built, 2026-08-29

Branch **`fix/trainer-private-notes`**, two commits (`ac2d59a` the route change,
`ee61fb2` the review fixes). Gate green (lint, tsc, 206 tests, build), reviewed
by `code-reviewer`, fixes written by `backend-dev`. **NOT pushed, NOT merged,
nothing applied to the database.** Next physical action: push → Vercel → then
run the SQL.

Three things that came out of the review and are worth keeping:

1. 🔴 **A pre-flight the migration was missing.** Its composite FK needs a
   NON-PARTIAL unique on exactly `trainer_clients (trainer_id, client_id)`.
   `uq_trainer_clients_one_active_trainer` looks like it and is not — partial
   (`WHERE status='active'`) and on `client_id` alone. Missing → the whole
   transaction dies with a bare **42830**. A `DO $preflight$` guard is now
   PASUL 0 and stops with the exact `ALTER TABLE` to run. The index probably
   exists (`velto_schema_refactored.sql:82`; `accept_invite()` upserts
   `ON CONFLICT (trainer_id, client_id)`), but that was never verified live.
2. **The SQL file used to prescribe the OPPOSITE order** ("SQL întâi, deploy
   după") and `code-reviewer` sided with the file against this memory. Resolved
   in favour of code-first and the file's header now says so with the argument:
   SQL-first returns **500** on "add client WITH a note" and, in CASE B, orphans
   an auth account `inviteUserByEmail` already created — the retry lands on
   CASE A and 500s identically, looping until the note field is cleared.
   Code-first only loses the typed note, silently and recoverably. Both windows
   are minutes and hit one flow. **Don't let anyone flip it back.**
3. **A `COMMENT ON COLUMN` that lives IN THE DATABASE** (`client_profiles.
   health_notes`, from `add_client_flow.sql:93`) pointed at the column being
   dropped. Editing the .sql file does not reach production — PASUL 9 now
   rewrites it. Watch for this class: DB-stored comments are invisible to grep
   over `src/`.

## ⚠️ The thing that is easy to get wrong: the two want OPPOSITE orders

- `trainer_private_notes.sql` DROPs a column the deployed code still writes
  (`route.ts` CASE A + CASE B) → **code first, then SQL.**
- `accept_invite_seat_cap.sql` ADDs `consented_at`, which the new UI must SELECT
  → **SQL first, then code.**

Sequence that satisfies both, four steps:
1. deploy A — the route change below (safe before AND after its SQL: the note
   write is non-fatal, so during the window a typed note is lost, nothing 500s);
2. run `trainer_private_notes.sql`;
3. run `accept_invite_seat_cap.sql`;
4. deploy B — the waiting-client UI.

## 1. trainer_private_notes — SMALL, do it first

Fixes a promise the UI makes twice ("Doar tu vezi această notă") that is false:
RLS filters rows, not columns, and the client legitimately sees their own link
row. Latent only — 0 of 25 links had a note on 2026-08-22.

Code needed (was ~80% written when we stopped, then reverted):
- `src/app/api/clients/invite/route.ts` — drop
  `...(trainerNotes ? { trainer_notes: trainerNotes } : {})` from BOTH writes
  (CASE A upsert ~line 308, CASE B insert ~line 504) and instead write the note
  AFTER the link (an FK ties it to the link) via a small `writePrivateNote()`
  helper using the admin client: upsert into
  `trainer_private_notes (trainer_id, client_id, note)`, `onConflict:
  'trainer_id,client_id'`, **non-fatal** (console.error + continue, like the
  starting-measurement insert at ~line 493). service_role, NOT the RPC —
  `auth.uid()` is NULL under service_role.
- Rewrite three now-false comments: `route.ts:18`, `clients-api.ts:277`,
  `clients-api.ts:534-537`.
- `get_trainer_note` / `set_trainer_note` keep their contract bit for bit, so
  nothing else in the app changes.

## 2. accept_invite_seat_cap — BIGGER, needs new UI

Today a trainer at their cap shares the reusable link and clients go active
anyway; the cap is decorative on that path. After the SQL such a claim lands as
**pending WITH consent** and the trainer activates it when a seat frees up.

What the SQL gives the client code:
- `trainer_clients.consented_at` (timestamptz, frozen by a trigger)
- `activate_waiting_client(p_client_id uuid)` → void, `authenticated`, raises
  ready-to-display Romanian (not yours / not pending / no consent / already has
  another trainer / cap reached). Refuses pending WITHOUT consent — those are
  trainer-created accounts and must never be activatable.
- `profiles_trainer_sees_clients` widened to `consented_at IS NOT NULL`, so the
  waiting person has a NAME in the list. Identity only; measurements, health
  notes, sessions and progress stay gated on `active`.

Code needed (4-layer, mirror `archiveClient`/`reactivateClient`):
- `clients-api.ts` — `ClientState` gains `'waiting'`; select `consented_at`;
  map `pending` → `consented_at ? 'waiting' : 'invited'`; add
  `activateWaitingClient()` calling the RPC (send only `p_client_id`).
- `client-state.ts` — badge + a filter option for the new state.
- store action with `apiClient.invalidate('clients:')`, hook, then an
  "Activează" button wherever archive/reactivate already live: dashboard
  `clients/page.tsx` and the mobile trainer list.

**Known gap left open on purpose** (separate ticket): invite CASE A still
upserts an ACTIVE link from a typed email via service_role — no consent, no cap.
It is also the way a waiting client can be flipped active over the cap.

## Why nobody will notice if this breaks

All 5 trainer accounts are `billing_exempt`, so the seat-cap branch is
unreachable in your own testing for weeks. Expire a test account's trial in the
DB to exercise it.
