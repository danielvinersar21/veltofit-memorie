---
name: backlog-verify-last-weight-hint
description: "CONFIRM (live-verify) the 'last weight used' hint works in a real repeated-exercise workout — code shipped, not yet tested live."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

**Live-verification TODO (not a bug).** The "last weight used" feature is built, gate-verified (type-check/lint/build) and code-reviewed clean — but NOT yet tested against real data, because it only shows once the same exercise recurs in a later completed session (user couldn't test before the next relevant workout). Flagged by user 2026-07-20.

**What to confirm when testable:** in the active workout, on a reps-metric exercise, the weight field shows `Ultima: X kg` (+ ghost placeholder) equal to the weight logged for that same exercise+set in the client's most recent COMPLETED session. Fastest path on the Spartan test plan: complete **Lower A (Wed)** logging a weight on **Romanian Deadlift**, then reach **Lower B (Sun)** which also has RDL → the hint should appear.

**Where it lives:** `getPreviousExerciseWeights` (`sessions-api.ts`), `fetchPreviousWeights` (`sessions-store.ts`, cache `sessions:prevweights:*`, invalidated on completeSession), `previousWeights` in `use-active-session.ts`, rendered in `client/workout/active/page.tsx` via the `.setPrev` class. Shipped in a commit on `main` (last-weight feature).

If it doesn't show: check that the prior session is `status='completed'`, that the exercise_id matches (same library row), and that `actual_weight` was persisted (only stored when weight > 0 alongside logged reps).

Related: [[backlog-workout-reporting-history]] (this is item 3 of that cluster, now implemented pending live confirmation).
