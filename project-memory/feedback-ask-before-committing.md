---
name: feedback-ask-before-committing
description: "Owner 2026-08-30: no git commits without being asked first. Write the change, run the gate, report — then wait."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T19:45:46.770Z
---

**Standing instruction, 2026-08-30: do not commit without the owner saying so.**
Applies to `git commit` and `git push` alike. Write the change, run the quality
gate, report what changed — then stop and wait.

**Why:** it came after a session where six commits went in across a few hours,
each announced but none asked for. The trigger was a near-miss: a `git add -A src/`
staged another Claude session's uncommitted work in the same working tree, caught
only because the staged list was printed before committing. Frequent unrequested
commits also make it harder for the owner to see one change as one thing.

**How to apply:** finish the work and hold it. Say plainly what is staged or ready
and offer to commit. Never `git add -A`, `git add .` or `git add <dir>` — stage
by explicit path, and print `git diff --cached --name-status` before committing so
foreign files are visible. Two Claude sessions have shared this working tree
before; assume they might again.

Related: [[use-project-agents-by-default]] — the owner wants the agents doing the
work, but that does not extend to committing it.
