---
name: verify-a-feature-by-finding-its-writer
description: How to tell a real feature from a rendered-but-never-written one — grep for who WRITES a state, not who displays it; the reader always exists first.
metadata:
  type: project
---

When checking whether a feature actually works, grep for the code that **writes** the
state, not the code that displays it. A displayed state with no writer is the most
common false "complete" in any codebase I inventory.

**Why:** UI and types get built ahead of the action that produces the state, so the
evidence of completeness (an enum value, a badge, a colour, a column) is all present
and greps positive. In this project the same shape has now appeared three separate
times: a session status rendered on two screens and never written by anything; a
join table that no app-created row ever populates; a service+store+hook trio with no
page calling it. Each one read as "built" in a doc and was a dead end in the product.

**How to apply:**
- Two greps, never one. First the reader (`case 'x':`, the label, the badge class),
  then the writer (`= 'x'`, `status: 'x'`, `.insert(`, `.update(`). If the second grep
  returns only tests, comments and type definitions, the feature does not exist.
- For a 4-layer architecture, the missing layer is usually the top one. Grep the
  service function's name across the repo: if every hit is inside
  services/stores/hooks and none is in a page or component, the back end is fine and
  the gap is UI-only — which is a much smaller and much more honest estimate.
- Say which direction works. "Library → plan day works, Library → client does not" is
  useful; "the library is half-built" sends someone to re-derive it.
- The inverse is also worth a check when a doc says something is dead code: grep the
  writer. A column that is still read but can no longer be written (its update
  function is called from nowhere) is a third state — frozen, not dead — and deleting
  the reader is a wide change, not a two-line cleanup.

Related: [[verify-project-status-from-git-refs]], [[state-file-drifts-when-not-updated]].
