---
name: backlog-verify-start-and-perf
description: "CONFIRM (live) at the next workout: A4 optimistic workout-start + the load-perf parallelization fixes. Code shipped + review-approved, not yet device-tested."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

**Live-verification TODO — user will test at their next workout (flagged 2026-07-21, testing ~next day).** Both changes are shipped and code-review-approved, but not yet tested on a real device.

1. **A4 — optimistic workout start** (`use-active-session.ts` + `client/workout/active/page.tsx`). Confirm:
   - Tapping **Start** shows the workout near-instantly (session is created in the background).
   - Logging sets fast still saves (revisit after finishing → data is there). Handlers await `sessionReadyRef`.
   - **Continuă** (resume) restores the previously-logged sets (merge via `mergeKeepingEdits`, no empty-flash overwrite).
   - Edge: fast log-last-set-then-finish may show a spurious "Nu am putut finaliza" error (pre-existing store `isSaving` guard, finding #4 — no data loss, tap again). If it's annoying in practice, fix with a dedicated `isCompleting` flag in `sessions-store.completeSession`.

2. **Load-perf parallelization** (`use-client-home.ts` #1, `use-client-detail.ts` #2, `plans-store.ts` `plans:usage` key #4). These are behavior-preserving refactors (review-approved) — verify as REGRESSION (everything still shows, just fewer serial round-trips):
   - `/client/home`: active-plan card, Finalizate, Greutate, Planuri finalizate, welcome card (no plan) — nothing blank/missing.
   - `/trainer/clients/[id]` + `/dashboard/clients/[id]`: active-plan card, KPIs, recent activity, completed plans, weight history all render.
   - Plan library: usage-count badges correct after create/clone/delete a plan (no stale badge — that was the #4 fix).

3. **A5 — in-workout overview drawer** (`workout-overview-drawer` + `client/workout/active/page.tsx`). Confirm: the list-icon button (top-bar right) opens a drawer listing all exercises with status (✓ done / › current / todo); tapping an exercise jumps to it WITHOUT losing logged sets (jump works forward and back). Design-review-approved (a11y sr-only status added).

Related deferred perf items: [[backlog-perf-followups]].
