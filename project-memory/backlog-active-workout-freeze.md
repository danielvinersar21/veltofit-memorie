---
name: backlog-active-workout-freeze
description: "Active-workout freeze (per-keystroke write storm). App fixes #1-#3 IMPLEMENTED + reviewed + gate-green 2026-07-29 (UNCOMMITTED, not device-tested). #4 (Pro/Micro) + #5 (realtime) still open."
metadata: 
  node_type: memory
  type: project
  originSessionId: 18f11716-6252-4229-81f7-dc1a12808b66
---

**UPDATE 2026-07-29 — fixes #1-#3 IMPLEMENTED (code done, gate green, NOT committed, NOT device-tested):**
- #1 Debounce: `use-active-session.ts` debounces set writes ~800ms (keyed `${exerciseId}:${setNumber}`) via `scheduleSetSave`; `onBlur` on both inputs flushes immediately (`handleSetBlur`→`flushSetSave`); `persistAllEnteredSets` re-persists every entered set at completion (idempotent). "12x100kg" set: ~5 writes → ~1-2.
- #2 Dropped redundant pre-flight SELECT in `recordSetPerformance` (`sessions-api.ts`) — relies on RLS `set_perf_client` + session_id FK, maps err 42501/23503 → "Session not found or access denied". code-reviewer CONFIRMED safe (RLS USING reused as WITH CHECK; keys off auth.uid() JWT not the passed clientId). Halves per-write query count.
- #3 Timeout/fail-fast: new `src/lib/api/timeout.ts` (`withTimeout`, 15s reads / 12s writes). apiClient wraps fetcher (clears pendingRequests on timeout → no dedup wedge). `timed()` wraps startSession/completeSession/updateSessionProgress/recordSetPerformance.
- code-reviewer: 0 critical, 1 WARNING FIXED — completion could silently lose a set (recordSet swallows errors). Now persistSet returns boolean, persistAllEnteredSets returns all-succeeded, handleCompleteWorkout BLOCKS completion + keeps storage + shows "conexiune slabă" error if any set unsaved.
- Files: src/lib/api/timeout.ts (new), src/lib/api/client.ts, src/features/sessions/services/sessions-api.ts, src/features/sessions/hooks/use-active-session.ts, src/app/(app)/client/workout/active/page.tsx. Gate: type-check + lint + 93 tests + build all green. UNCOMMITTED on main. NOT device-tested on real mobile — live-confirm the freeze is gone.
- STILL OPEN: #4 upgrade off NANO (Pro→Micro, also kills free-tier cold starts); #5 realtime/shorter-TTL so trainer sees completion without refresh. Minor pre-existing: no delete-on-clear for a set typed then cleared (out of scope).

---
ORIGINAL DIAGNOSIS (kept for reference):

**Symptom (reported 2026-07-29, prod test with 2 trainers on mobile data):** client logging a workout + trainer watching → BOTH froze simultaneously, unblocked after a while, then client could finish and trainer saw completion only after a manual refresh. User confirms it also freezes for them on the client side while tracking a workout. Reproducible.

**Root cause (diagnosed by reading the code):** the active-workout flow writes to the DB **on every keystroke**.
- `src/features/sessions/hooks/use-active-session.ts` — `handleSetValueChange` / `handleSetWeightChange` are wired to input `onChange` and call `recordSet` on every keystroke where the field is non-empty. NO debounce.
- `recordSet` → `recordSetPerformance` (`sessions-api.ts`) does **2 queries per call**: a SELECT ownership check on workout_sessions + the UPSERT on workout_set_performance.
- `src/lib/api/client.ts` apiClient has **no timeout and no retry**; writes bypass it and also have no timeout. A hung request stays pending indefinitely. Worse, the pending-dedup (client.ts:51-53) hands the SAME hung promise to every caller of a key → reads pile onto one stuck promise.
- So logging "100 reps @ 50kg" on one set = ~6 writes (1→10→100 + weight), a ~18-set workout ≈ ~150 write round-trips, many near-simultaneous, from mobile. On NANO each write = WAL + fsync → the burst exhausts the instance's disk-IO/CPU budget → whole DB throttles → BOTH users' in-flight requests stall at once → recovers when budget replenishes. The two users share nothing but the backend, which is why they froze in sync.
- Trainer sees completion only after refresh because the trainer's live view reads from cache with **TTL 2 min (no realtime)** — see `fetchTrainerClientSessions` / `fetchTrainerActivity` in sessions-store.ts.

**It's BOTH infra + app, compounding — but the trigger is the app design** (per-keystroke write storm). At 2 users, indexed queries shouldn't saturate even NANO; the write storm does.

**Fix plan (ordered by impact/cost), NOT yet applied:**
1. 🟢 FREE, biggest: **debounce set writes** — persist once on input `onBlur` or debounce ~700ms after typing stops, instead of every keystroke. Collapses the storm ~5-10×. The root fix. (hook: use-active-session.ts handlers.)
2. 🟢 FREE: **drop the redundant SELECT** in `recordSetPerformance` (RLS `set_perf_client` already enforces ownership) → halves per-write cost. (sessions-api.ts ~L248-255.)
3. 🟢 FREE: **add a request timeout + fail-fast** to apiClient + the mutation path so a stalled write errors quickly instead of freezing the UI; value is already in local state + sessionStorage (`velto:activeWorkout`), so retry is safe. Also mitigates the pending-dedup-stall (client.ts:51-53).
4. 🟡 PAID: **upgrade off NANO** (Pro plan → Micro+; Pro also removes free-tier auto-pause cold starts). Headroom so a burst doesn't throttle the whole instance. Secondary to #1 — do debounce first, Pro for headroom. Related sizing watch in [[backlog-supabase-db-health]].
5. 🔵 nice-to-have: realtime or shorter TTL for the trainer's live workout view so completion shows without a manual refresh.

Recommendation given to user: do #1-#3 (free, root cause) AND buy Pro (#4) for headroom. Pro alone only hides the symptom; debounce fixes it. User has not yet said to implement — offered.
