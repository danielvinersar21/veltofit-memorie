---
name: backlog-view-workout-mid-session
description: Backlog feature — let a client view the full workout after starting an active session.
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

**Backlog feature (client side).** After a client starts a workout (active session), give them a way to view the whole workout — the full exercise/set list / overview — even once the session is already in progress.

**Why:** Currently, once the active session starts (`(app)/client/workout/active`), the client is in the set-by-set logging flow and has no way to step back and see the full workout at a glance (what's coming, total exercises, etc.). They asked for a way to review the workout mid-session without losing progress.

**How to apply:** When picking this up, look at `src/app/(app)/client/workout/active/page.tsx` and `src/features/sessions/` (active-session hook/store). Likely an overview drawer/modal or a toggle between "logging" view and "overview" view of the session's exercises — must not reset or lose logged sets. Confirm exact UX with the user before building (drawer vs full screen, read-only vs jump-to-exercise).

Flagged by user 2026-07-20.
