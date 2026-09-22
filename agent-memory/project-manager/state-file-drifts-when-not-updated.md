---
name: state-file-drifts-when-not-updated
description: A project-state file skipped for a few weeks goes silently wrong — the largest thread of work can be entirely missing from it. Date it and rebuild from files.
metadata:
  type: project
---

A memory/state file that is not updated every session does not degrade gracefully — it
stays confidently wrong. A wrong state file looks exactly like a correct one.

**Why:** on this project the state file went ~two weeks untouched while the single
biggest piece of work (the payment integration) was designed, built, reviewed and
committed. Nothing in the file hinted anything was missing; it simply ended, and the
last entry read as current. The same failure mode is documented in the repo's own
instructions, which carry a "last verified" date for exactly this reason.

**How to apply:**
- Put a `_Last updated: <date>_` line at the top and check it first. If it is more than
  a week or two old, **rebuild the live sections from files and refs before reporting
  anything** — see [[verify-project-status-from-git-refs]].
- Keep the structured sections (Completed / In Progress / Backlog / Blockers /
  Decisions) at the top and let the append-only session log live below an "Archive"
  line. A file that is only a session log cannot answer "where are we".
- Record *why* a decision was made next to it, and mark decisions that supersede an
  earlier one. Priorities reverse (this project reversed its payment approach twice);
  without the supersede note, old priority lists keep looking authoritative.
- Watch for the sibling failure: **a decision that changed in code but never reached
  the documents and the user-facing copy.** Worth checking as a category, because
  nothing breaks when copy goes stale — it just becomes false.
