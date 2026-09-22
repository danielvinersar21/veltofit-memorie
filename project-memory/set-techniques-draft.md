---
name: set-techniques-draft
description: "Supersets/dropsets/rest-pause — BOTH phases SHIPPED to main 2026-08-16 (phase 1 f67bd40, phase 2 f9d1a17) with the migration applied and 20 e2e tests. Remaining: the gym test, and 2 owner decisions."
metadata:
  node_type: memory
  type: project
  originSessionId: 942ae820-f0a5-4369-aaa1-70945c3bdfd1
  modified: 2026-08-15T18:07:30.805Z
---

Spec approved 2026-08-10 (`docs/plans/set-techniques-v1.md`, six owner decisions). **Both
phases are now built.**

## State 2026-08-15
- **Phase 1 (dropset + rest-pause) — MERGED to main**, `f67bd40`.
- **Phase 2 (superset) — MERGED to main 2026-08-16**, `f9d1a17` (branch
  `feat/set-techniques-superset`, 12 commits + a merge). Live in production via Vercel; still
  NOT gym-tested.
- **A conflict of my own making, worth not repeating:** commit `96ff8ac` swallowed
  `docs/plans/measurements-bilateral-v1.md`, a doc another session was writing at that moment.
  Cause: `git add -A` with a blocklist built at the start of the work — a new file appeared
  later and was not on it. Resolved by merging main and taking main's version wholesale. In a
  repo where someone else writes in parallel, add explicit paths, never `-A` minus exclusions.
- ✅ Migration APPLIED by the owner; verified against the live schema.
- ✅ **e2e-verified**, 6 new tests across 4 specs (suite now 20 passing, 1 pre-existing fixme):
  - `client/superset.spec.ts` — both members on one screen, each set logged against its own
    exercise, surviving a resume; plus finishing the workout (nothing in the suite had ever
    completed one, grouped or not).
  - `trainer/superset-builder.spec.ts` — link/unlink asserted against the STORED rows, not the
    screen: local state renders A1/A2 whether or not the write lands.
  - `trainer/superset-copy.spec.ts` — cloning a plan (the `clone_program` RPC, which lives in
    the DATABASE where no type-check or unit test can reach it) and copying a week. Both must
    RE-KEY: two copies sharing a key are one group to every reader.
  - `trainer/plan-wizard.spec.ts` — the whole build-a-plan flow end to end, plus the publish
    guard. The library spec's long-standing TODO about this is now covered.

## What phase 2 actually is
`group_key` + `group_position`, nullable, on `plan_day_exercises` AND `workout_exercises`.
One pure module holds the whole model — `src/features/plans/exercise-groups.ts`, 32 tests:
reading is a fold, editing returns rows to write. A plain exercise is deliberately a group of
one, which is what keeps the tracking screen to ONE render path.

UI is a single control, "superset with the one above". Chaining it builds a group of any size,
so n members needed no extra interface.

## Traps this surfaced (worth keeping)
- **FIVE copy paths would each flatten a superset silently**: addWorkoutToDay, duplicateWeek,
  copyWeekTo, cloneGlobalWorkout, and the `clone_program` RPC (it lists columns explicitly).
  All re-key rather than copy — two copies sharing a key would FUSE into one block, since
  grouping reads a contiguous run.
- **The workout builder wrote sort_order per bucket** (existing renumbered among themselves,
  new ones appended), so an exercise inserted mid-list jumped to the end on save. Invisible
  until supersets made contiguity load-bearing. Fixed.
- **`sortOrder: existing.length` collided with UNIQUE(day_id, sort_order)** after deleting a
  middle exercise — adding an exercise just failed. Pre-existing, fixed.
- The tracking screen persists an EXERCISE index, not a group index, even though it navigates
  by group: old sessions already hold one, and a trainer can regroup mid-session.

- **The group toggle had to become optimistic.** Its own spec caught it: the A1/A2 tags only
  appeared once the write returned, which under load is seconds — long enough for a trainer to
  tap twice and regroup what the first tap grouped.
- **Two silent failures on the same theme.** A failed exercise-add showed nothing at all (the
  mobile wizard renders no save indicator) and a second tap on "Copiază" started a SECOND
  clear-and-copy over the first. Both fixed; both were invisible in code and only appeared in a
  real run. Treat "the write is slow" as a UX state on this builder, not an edge case.

## Owner feedback after testing it by hand (all applied)
- **Letters count SUPERSETS, not every group.** Two plain exercises then a superset rendered
  "C1/C2" — the plain ones had silently spent A and B, though they carry no label. First
  superset of a day is now A wherever it sits. Both label paths (stored columns / link flags)
  had to move together.
- **The seam is drawn on the JOIN**, not as a rail on the lower card: facing corners flatten,
  the list gap closes, and a link badge sits on the seam (one per pair, so a triset gets two).
- **Step 2's forward action is "Vezi plan", disabled until every training day has exercises** —
  the condition publishing already enforced, moved to where the trainer can act on it, with the
  missing days named via the same `formatSlot` the publish error uses. Note it covers ALL weeks,
  so a 4-week plan stays closed until weeks 2-4 are filled; "Copiază din..." is the path there.

## Open
1. **Gym-test both phases — now in PRODUCTION, untested at a gym.** Phase 2 rewrote the
   navigation of the most fragile screen — see [[backlog-set-write-ordering]] and
   [[backlog-active-workout-trio]]. A real client (`d.vinersar21@gmail.com`) is on the Spartan
   Foundation plan, which has no supersets yet, so the grouped path only goes live for someone
   when a plan with groups is assigned.
2. **Owner decision — parallelise copy-week?** It clears the destination and re-inserts row by
   row, sequentially; per-day parallelism would cut it several-fold. Left alone deliberately: a
   write burst on the surface with the freeze history is the owner's call.
3. **Owner decision — is the all-weeks gate too strict** in practice? Relaxing it to the current
   week would make the button promise something publishing refuses.
4. Uncovered on purpose: report §5.4 labels (cosmetic), the workouts-library `test.fixme`, and
   `addWorkoutToDay` + `duplicateWeek` — both group-aware but reachable from NO UI at all.

The suite's latency flake is documented in [[e2e-suite-and-db-upgrade]] — recognise the
"Conexiune lentă" signature rather than debugging it as logic.
