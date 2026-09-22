---
name: repo-is-blind-to-real-world-events
description: Git and files cannot tell you a user signed up, paid, or churned — once a product is live, a status report built only from the repo will silently omit the most important facts.
metadata:
  type: project
---

Files and refs are authoritative for **what was built**. They are blind to **what
happened**. Once a product takes real traffic, the two diverge and the second one
matters more.

**Why:** on a project I reported on, two real users signed up nine days apart and
the owner found out a week later, by accident, looking at a database table. Nothing
in the repo changed when they arrived. My whole verification method — refs, reflog,
reading source — could not have surfaced it, and my state file went a week describing
a product with no users while it had users. The state file was not wrong about the
code; it was silent about reality, which reads identically.

**How to apply:**
- The moment a project crosses from pre-launch to live, add a standing question to
  every status pass: *has anyone used it since we last spoke, and how would we know?*
  Ask the owner outright — this is one of the few facts only they can supply.
- Treat **"is there any alerting to the owner?"** as a first-class backlog item, not
  ops polish. A product that cannot tell its owner a customer arrived will keep
  producing this failure. Note the usual design trap: signups happen in the database,
  so an alert placed in application code misses every path that bypasses it, and a
  DB trigger cannot send mail on its own — it needs an outbound hop.
- Some facts are **only** knowable in a vendor dashboard or the live database:
  applied migrations, webhook event subscriptions, subscription/trial state, payment
  settings. When those are load-bearing, say "this cannot be verified from files" and
  name where to look, rather than inferring from the SQL sitting in the repo.
- **Derive the dates nobody wrote down — but check the premise first.** A 30-day trial
  that started on a known date ends on a knowable one; a first charge scheduled in a
  vendor dashboard is a real deadline. Surface these as approaching dates, flagged as
  derived and needing confirmation.
  ⚠️ **I got this wrong once and it cost trust:** I derived "their trials end in
  October" from two signup dates, when the pricing model had been changed weeks
  earlier so the free tier has **no trial at all** — and the owner had already said
  so. The owner's own notes recorded "an agent re-derived it wrongly". Before
  reporting a derived deadline, confirm the *rule* it rests on (which tier, does that
  tier have a trial, who grants it) against the current pricing decision and the
  user's own memory notes, not against the general product description. A false
  deadline is worse than a missing one: it sends the owner chasing nothing.
- Watch for a document contradicting *itself* across time: a "nobody has ever signed
  up" line and a "two people signed up" line can coexist in one file, written weeks
  apart. The newer section wins; say so explicitly instead of quietly picking one.

Related: [[verify-project-status-from-git-refs]], [[state-file-drifts-when-not-updated]],
[[vendor-account-configured-is-not-deployment-wired]].
