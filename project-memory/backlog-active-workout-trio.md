---
name: backlog-active-workout-trio
description: "fix/active-workout-trio (dad3d35) — 3 active-workout fixes, VERIFIED IN MAIN AND ON GITHUB 2026-09-24. The old 'NOT merged, NOT pushed' line was stale and caused a false alarm. One pre-existing bug still open: weight-only sets are discarded."
metadata: 
  node_type: memory
  type: project
  originSessionId: 942ae820-f0a5-4369-aaa1-70945c3bdfd1
  modified: 2026-08-07T11:18:01.534Z
---

**Committed 2026-08-07 as `dad3d35` on `fix/active-workout-trio`. ✅ MERGED INTO MAIN AND PUSHED — verified 2026-09-24 with `git merge-base --is-ancestor dad3d35 main` (and against `origin/main`). In production since.**

> ⚠️ This note previously read "NOT merged, NOT pushed" and was never corrected. On 2026-09-24 a status report repeated it and told the owner he had seven weeks of work sitting on one disk. He did not. **The only branch not in main is `feat/exercise-media`.** Before ever claiming a branch is unmerged, run `git merge-base --is-ancestor <sha> main` or `git branch --no-merged main` — do not trust this file's status lines without it.

The three fixes are live; the branch ref just still exists locally. Device-testing was never done.

Three fixes to the client active-workout screen (`page.tsx` + `use-active-session.ts`):
1. Weight field for loaded non-reps sets (carries, loaded holds, timed machine sets).
2. Resume position restored independently of logged sets.
3. `saving` stays true after successful completion so the button doesn't flip back before routing.

**Why "loaded" is keyed off `exercise.category`, not equipment — the reusable insight:**
The first attempt derived it from the `exercise_equipment` join (`free_weights`/`machines`/`cables`). That is a **dead table for app-created data**: `createCustomExercise` (`exercises-api.ts`) inserts only the `exercises` row + one `exercise_muscles` row, and `ExerciseInput` has no equipment field. The ONLY writer is the global seed (`velto_exercises_seed.sql`, ~55 global rows). So every trainer-made exercise has `equipment: []` forever — including Farmer Carry from the spartan seed, the exact case the fix was for. Now uses `category ∈ {strength, olympic}`, which rides along on the plan payload (`exercise:exercises(*)`). **Don't build features off `exercise_equipment` until something writes it.**

**Two guards added because the naive fix #2 regressed the optimistic-start path:**
- `userNavigatedRef` — set in `goToExercise`. The workout renders optimistically while `startSession` is in flight; a fresh session returns `current_exercise_index = 0`, so restoring unconditionally yanked a fast client back to exercise 1. Sets get this protection via `mergeKeepingEdits`; the index had no equivalent.
- Range clamp — a trainer can remove exercises from a plan mid-session (`removeDayExercise` has no assigned/in-progress guard). An out-of-range index made `exercises[i]` undefined → blank screen, since `isLoading` is false, `error` is null and `exercises.length !== 0` so no empty state fires.

**Gate green** (lint, type-check, 93 tests, build) both before and after the fixes.

**STILL OPEN — re-verified against `main` on 2026-09-24, the code is unchanged:**
Weight-only sets are silently discarded at completion. `persistAllEnteredSets` (`use-active-session.ts:649`) filters on `p.actualValue.trim()`; `persistSetNow` (`:522`) early-returns `true` when reps are empty. The service requires a non-empty `actual_value`. A client who types 24 kg on a carry but forgets the metres gets the "Seturi incomplete" warning, and if they continue, `clearWorkoutState()` wipes the sessionStorage copy too — the 24 kg exists nowhere. Fix needs a product call: either persist weight with a placeholder value, or block completion on weight-only sets.

Also noted, not acted on: trainer plan-report pages print `actual_value` only, never `actual_weight` (the session-detail drawer does show it) — so the trainer can't see the weight data this fix produces more of.

Unverified: whether live RLS on `exercises` lets a CLIENT read a trainer-custom exercise row (schema files say `is_global = true OR created_by = auth.uid()`, which would deny). The app works today, so it was probably widened — but nobody has run `select polname, pg_get_expr(polqual, polrelid) from pg_policy where polrelid = 'exercises'::regclass;` to confirm. The category-based fix doesn't depend on it either way.

Live-confirm alongside [[backlog-verify-start-and-perf]] and [[backlog-active-workout-freeze]] at the next real workout.
