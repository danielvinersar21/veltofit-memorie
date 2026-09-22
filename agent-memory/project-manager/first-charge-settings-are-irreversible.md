---
name: first-charge-settings-are-irreversible
description: Payment-processor invoice settings (numbering series, invoice template, statement descriptor) freeze at the first real charge — check them before a launch date, not after.
metadata:
  type: project
---

Before saying "we can launch today" on anything that takes money, check the payment
processor's **invoice/billing settings**, separately from whether payments work.

**Why:** "the integration works" and "the first invoice is legally valid" are different
claims, and only the second one is irreversible. Invoice numbering in particular freezes
at the first issued invoice — if it launches on the processor's default (a separate
sequence per customer) it stays that way for every invoice already issued. The same
class: the invoice template (legal entity name, tax id, address, bank account, any
required non-VAT wording) and the card-statement descriptor, which is what a customer
sees on their bank statement and therefore what drives "I don't recognise this charge"
disputes.

These live in a **dashboard, not in code**, so no test, no code review and no CI gate
will ever mention them. In this project they sat unchecked in an operations runbook while
every other billing item had been ticked off, and no blocker list carried them — the
checklist existed and was simply never re-opened.

**How to apply:** when a launch date is on the table, open the operations/runbook doc and
read its checkboxes as *state*, not as prose. Anything marked "before the first invoice"
or "cannot be rewritten retroactively" goes at the top of the blocker list, above code
work — it is usually 30 minutes of clicking and it is the only category that cannot be
fixed the next day.

Related: [[legal-copy-drifts-with-feature-flags]] (same root: processor settings are
per-mode configuration that the product's own text quietly assumes),
[[verify-project-status-from-git-refs]].
