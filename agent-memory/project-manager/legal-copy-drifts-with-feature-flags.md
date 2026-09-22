---
name: legal-copy-drifts-with-feature-flags
description: Terms/privacy/cookies pages are a product surface — a flipped feature flag, an unconfigured payment-processor setting, or a promised right with no procedure behind it silently makes them false.
metadata:
  type: project
---

Treat legal pages (terms, privacy, cookies) as **product surfaces that go stale**, not
as documents that are written once. Before reviewing or trusting them, check three
things in the code, not in the text:

1. **Feature flags.** One boolean can disable several user-facing fields at once. The
   policy usually gets updated for the field somebody remembered, and stays wrong for
   the others the same flag turned off.
2. **Payment-processor settings that exist per mode.** Cancellation timing, downgrade
   timing, trial handling and dunning outcomes are dashboard configuration, not code.
   Configured in test mode ≠ configured in live. Terms that describe them are
   describing test mode until proven otherwise.
3. **Conditional grants stated unconditionally.** Anything the product grants *only
   under a condition* (a trial that is once-per-account, a cap, a discount) tends to
   appear in the terms as a flat promise. The UI is usually conditional and the legal
   text is usually not — that gap is the expensive one, because the customer has the
   text as evidence.
4. **Rights promised with no procedure behind them.** "We delete your account within 30
   days", "we stop if you object" — check that something exists to RUN (a script, an
   RPC) and that the schema lets it run. Seen 2026-09-21: three published pages promised
   deletion, the only deletion script was test-only by its own words, and a foreign key
   with no ON DELETE would have made the database refuse the delete. Not false until the
   first request arrives — which is why nobody notices it.

**When a section is rewritten to say the OPPOSITE**, the sections that survive with the
old story are not the ones that *describe* the thing — those get patched, because they
are what the author was looking at. It is the ones that grant a **right over** it:
access, deletion, withdrawal of consent, who else can see it, what a parent is liable
for. Sweep those by name after any inversion; a caveat added to the description does
not reach them.

**Why:** in this project the pattern has produced the worst errors repeatedly — a
policy that denied having a payment processor while one was integrated, a field
described as both disabled and in use. The owner names it himself as the thing that
costs him most.

**How to apply:** when asked to read legal copy, grep for the constants the text
quotes and read the operations doc for the processor settings before commenting on
wording. Also check the file's own header comments — long explanatory headers go stale
faster than the body and are what the next agent reads first.

Related: [[verify-project-status-from-git-refs]], [[state-file-drifts-when-not-updated]],
[[feedback-plain-language-reports]].
