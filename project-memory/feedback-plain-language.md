---
name: feedback-plain-language
description: "Owner asked 2026-08-21 for plain-language explanations — status updates had become too technical to follow. Keep the depth in the work, not in the report."
metadata:
  node_type: memory
  type: feedback
---

2026-08-21, mid-way through the billing work, the owner said the updates had
become too technical to follow and asked what had actually been done and what
was next.

**Why:** the owner is the product decision-maker, not a DBA. Reports full of
policy names, SQLSTATE codes, trigger semantics and file:line citations cost
them the thread — and when they lose the thread they cannot make the call that
the report exists to enable. A status update whose reader has to decode it has
failed at its only job.

**How to apply:** keep the rigour in the work — adversarial review, verifying
claims personally, refusing to declare a green on a vacuous test — and strip it
out of the report. Say what changed for a *person using Velto*, what it costs,
and what decision is now on the table. Reach for a concrete consequence ("a
trainer could have seen another trainer's client's injury notes") over the
mechanism ("a FOR ALL policy missing WITH CHECK"). Offer the detail if they want
it; don't lead with it.

Technical depth is still welcome when they ask a technical question — they
asked precise ones about `health_notes` and about the trial→Stripe transition.
The rule is about *unprompted status reports*, not about answering.

Related: [[use-project-agents-by-default]].

**Extends to product copy, not just reports** (2026-08-26): he rejected a landing subhead
containing "notează ce a ridicat" as *"cam plastica, cam scrisa de AI"*. The test for copy is
"would a trainer say this out loud?" — and his own spoken description of a feature is better raw
material than anything composed for him. Reuse his nouns and verbs instead of upgrading them.
See [[landing-light-redesign]].
