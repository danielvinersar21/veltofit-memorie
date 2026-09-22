---
name: backlog-perf-followups
description: "Backlog — remaining load-performance follow-ups from the 2026-07 audit (over-fetch on plan-report, duplicate measurement read on home)."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

Leftover items from the load-performance audit (2026-07-21). The audit found the codebase CLEAN overall (no N+1, stable effect deps, apiClient caching consistent, no invalidation storms). The two real waterfalls (`use-client-home`, `use-client-detail`) and the `plans:usage` cache key were fixed. These two are lower-impact and deferred:

1. ~~**Plan-report pages over-fetch.**~~ **DONE (2026-07-23).** Extracted `useClientActivePlan(clientId)` (`features/progress/hooks/use-client-active-plan.ts`) — reads only `getClientDetail` + (conditionally) `getActivePlanInfo`, reusing the same cache keys (`client:detail:` / `progress:`) so it still dedupes from the client-detail page. Both plan-report pages (app + dashboard) point at it now instead of `useClientDetail` (was ~7 payloads → 2).

2. ~~**Duplicate measurement read on client home.**~~ **SKIPPED — not safe.** The "duplicate" isn't truly redundant: `getMeasurements` (the list `<MeasurementsSection>` loads) is `.limit(10)` (last 10), while `getWeightSummary` queries the ALL-TIME earliest + latest weight for the "kg de la start" delta. Deriving the summary from the last-10 list would give a WRONG start weight (oldest of the last 10, not the true first). The two reads serve genuinely different scopes, so they stay separate.

Note for context: several intentionally-uncached direct reads are correct as-is (getPlanFull in drawers/plan-builders wants fresh structure after edits) — do NOT "fix" those.
