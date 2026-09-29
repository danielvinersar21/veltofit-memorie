---
name: demo-data-is-a-test-surface
description: Demo/seed data is load-bearing — e2e suites and marketing screenshots both ride it, so a gap there makes a freshly shipped feature look broken to the only audiences that see it.
metadata:
  type: project
---

Treat demo/seed data as a **test surface and a shop window**, not decoration. When a
feature ships, ask what it looks like *on the demo data*, because that is usually the
only data anyone outside the team ever sees it with.

**Why:** on this project (2026-09-28) a read-only "open a finished workout" feature went
live the same day it was discovered that none of the ~40 finished demo sessions had any
logged sets, and two of three assigned demo plans had completely empty weeks. So the
newest feature opened an empty page — on the exact slice that the Playwright suite runs
against in production (isolated by an `is_demo` flag) and that every marketing
screenshot is taken from. Separately, a plural bug ("1 CLIENȚI") that had been live for
months was found *only* because that counter was about to appear on the landing page —
no user had ever reported it.

**How to apply:**
- Two questions after any read/detail feature ships: does the demo account have rows
  that make it non-empty, and do the e2e specs or captures depend on those rows?
- **Screenshot and demo-data work finds real product bugs.** When the user reports a bug
  found while making captures, log it as a product bug with its own priority, not as a
  marketing chore — and note the discovery route, because "nobody reported it" tells you
  how thin the real usage is.
- Watch for the trap where **the fix is blocked from where the problem is**: a seed file
  may be deliberately local-only (production holds real customers, and invented rows
  would pollute real counts). Then the gap is an open *decision* for the owner, not a
  task someone forgot. Say so plainly instead of listing it as a to-do.
- A screenshot is a claim about the product at a point in time. Renames, nav changes and
  copy fixes silently falsify old captures — and the `alt` text beside them, which is
  the copy most likely to keep describing the previous version. See
  [[legal-copy-drifts-with-feature-flags]] for the same failure mode in legal text.
