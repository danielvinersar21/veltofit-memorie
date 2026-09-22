---
name: backlog-memory-has-no-backup
description: "MUST DO, not optional: the 48 memory files exist in exactly one copy on one laptop with no backup of any kind. Owner confirmed 2026-08-29 it needs doing; not started."
metadata: 
  node_type: memory
  type: project
  originSessionId: dec69265-dcee-4170-8138-beb9b84c8c1d
  modified: 2026-08-29T18:56:53.535Z
---

**Status: TO DO, confirmed by the owner 2026-08-29 as something that must be done — not a
"maybe". Not started. Do not close this by arguing it's low risk.**

## The problem, verified three ways (not assumed)

The memory lives at `~/.claude/projects/-Users-danielvinersar-repos-velto-webapp/memory/` —
outside the repo, in the home folder. It is Claude Code's default location; nobody chose it.

- `git rev-parse` from inside that directory → **not a git repository**
- `~/.claude/.git` → **does not exist**
- `tmutil destinationinfo` → **"No destinations configured"** — Time Machine is not set up
- Not under `~/Library/Mobile Documents`, so not in iCloud Drive either

48 files, 2,689 lines, 276 KB, in **exactly one copy, on one disk, with no backup at all.**

## Why it actually matters

The code is safe — 331 commits are on GitHub. What is not anywhere else is the **why**:
why the free beta was dropped and what changed the owner's mind, why manual invoicing was
reversed the same day it was decided, that a column REVOKE cannot subtract from a table grant,
that PostgREST returns 200 even when RLS threw the write away, and every deferred item carrying
"don't re-propose". Git tells you *what* changed. Only these files say *why*.

**The repo is private** — confirmed 2026-08-29: the GitHub API returns 404 unauthenticated for
a repo we demonstrably push to. That matters because the memory holds pricing strategy,
competitor notes and the owner's own words about launch. Copying it into this repo is safe;
copying it into a public one would not have been.

## How to do it

Destination: **`.claude/memory/notes/`** — verified not gitignored (`.gitignore:37` only ignores
`.claude/projects/`), and it sits beside `.claude/memory/velto-project-state.md`, which is
already tracked. Copy the whole folder including `MEMORY.md`.

**The open design question — decide before writing code.** PrintBox does this as step 6 of a
written end-of-session ritual. We have no such ritual ([[workshop-exchange-ionut]], gap #3), and
our house style is enforcement over remembering. But the obvious enforcement point doesn't work
cleanly:

- **pre-push hook** — runs *after* the commit exists, so files copied there would not be in the
  commit being pushed. Wrong moment.
- **pre-commit hook** — right moment, but we have no pre-commit hook today and adding one puts a
  copy step in front of every commit, including ones that touched no memory.
- **an npm command run deliberately** (`npm run memory:backup`) — honest, matches how
  `ds:audit:save` already works, but it is a remembered step, which is the thing we criticised
  PrintBox for.

Recommendation if nobody has a better idea: the npm command **plus** a check in the existing
pre-push hook that only *warns* when `.claude/memory/notes/` is more than a few days behind the
live folder. That keeps enforcement where it already lives without blocking a push over docs.

## How to apply

Do this as its own change, not folded into feature work. It touches no application code, so it
cannot break the product — but it does put business reasoning into git history permanently, so
get it right the first time rather than committing and rewriting.

Related: [[workshop-exchange-ionut]] (where this was found), [[backlog-exposure-deferred]].
