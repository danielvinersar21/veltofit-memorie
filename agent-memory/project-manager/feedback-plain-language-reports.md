---
name: feedback-plain-language-reports
description: Status reports must be plain-language and lead with consequences, not mechanisms — keep the rigour in the work, strip it from the report.
metadata:
  type: feedback
---

Write status reports in plain, spoken language, leading with the **consequence** for
the owner or his users — not the mechanism that produces it. Keep file paths, line
numbers and architecture detail in the memory file, not in the report.

**Why:** he said the reports had drifted so technical he could no longer follow them,
and he is the only person who can act on them — an unreadable report is a report that
does nothing. The rigour is still required in the *work* (verify everything against
files); it just must not leak into the summary.

**How to apply:**
- "A paying trainer can silently stay on the free plan" ✅, not "PostgREST returns 204
  with error === null on a zero-row update" ❌ (that line belongs in the memory file).
- Every backlog item gets *why it is there* and *how long* (S/M/L). He prioritises by
  consequence and cost, so an item without both is unusable to him.
- Mark clearly what is **in production** vs what is only **written on disk**. This
  distinction is the one he most often loses track of, and it is where real risk hides.
- Same rule for product copy: he rejects text that sounds written by an AI. Test each
  line against "would a person say this out loud?"
- **Always close a review with what you checked and found clean.** He asks for this
  explicitly ("e la fel de util"): a finding list alone leaves him unable to tell
  "no problem there" from "nobody looked there", so the clean list is what makes the
  findings trustworthy. Name the file and the constant you checked against.

Related: [[user-owner-profile]], [[legal-copy-drifts-with-feature-flags]].
