---
name: feedback-guide-manual-tests-step-by-step
description: "Owner wants manual test runs walked through step by step, live, not handed to him as a checklist."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6446ed31-fe38-4c22-8d5c-73122b5e1525
  modified: 2026-09-07T08:28:57.232Z
---

Owner, 2026-09-07: when a manual test from `docs/plans/launch-readiness-v1.md`
comes due, he wants to run it **together, step by step** — I tell him the next
action, he does it, he reports what he saw, I say whether that is right and what
comes next. He asked for this explicitly after I wrote tests 13–16 into the plan.

**Why:** the list in the plan file is the record of *what* to test, not a script
he executes alone. Several of these tests fail in ways that look like success —
a password-reset link that lands on `/login` reads as "the app logged me out",
a webhook that never arrives leaves the account on Free while the payment page
says thank you. He wants a second pair of eyes on the interpretation, not just
the instruction.

**How to apply:** when he says "hai să testăm X" — or when a blocker he was
waiting on clears — don't paste the whole test block. Give one step, wait, read
his answer, then the next. Say up front what a *pass* looks like and what the
known failure signature is, so he knows what he is looking at. Keep the plan file
as the source of the steps; see [[launch-readiness-plan]].
