---
name: client-home-perf-findings
description: "Client-home load perf: what was fixed (b8fe637 + db7b19e), what was MEASURED and disproved (prefetch is not caused by remounts). ProtectedRoute unmount is FIXED; the stale-deps half and the AppLayout role default may still be open — verify before quoting."
metadata: 
  node_type: memory
  type: project
  originSessionId: 942ae820-f0a5-4369-aaa1-70945c3bdfd1
  modified: 2026-08-07T14:46:54.856Z
---

Investigated 2026-08-07 from owner's prod network waterfall (`/client/home`, 30 requests, 11.45 s finish).

## Fixed and shipped (`b8fe637`, on main)
`getActivePlanInfo` fetched the whole plan tree (~30 kB) to answer "which days are real training days", and a second serial `plan_days` query for "next session". Now one bounded query — `plan_weeks?select=week_number,plan_days(id,day_number,title,is_rest_day,plan_day_exercises(count))` — answers both. Home's `usePlanDetail` (drawer only) was also deferred until after the home data lands. Prod transfer 85.1 kB → 47.1 kB.

**Verified against real data** (logged in as the test client, dev build): `dayRows` = 28 days, all with exercises, counts 6/1/5. So the real computation runs, not the fallback.

**The trap worth remembering:** `computeTrainingDayProgress` falls back to the nominal `days_per_week × weeks_count` grid when no day has exercises — and that number frequently EQUALS the real one (a 4×7 plan with no rest days is 28 either way; Alex M.'s 4×3 is 12 either way). A silent fallback is therefore invisible in the UI. Added a `console.warn` for "days exist, none has exercises".

## MEASURED and DISPROVED — don't re-derive this
The `?_rsc=` prefetch churn (up to 7 per route, 24+ per load) is **NOT** caused by the app remounting. Measured with a mount/unmount probe in TabBar on a LOCAL PRODUCTION BUILD (`next start`, port 3100 — dev is useless here: Next disables Link prefetch in dev and StrictMode double-invokes effects):

```
[PROBE] TabBar MOUNT     ← exactly once, zero unmounts, alongside 26 prefetch requests
```

Hashes `1yixb` and `j79z8` appear across every route = full prefetch passes by Next's router. It is Next.js router behaviour, not app re-rendering. The only lever is `prefetch={false}` on the tab links. **A plausible remount theory was wrong here — measure before acting on it.**

## RESOLVED 2026-08-07 (`db7b19e`, `b2aeb28`)
Both items below are FIXED and pushed, plus the `workout_sessions` consolidation (3 reads → 2). Review-driven follow-ons in the same commit: TopNav name no longer blinks on background refresh; `/role-selection` joins `hidesChrome` (no role → no AddClientDrawer → the trainer "Adaugă client" button was dead); `signOut` now bumps an epoch + `apiClient.clearPending()`, because keeping the tree mounted made the button tappable mid-refresh and an in-flight `fetchUser` could write the signed-out identity back (name/avatar residue on a shared device).

**STILL OPEN — the ProtectedRoute fix only half-delivers.** `fetchUser` assigns a brand-new `User` object on every success (`auth-store.ts:144`), so any hook with `user` in its deps still re-runs on every auth refresh: `use-client-home.ts:131`, `use-active-session.ts:417`, `use-plan-detail.ts:46`, `use-workouts.ts:57`, `use-clients.ts:23`, `use-plans.ts:53`, `use-invites.ts:51`, `use-trainer-activity.ts:50`, `use-client-progress.ts:39`. Component state/timers/typed values now survive, but the refetches do not stop. Fix = switch those deps to `user?.id` (`use-dashboard-counts.ts:79` already does).

## Original diagnosis of those two (kept for context)
1. ~~**`ProtectedRoute` unmounts the whole app subtree whenever `isLoading` is true**~~ ✅ **FIXED in `db7b19e`** ("stop guessing the role, stop unmounting the app to refresh auth"), verified in the source 2026-09-02: the guard is now `if (isLoading && !user)`, with the reasoning written into the component at `src/components/layout/protected-route/index.tsx:48-59`. The original text follows, because the *shape* of the trap is still worth recognising elsewhere: (`if (isLoading) return loadingFallback`), and `auth-store.invalidate()` → `fetchUser()` → `set({isLoading:true})`. So a new-plan notification (`use-notifications.ts:53`) or a plan assignment (`use-plan-detail.ts:57`) tears down TopNav + TabBar + the page and re-runs every hook. Does NOT happen on a normal load. There is already a warning comment about a related trap in `auth-form/index.tsx:92-95`.
2. **`AppLayout:42` defaults `role` to `'trainer'` when profile is null.** Today `ProtectedRoute` never renders children without a profile, which hides it. **Fix this BEFORE touching ProtectedRoute** — otherwise a client can flash the trainer tab bar. Owner explicitly asked about this risk.

## Not duplicates, don't "fix" them
`profiles` ×2 = own + trainer's. `notifications` ×2 = initial + the 30 s poll. `client_measurements` ×3 = list + `getWeightSummary`'s two deliberate `limit(1)` probes (newest/oldest weight). `client_plan_progress` ×2 = active vs completed-plans list. The one real redundancy is **`workout_sessions` ×3** — `getActivePlanInfo` (plan-scoped) and `getWorkoutStatsForClient` (all completed) overlap and could be one query.

Notifications polling does NOT slow the load (one 0.6 kB parallel request) and must not simply be deleted: its `invalidateAuth()` is how a client discovers a newly assigned plan. Cheapest reduction is raising the interval 30 s → 2 min (auth TTL is 2 min anyway, `auth-store.ts:128`).

Test client for prod + local: `d.vinersar21@gmail.com`. "Alex M." on prod is also a test client. Same Supabase project (`wcpzdurualepyjjrracm`) for local and prod, so a query verified locally holds on prod.
