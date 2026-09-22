---
name: backlog-client-active-workout-flow
description: "Backlog — client active-workout flow: slow start on mobile (the remaining open item; resume-CTA and workout-preview are DONE)."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

Backlog items for the **client active-workout flow** (`src/app/(app)/client/workout/active/` + `src/features/sessions/`). Flagged by user 2026-07-20.

1. **Slow start on mobile.** (OPEN) Tapping "Start" on mobile takes too long before the workout opens. Investigate the start path (session create + plan fetch) — likely a slow/serial network round-trip (create `workout_sessions` row → fetch plan/day exercises) or missing optimistic UI. Check `use-active-session.ts` start logic and whether it blocks on the session insert before rendering. Consider optimistic navigation + skeleton, and/or prefetch the day's exercises.

2. ~~Resume instead of Start after reopening the app.~~ **DONE (commit a2cc598).** Home + plan detect an in-progress session (`fetchActiveSession`) and relabel the CTA to "Continuă"; `startSession`/`completeSession` invalidate `sessions:active:`.

3. ~~"Vezi" shows only that workout, not the whole plan.~~ **DONE (commit a2cc598).** Resolved as the opposite of the original note: the home/next-workout "Vezi" now opens a read-only `WorkoutDetailDrawer` for the single current/next workout; the plan page's bottom "Vezi planul" still opens the whole-plan drawer.

Related: [[backlog-view-workout-mid-session]] (viewing the workout mid-session is a distinct but adjacent need).
