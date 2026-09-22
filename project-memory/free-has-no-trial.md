---
name: free-has-no-trial
description: "Free = 2 locuri, fără card, FĂRĂ trial, pentru totdeauna — trialul vine din Stripe la Checkout, doar pe Plus/Max"
metadata: 
  node_type: memory
  type: project
  originSessionId: f6937a95-9895-4af4-a554-c0ce612b3895
  modified: 2026-09-20T08:39:24.296Z
---

Aplicat în producție **31 august 2026** prin `docs/database/billing_free_has_no_trial.sql`.

**Modelul vechi** (până atunci): orice cont nou primea `plan_tier='free'`,
`subscription_status='trialing'`, `trial_ends_at = now() + 30 zile` → ramura din
`trainer_seat_limit()` care întoarce 9999, adică 30 de zile cu clienți
nelimitați fără card.

**Modelul de acum:**

- **Free** → fără card, **2 locuri**, **fără trial**, pentru totdeauna.
  Un cont nou primește `subscription_status = NULL` și `trial_ends_at = NULL`.
- **Plus / Max** → card la înscriere, pachetul ales gratis 30 de zile, apoi se
  facturează automat. **Trialul îl acordă Stripe la Checkout, nu aplicația.**

## De ce contează că ții minte asta

Un cont de antrenor pe Free cu `trial_ends_at = NULL` **nu e o defecțiune** și
**nu are niciun termen care expiră**. Pe 20 sep 2026 un agent a văzut exact
starea asta la cei doi antrenori organici ([[trainer-accounts-who-is-who]]), a
presupus un trial de 30 de zile și a raportat un termen fals în octombrie.
Owner-ul lămurise deja asta cu o săptămână înainte.

## Capcana din fișier, de reținut

Revendicarea trialului în `billing_trial_ledger` s-a **mutat** din triggerul de
înscriere în `claim_trial_for_trainer()`. Dacă triggerul ar mai revendica la
înscriere, fiecare cont ar fi marcat „trial consumat" înainte să fi primit ceva,
iar Checkout n-ar mai da trial nimănui — tăcut, și imposibil de dedus din
simptom.
