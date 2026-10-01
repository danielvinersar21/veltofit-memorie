---
name: feedback-admin-direct-commit
description: 30 sep — pentru schimbări mici în admin, owner-ul a spus de 3 ori „fă direct commit și push"; nu mai cere confirmare pe listă pentru ele.
metadata:
  type: feedback
---

Pe 2026-09-30, la paginile de admin (Campanii, Cheltuieli), owner-ul a răspuns de trei ori la rând
„fă direct commit și push" când i-am cerut confirmarea listei de fișiere.

**Why:** pentru iterații mici pe cod deja revizuit și testat, confirmarea listei îl încetinește.
**How to apply:** după ce poarta de calitate trece, pentru schimbări de cod în admin fă commit + push
direct (stage pe cale, niciodată `git add -A`, fără fișierele străine). Rămâne să întrebi pentru
migrări de bază de date, schimbări pe landing/pagini publice sau legale, și orice e ireversibil.
Rafinează [[feedback-ask-before-committing]], nu îl anulează.
