---
name: memory-status-claims-need-a-git-sweep
description: "TASK, not yet done: sweep every memory note's git-checkable status claim (merged/pushed/applied) against the repo and correct them. Triggered by a stale note that caused a false alarm on 2026-09-24."
metadata:
  type: project
---

**Open task, agreed with the owner 2026-09-24. Not started.** Background work — he explicitly said *not* to spend a morning on it.

## What happened

`[[backlog-active-workout-trio]]` said "NOT merged, NOT pushed" from 2026-08-07 and was never corrected after the branch landed in `main`. On 2026-09-24 a `project-manager` run read the note, believed it, and reported to the owner — in red — that he had seven weeks of work sitting on a single disk with no Time Machine. He did not. One `git merge-base --is-ancestor dad3d35 main` would have caught it; nobody ran one, including me before relaying it.

## Why this is worth a sweep

There are ~60 notes in this directory and a large share carry a **status claim**: merged / not merged, pushed / not pushed, applied in prod / written but not applied, SHIPPED / on a branch. Those claims were true when written and **nothing updates them when the world moves**. The index in `MEMORY.md` repeats many of them, so a wrong one gets read twice.

The failure is asymmetric and that is the point: a note that says "done" when it isn't makes us skip real work; a note that says "not done" when it is makes us raise a false alarm and burn the owner's trust in the reports. The second one is what happened.

## What the sweep should actually do

Only the **git-checkable** half — do not try to verify everything:

- **Merged?** `git merge-base --is-ancestor <sha> main` (or `git branch --no-merged main` once, for the whole list).
- **Pushed?** same test against `origin/main`, plus `git log --oneline origin/main..main` for anything local-only.
- **Commit exists / is where the note says?** `git cat-file -e <sha>` and `git log --oneline -1 <sha>`.
- Correct the note **and** its `MEMORY.md` line together — they drift apart otherwise, which is how this one survived.

**Explicitly out of scope:** "applied in prod" for SQL. Git cannot answer it — the repo does not record which of the 58 `.sql` files ran (see CLAUDE.md §3). Those claims can only be checked against the live database, which is a different job and needs the owner's credentials. Do not guess at them during this sweep; leave them alone and list them.

## The rule that applies even without the sweep

Before repeating any status claim from a memory note in something the owner reads, verify it. Notes record what was true when written, not what is true now. This applies hardest to anything about to be reported as urgent — urgency is exactly when a stale claim does damage.

Related: [[backlog-active-workout-trio]], [[feedback-plain-language]].
