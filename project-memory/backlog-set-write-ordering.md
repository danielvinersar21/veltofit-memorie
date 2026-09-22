---
name: backlog-set-write-ordering
description: "The 25kg→2kg bug: out-of-order set writes. Fixed + pushed to main 2026-08-07 (42dbab9). Hypothesis 2 (dropped keystroke in the controlled input) still UNTESTED — needs a live check at the gym."
metadata: 
  node_type: memory
  type: project
  originSessionId: 942ae820-f0a5-4369-aaa1-70945c3bdfd1
  modified: 2026-08-07T13:08:56.844Z
---

**Owner-reported 2026-08-07:** on the client side, weight logged as 25 kg sometimes came back as **2 kg** at the next workout — inconsistently. Separately, last workout's REPS were never shown.

**Fixed + merged + pushed to main as `42dbab9`. NOT device-tested.**

## The reps half was never a bug
`getPreviousExerciseWeights` only ever selected `actual_weight`, and filtered `.not('actual_weight','is',null)`. So reps were never surfaced (the reps placeholder is the PLANNED value from the plan), and unloaded exercises — plank, run — could not have a hint at all, because the filter removed them from the query. The data was in `actual_value` the whole time. Renamed to `getPreviousSetPerformance`, now returns `{ reps, weight, metric }`.

## The weight half: two independent ordering causes
1. **Concurrent debounced writes.** A set's saves were independent HTTP requests. Type "2", pause past the 800 ms debounce, type "5" → both "2" and "25" go out, last delivered wins. Fixed by chaining writes per `exerciseId:setNumber`, with each run re-reading `performanceRef` **when it starts** (so a queued save carries the freshest value) and coalescing not-yet-started runs (without that, a stalled connection stacks 12 s writes and completion waits on the whole backlog with the rating screen disabled — the same class of freeze this work came from).
2. **`withTimeout` never cancelled anything.** It only stopped waiting. A write that blew the 12 s ceiling stayed in flight and could commit later, restoring the older value. `timed()` in `sessions-api.ts` now attaches an `AbortController` and aborts on timeout. `postgrest-js` 2.90.1 exposes `.abortSignal(signal)` — verified in `dist/index.d.mts`. Caveat: abort closes the socket client-side; if the server already committed, it stays committed (Postgres usually cancels on connection close).

## Two reusable traps found here
- **A rename can silently kill cache invalidation.** `apiClient.invalidate` is a `startsWith` prefix match, so a stale prefix matches nothing and fails SILENTLY — the hints simply stopped being invalidated. Grep the prefix, not just the symbol, after renaming a cache key.
- **PostgREST `!inner` embeds pick the driving relation, and it may be the wrong one.** `workout_set_performance ... workout_sessions!inner(client_id, ...)` drives from the SET table, scanning the given exercise ids across EVERY client on the platform before filtering by client — exercises are a shared global library. On the workout-start hot path, on NANO. Fix pattern: two bounded steps — fetch the client's recent session ids first, then `.in('session_id', ids)`. See [[backlog-supabase-db-health]].

## STILL OPEN — hypothesis 2, untested
The fix addresses ordering. It does NOT address the other candidate cause: a **dropped keystroke in the controlled input**. Every keystroke re-renders the whole tracking screen with all sets; on a slow phone the second character can be swallowed before state propagates, so "2" is all that ever exists locally.

**How to tell them apart at the gym:** if the field shows "25" while training and 2 appears next time → ordering (fixed). If "2" appears on screen immediately as you type → it's the render path, and the fix does nothing for it.

Also still open from [[backlog-active-workout-trio]]: weight-only sets are silently discarded at completion.
