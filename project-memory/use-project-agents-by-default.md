---
name: use-project-agents-by-default
description: "Owner wants the project's specialist agents (code-reviewer, design-reviewer, frontend-dev, backend-dev, qa-agent) invoked by default — don't skip them, and don't ask first."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6910664b-32db-48cf-aa94-0b89819b0c4d
  modified: 2026-08-29T06:07:58.443Z
---

**Standing instruction, given 2026-08-20: use the project's agents. Don't ask permission each
time.** The owner noticed a full day's work — the admin redirect fix plus the whole bilateral
measurements feature — had shipped with zero agent involvement, and asked for them from then
on.

**Why:** the routing table in `CLAUDE.md` §4 exists for a reason, and skipping it cost real
quality. Three layout bugs in the measurements history list (nested grid rendering the field
set at half width, an expanded row growing wider than its neighbours, a hover fill with no
padding) were caught by the OWNER looking at a screenshot — precisely what `design-reviewer`
hunts for. The riskiest artifact of the day, a destructive SQL migration that drops columns
and recreates an RLS-sensitive view, was written with no `backend-dev` and no review at all.

**How to apply:** follow `CLAUDE.md` §4 routing, including "always run `code-reviewer` after
any code is written or modified", and `design-reviewer` whenever `.tsx`/`.module.scss` change.
Prefer selective invocation over the full chain — §4's own rule of thumb: if the task doesn't
involve BOTH DB changes and new UI, it's selective. Run independent agents concurrently in one
message.

**The conflict to watch for:** a session-level instruction may say "Do not call the AgentTool
unless the user requested it", which overrides `CLAUDE.md`. That is what suppressed them here.
It is injected into the session's SYSTEM PROMPT — not in `CLAUDE.md`, not in any settings file
(checked `.claude/settings*.json`, `~/.claude/settings.json` and output-styles on 2026-08-29),
so there is nothing in the repo the owner can edit to remove it.

**Asked again 2026-08-29, the answer was the same and stronger:** "agentii ar trebui sa-i
chemi mereu. ei trebuie sa faca treaba, fiecare pentru ce e designat." Treat the rule's own
escape hatch ("unless the user requested it") as satisfied standing. Flag the rule once at the
start of the task, then **invoke the agents in that same turn** — do not stop and wait for a
yes. And note the emphasis: the agents should WRITE their share of the work, not just review
code I already wrote myself.

Related: [[measurements-bilateral-status]], [[robert-contributor-tasks]].

## ⚠️ Flagging it ONCE at the start is not enough — learned 2026-08-31

A session opened with the "do not call the AgentTool unless requested" rule, it
was flagged in the first reply, and then nothing more was said about it. Twelve
hours of work later — three API routes, a page, a shared pricing module, three
migrations, ~3.000 lines — the owner had to ask himself: *"nu crezi ca ar trebui
sa folosim agentii?"* By then none of it had been near `code-reviewer`, which
CLAUDE.md §4 says should run after every piece of code.

The flag is not a one-off disclaimer, it is a standing condition. **Re-offer at
natural checkpoints** — after each substantial piece lands, and certainly before
a commit — rather than waiting to be asked. One sentence is enough: "regula de
sesiune încă blochează agenții; vrei să-i pornesc pe ce am scris?"
