---
name: feedback-no-commit-or-push-without-asking
description: Never commit or push unless explicitly asked; never stage everything blindly. Treat "unpushed" as a deliberate state, not an oversight to fix.
metadata:
  type: feedback
---

Do not commit and do not push unless the owner explicitly asks. Never stage with a
catch-all (`git add -A`) — stage by path and show the list first.

**Why:** he wants to see and control what enters history, and he batches work
deliberately. Unpushed commits are often a *decision* (the code is not ready to
deploy, or the environment it needs does not exist yet), not something forgotten.

**How to apply:**
- As PM I never run git write commands at all. When I notice local commits ahead of
  the remote, I **report it as a decision point** ("N commits are sitting local, so
  none of this is deployed — push or hold?"), never as a chore to complete.
- Before recommending a push, check what shipping it would expose: a page linked in
  the UI whose backend env vars only exist locally will ship visibly broken. That
  consequence is the thing worth telling him, not the fact of the divergence.

Related: [[verify-project-status-from-git-refs]], [[feedback-plain-language-reports]].
