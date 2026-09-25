---
name: feedback-check-legal-text-before-offering-options
description: "Read the live legal pages BEFORE offering the owner product options — on 2026-09-24 one of three options I offered contradicted his own terms page, and I only found out after he picked it."
metadata:
  type: feedback
---

Before putting product options in front of the owner, check whether the live
legal pages already answer the question. `src/app/terms/page.tsx`,
`src/app/privacy/page.tsx` and `src/app/cookies/page.tsx` are **published
promises**, not documentation — a product decision that contradicts them makes
the site false the day it ships.

**Why:** on 2026-09-24 I offered three options for "what happens to a trainer's
clients when the trainer deletes their account". He picked the third — delete
everything, including the clients. Only then did I read the terms, which say, in
his own words: *"Tu decizi ce înregistrezi despre clienții tăi [...] pentru
datele astea, tu ești operator"* and *"Contul clientului tău este tot al lui — el
se autentifică singur, își vede datele și ni se poate adresa direct."* His choice
would have made that page false. I had to walk the decision back one message
after he made it.

Two things went wrong, and the second is the one worth fixing:
1. I presented an option I had not checked against the live text.
2. My own recommendation ("nu l-aș recomanda") was framed as taste — *deleting a
   third party's data* — when a concrete, citable constraint existed the whole
   time. A vague objection is easy to overrule; a quoted sentence from his own
   site is not. He changed his mind in one message once I quoted it.

**How to apply:** when a decision touches what happens to user data, accounts,
deletion, retention, consent, payments or what a third party receives, grep the
three legal pages first and quote the binding sentence *inside the options*. If
the pages are silent, say so — "the text doesn't cover this, so either choice is
open" is itself useful. And if he picks something the text forbids, the honest
answer is not "ok" and not "no": it is *"that needs the text changed first"*,
with the cost of each path.

The legal pages have been rewritten into falsehood four times already by editing
one spot and leaving the neighbours telling the old story — see
[[legal-pages-hardening]]. Same failure mode, one level up: here the code would
have been the thing that made the text false.

Related: [[velto-positioning-statement]], [[feedback-plain-language]].
