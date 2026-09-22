---
name: backlog-exposure-deferred
description: "Two exposure findings from the 2026-08-29 blind inventory, deferred by the owner the same day: owner's personal email committed in a .sql comment, and 37 unguarded design-system pages shipping to production."
metadata: 
  node_type: memory
  type: project
  originSessionId: dec69265-dcee-4170-8138-beb9b84c8c1d
  modified: 2026-08-29T18:36:05.072Z
---

Found by the blind-inventory agents on **2026-08-29**, confirmed by reading files, and
**deferred by the owner the same day**. Neither is an oversight — both were seen and parked.
Don't re-report them as discoveries; do raise them before real signups.

**1. Owner's personal email is committed.** `docs/database/admin_dashboard.sql:388` carries it
inside a comment (`-- SELECT id FROM auth.users WHERE lower(email) = lower('…')`). One-line fix
whenever it's picked up: replace with a placeholder. The repo is otherwise clean of secrets —
zero JWTs, zero service-role keys, no `.env` file ever committed, verified across all history.

**2. 37 `design-system/*` pages have no auth guard.** `src/app/design-system/layout.tsx` has no
`ProtectedRoute` and no `useAuthStore` check, and `src/middleware.ts:108-110` *deliberately*
excludes the path from the matcher — the comment says "allow on all subdomains for development".
They are in the production build (`.next/server/app/design-system*`). They expose component
demos and token tables, not customer data, which is why it was parked rather than fixed.

**Why it matters more later than now:** the app is pre-launch with no real users. Both become
real the day strangers can sign up — see [[launch-readiness-plan]] and [[backlog-launch-blockers]].

**How to apply:** when either is picked up, do it as its own change, not folded into feature
work. For #2 the cheaper option is excluding the route group from the production build rather
than adding a guard, since the pages are dev tooling and nothing links to them from the app.

Related: [[qa-gate-blind-spots]] — the quality gate catches neither of these, and never will;
they were only found because an agent was asked to inventory exposure specifically.
