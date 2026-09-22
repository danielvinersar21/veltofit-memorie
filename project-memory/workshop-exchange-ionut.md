---
name: workshop-exchange-ionut
description: "Workshop-manual exchange with the owner's cousin Ionuț (PrintBox): his manual arrived 2026-08-29, ours was written and sent back the same day. Awaiting his side of the comparison."
metadata: 
  node_type: memory
  type: project
  originSessionId: dec69265-dcee-4170-8138-beb9b84c8c1d
  modified: 2026-08-29T18:44:04.537Z
---

**2026-08-29.** The owner's cousin Ionuț runs PrintBox (personalised gifts, WooCommerce, own
production) and works the same way — Claude Code on a private repo that holds the whole
business. He sent `_cum-e-construit-atelierul.html`: a manual of his workshop plus a protocol —
15 comparison questions and 4 copy-paste prompts (blind inventory → compare with evidence →
adopt exactly one → write your own manual in the same format and send it back).

**Our side is done and sent.** `_cum-e-construit-atelierul-velto.html` at the repo root; also
published as an artifact. Verdict on his grid: **5 DA / 6 PARȚIAL / 4 NU**.

**What we're waiting for:** his answers to the same 15 questions, and specifically how the
agenda holds up after a year — what went into it and was never read again, and whether the
"rewrite whole, max 35 lines" rule survives under pressure. That last one is where we failed.

**Method worth reusing, not just here:** the inventory was run by two agents with their own
context that had never seen his manual. Contamination stops being a discipline problem and
becomes structurally impossible. Use this shape for any audit where the main session already
knows what it hopes to find. See [[use-project-agents-by-default]].

**What his grid showed we lack**, ordered by cost — none of it adopted, deliberately (his own
rule: rules that don't come from a real problem don't get followed):
1. A shared surface — no agenda, no checkboxes. Gym-floor testing never gets back to the repo.
2. A short rewritten "where I left off" — ours grows instead ([[client-home-perf-findings]] era
   journal is now 729 lines).
3. A written end-of-session ritual for docs/memory; the pre-push gate covers code only.
4. Backup for memory — it lives outside git and has no copy. His is scripted into the repo.
5. Reasons and dates on rules; 1 of 10 "non-negotiable" rules carries a why.
6. A measurable number written *before* a plan is applied.
7. A data-source registry, and a tool registry.

**Where we're ahead, and what he should take:** reasons written inside the file that enforces
the rule (pre-push hook, CI config, token file with measured contrast); an automatic gate that
refuses the push instead of a remembered checklist; agents as files with tool grants — and the
lesson that a prose "read-only" claim enforces nothing, check the grant; deferred items carrying
*instructions* ("don't re-propose"), not just reasons; pre-flight queries mandatory before any
migration.

Related: [[backlog-exposure-deferred]] (found during the same inventory), [[feedback-plain-language]].
