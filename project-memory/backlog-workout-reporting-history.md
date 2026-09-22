---
name: backlog-workout-reporting-history
description: "Backlog — post-workout trainer report, client recent-workout detail view, and 'last weight used' hint per exercise."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

Backlog cluster: **workout reporting & history**. Flagged by user 2026-07-20. Feasibility checked against current code (see notes per item — much of the data/plumbing already exists).

1. ~~**Trainer gets a report after each client workout.**~~ **DONE (feed approach chosen over push).** Built: a "recent client activity" feed on trainer home (app) + dashboard via `getTrainerRecentActivity` (`sessions-api.ts`), `useTrainerActivity`, `TrainerActivityFeed`; tapping a workout opens the reused `SessionDetailDrawer` with per-set detail. Also made the recent-activity rows on both client-detail pages (`(app)/trainer/clients/[id]` + `(dashboard)/dashboard/clients/[id]`) tappable → same drawer, and moved the whole-plan "Vezi raport" link into the active-plan card. No DB trigger — reads existing data via RLS (`sessions_trainer`/`set_perf_trainer`). A push notification was explicitly NOT chosen (would need a DB trigger + a trainer notification surface that doesn't exist).

2. **Client can view recent / completed workouts (with detail).** Let the client review what they did in past sessions.
   - *Already exists:* the "Antrenamente recente" section (recentSessions) currently lives on `/client/progress`. `fetchSessionDetail` returns full per-set detail.
   - ~~*Gap 2a — MOVE placement:*~~ **DONE (commit 429561d).** "Antrenamente recente" moved from `/client/progress` to `/client/plan` (below "Vezi planul"). Extracted a reusable `RecentSessions` component + light `useRecentSessions` hook.
   - ~~*Gap 2b — detail view:*~~ **DONE (commit 459029a).** Tapping a recent session on /client/plan opens `SessionDetailDrawer` (`src/features/sessions/components/session-detail-drawer/`) with exercises + logged reps/weight per set. `RecentSessions` gained an `onSelectSession` prop; store got a dedicated `isLoadingDetail` flag.

3. **Show the last weight used for an exercise, in the next session.** When the client logs e.g. bench press at 20kg, the next time bench press comes up the field should hint the previous weight ("ultima: 20 kg").
   - *Already exists:* `workout_set_performance.actual_weight` (per-set kg) is persisted; the active session already reads `dbPerf.actual_weight` but only for the CURRENT session. There is an unused `.setPrev` class in `client/workout/active/styles.module.scss` — likely a placeholder for exactly this.
   - *Gap:* on entering the active session, look up the most recent `actual_weight` per `exercise_id` from the client's PRIOR completed sessions (a query in `sessions-api`, e.g. latest set performance per exercise), and surface it as a non-editable hint next to each set. High value, clearly feasible.

Related: [[backlog-view-workout-mid-session]], [[backlog-client-active-workout-flow]].
