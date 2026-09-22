---
name: stripe-webhooks-need-cli-locally
description: Local Stripe payments look successful but leave the account on Free — the webhook cannot reach a laptop. Needs `stripe listen`. Cost the owner one confusing test on 2026-09-03.
metadata:
  type: project
---

**A payment made against a local dev server does NOT update the account.** Stripe
takes the money (test mode), the browser returns correctly, and the trainer stays
on Free — because Stripe delivers the subscription through a WEBHOOK, sent from
Stripe's servers to a public URL, and a laptop has no public URL. The owner hit
exactly this on 2026-09-03 testing from his phone over `nip.io` on a LAN IP.

The bridge, which nothing in the repo documented:

```
stripe listen --forward-to localhost:3000/api/billing/webhook
```

It prints a `whsec_…` that must replace `STRIPE_WEBHOOK_SECRET` in `.env.local`
(signature verification fails otherwise), then restart the dev server. An
already-paid session can be recovered without paying again: Stripe Dashboard →
Developers → Events → Resend.

**Why:** without this, a local billing test silently proves the wrong thing. The
redirect looks right, the money moves in Stripe, and only the part that decides
what the trainer can actually DO is missing — the half you were testing.

**How to apply:** say this BEFORE anyone tests a local payment, not after. And
read the account state from Stripe, not from `trainer_subscriptions`, when the
two disagree about a local run.

🔑 The silver lining, verified in code the same day: `/api/billing/checkout` asks
STRIPE — not our own table — before selling. So a trainer whose webhook is
missing is refused with `already_subscribed` rather than sold a second
subscription. That guard is what makes a lost webhook a delay instead of a double
charge, and it works because `stripe_customer_id` is written at CHECKOUT time,
not by the webhook. Do not "simplify" that check to read the local column.

## Măsurat 2026-09-05, patru evenimente reale

| | |
|---|---|
| livrare (Stripe → noi, prin CLI) | 0,5 – 1,8 s |
| handler (re-citire din Stripe + scriere) | 0,3 – 0,7 s |
| cap la cap | 0,9 – 2,4 s |

Din perspectiva antrenorului, pe drumul normal: **zero reverificări, 123 ms** —
webhook-ul scrisese rândul înainte ca redirectarea de la Stripe să aducă pagina
înapoi. Cronometrul de 2 s nu s-a armat niciodată. Cifre de local: în producție
dispare hop-ul prin CLI și apare pornirea la rece pe Vercel.

## 🔧 Recuperarea, verificată — NU cere o plată nouă

Semnătura problemei: abonament ACTIV în Stripe, cont pe Free la noi.
`stripe subscriptions list --status active` le arată.

```bash
stripe subscriptions update <sub_id> -d "metadata[recovered_at]=<data>"
```

`customer.subscription.updated` rescrie rândul, pentru că un abonament cu status
viu revendică un rând care nu ține niciunul (`mayTakeTheRow`). Verificat: rândul
a revenit pe `max`/`active`/30 de locuri în ~1,8 s.

**Procedura e acum SCRISĂ în repo** — `docs/plans/billing-operations-v1.md`,
secțiunea E2, adăugată 2026-09-05. Nu mai trăiește doar în nota asta.

See [[stripe-integration-state]] for what is built.


---

## Două capcane în plus, găsite la testul 15 (2026-09-07)

**1. `stripe listen` e uneori exact ce NU trebuie pornit.** Regula de sus rămâne
adevărată pentru testarea fluxului complet. Dar `.env.local` arată spre baza de
date de **PRODUCȚIE** (`wcpzdurualepyjjrracm`, același proiect ca live-ul), în
timp ce cheile Stripe sunt de test. Deci cu `stripe listen` pornit, webhook-ul
chiar ajunge și scrie în baza reală un abonament cu identificatori din modul
test. Pentru un test care vrea doar FACTURA — aia se creează în Stripe oricum —
îl lași oprit și nimic nu atinge baza.

**2. Localul nu e pe `localhost`.** `.env.local` are
`NEXT_PUBLIC_APP_URL=http://app.192.168.1.132.nip.io:3000` și perechea pentru
dashboard (adrese `nip.io`, ca să meargă testarea de pe telefon în rețea). Iar
`success_url` de la Stripe se construiește din variabilele astea, NU din pagina
de unde ai plecat. Dacă deschizi `dashboard.localhost:3000`, Stripe te întoarce
pe originea `nip.io` — **altă origine, alt `localStorage`, alt cont logat**.
Owner-ul a confirmat: același browser, alt cont. Nu e defect de produs; în
producție cele două coincid. **Pornește testele locale de facturare direct de pe
adresa `nip.io`.**
