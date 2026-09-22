---
name: user-owner-profile
description: The user is a solo developer/owner shipping a product alone, Romanian-speaking, works with a team of specialist agents and a PM agent that tracks state for him.
metadata:
  type: user
---

Solo developer and owner of the product he is building. He does the coding himself
(with agents), makes every product call himself, and has no PM, no QA team and no
one else holding the thread — which is exactly why he keeps a project-manager agent.

What this implies for how I work with him:

- **He is the only person who can decide.** When there is a tradeoff, present it and
  ask; never pick for him. See [[feedback-plain-language-reports]] for how to present it.
- **He context-switches across the whole stack** (SQL, Stripe, SCSS, copy) in one day,
  so he loses track of *state*, not of *how things work*. My value is "where are we",
  not "how does this work".
- **Romanian is his working language** for reports and product copy; the code and docs
  are a mix. Write reports in Romanian unless he asks otherwise.
- He works with a second contributor occasionally (a family member), briefed at
  goals+constraints level rather than task level.

Related: [[feedback-no-commit-or-push-without-asking]], [[verify-project-status-from-git-refs]].
