---
name: backlog-review-round-2026-08-20
description: "Open follow-ups surfaced by the 2026-08-20 agent review round that were deliberately NOT fixed then — e2e admin spec, role-write-once trigger, stat-tile focus quirk, for_debug_db re-dump, trainer-opens-invite edge."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6910664b-32db-48cf-aa94-0b89819b0c4d
  modified: 2026-08-20T08:54:31.704Z
---

From the first proper agent review round on `feat/measurements-bilateral` (2026-08-20:
`code-reviewer` + `design-reviewer`, then four fix agents). These were found, judged real, and
**deliberately deferred** — they are not oversights, and each has a stated reason.

1. **E2E spec: admin cross-host login.** `e2e/specs/auth/admin-cross-host.spec.ts` does not exist
   yet. The admin redirect-loop bug was **transient** — the old code did eventually reach the
   right page — so a final-URL assertion passes over it. The spec must attach a
   `page.on('framenavigated')` log *before* `goto` (Playwright emits it for History API
   navigations, which is what `router.replace('/role-selection')` was) and assert
   `/role-selection` never appears. Blocked on: `e2e/support/env.ts` has no admin credentials —
   needs `E2E_ADMIN_EMAIL`/`E2E_ADMIN_PASSWORD` against an `is_demo` admin account.
   Second test: client on dashboard host still moves to app host.

2. **Role write-once as a DB trigger.** `setProfileRole` now enforces it in three statements
   with Postgres-level exclusion, which is correct but lives in app code. The stronger form is
   `BEFORE UPDATE ... IF OLD.role IS NOT NULL AND NEW.role IS DISTINCT FROM OLD.role THEN RAISE`.
   Deferred because migrations here are applied BY HAND in Supabase, so shipping code that calls
   a not-yet-created object would break production at merge. Do the trigger separately, apply it
   first, then simplify the service if you want.

3. **Focus ring fails WCAG 2.2 SC 1.4.11 — being fixed, verify it landed.** The `focus-ring`
   mixin emitted `rgba(var(--primary), .2)`, which Sass cannot composite, so the declaration was
   invalid and dropped app-wide — ~98 call sites had NO keyboard focus indicator, including
   everything using `button-base`/`input-base`. Fixed with `color-mix()`. The owner then chose
   "solid outline + glow" to clear 3:1. See [[use-project-agents-by-default]] for why this went
   unnoticed for so long.

4. **`stat-tile` hover erases its focus ring.** `stat-tile/styles.module.scss` includes
   `focus-ring` then sets `&:hover { box-shadow: … }` at equal specificity, later in source, so a
   focused+hovered tile loses the indicator. Should self-resolve once the ring is a real
   `outline` (a separate property), but CONFIRM rather than assume.

5. **`docs/database/for_debug_db.sql` must be re-dumped** after `measurements_full_set.sql` is
   applied. It is a raw dump of live state, already stale in the other direction (missing
   `quads`/`calves` from 2026-05-29). Editing it by hand would make a dump assert a schema that
   is not live — re-dump, don't edit.

6. **Product decision, not a bug: a trainer who opens a client invite link.** Their role stays
   `trainer` (correct — role is write-once), but the Phase B invariant still routes them to
   `/client/home`, whose `profile.role !== 'client'` guard bails and hangs the page. **Not a
   regression** — the old guarded upsert produced the identical outcome. Changing it means
   changing the Phase B invariant, which is a product call. The information needed to handle it
   is already in hand: the cache refresh now runs unconditionally, so the stored role is readable
   right after as `useAuthStore.getState().profile?.role`.

7. **Nine more hand-written `--primary-20` focus rings** still decoration-only, outside the six
   components that were fixed: `components/ui/date-input` (needs the `focus-ring-styles` +
   `:focus-within` treatment, not a straight swap), six under `src/features/`
   (trainer-activity-feed, exercise-autocomplete, recent-sessions, workout-overview-drawer,
   set-presets, template-picker ×2), and two page-level SCSS files under `src/app/(app)/`.

8. **Product gap: a trainer can invite an admin's email.** `src/app/api/clients/invite/route.ts`
   returns 409 for `role === 'trainer'` but says nothing about `'admin'`, so the invite creates an
   active `trainer_clients` link. No longer a privilege problem — the admin's role now survives
   the write — but an odd state. Widening the guard changes an API response, so it is a product
   call.

**Three separate code paths wrote `profiles.role`**, each carrying the same promise in a comment
("Never overwrite an existing role") and none enforcing it in the database: `setProfileRole`
(no guard at all — the one the admin bug ran through), `auth-redirect.ts`'s invite path
(read-then-write), and the invite API route (read-then-write). All three are now closed, and
`setProfileRole` is the documented owner of the invariant. If a fourth appears, route it there or
use `.is('role', null)`.

Related: [[measurements-bilateral-status]], [[backlog-admin-dashboard]].
