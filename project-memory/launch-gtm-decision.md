---
name: launch-gtm-decision
description: "Velto go-to-market decided 2026-08-20: NO free beta — real prices + 30-day no-card trial + manual invoicing for the first ~10 trainers. Landing pass BUILT & uncommitted. Two RO competitors found."
metadata: 
  node_type: memory
  type: project
  originSessionId: 78f05d09-0105-4598-88cf-b9ebb7bb196a
  modified: 2026-08-29T20:06:29.096Z
---

## ⚠️ REVERSED 2026-08-30: the trial now REQUIRES A CARD

The "30-day no-card trial" below is **no longer the model.** Spec:
`docs/plans/trial-end-and-card-v1.md`.

**Why it broke.** The trial granted unlimited clients, Free grants 2, and nothing
happened on day 31 except "you cannot add more". So: sign up, add 100 clients in
30 days, keep all 100 active free forever. The cap only bit growth, never the
existing roster — a door anyone walks through, not a conversion gap. Owner
spotted it and rejected the soft options.

**The new model:** Free stays cardless at 2 clients. Paid plans **take a card at
signup**, are free for 30 days, cancel any time inside that window. So the seat
cap on day 1 equals the cap on day 31 and the over-cap state stops being the
default outcome of every successful trial.

**The consequence that reorders everything: Stripe is now a DAY-1 blocker.** With
no card, Stripe was only needed by day 31, which bought three weeks. If signup
takes a card, there is no public signup at all until Checkout exists.

Owner accepted the downside in his own words: some people drop off when they see
a card, *„dar până la urmă așa sunt 90% din SaaS-uri."* Mitigation is copy —
"anulezi oricând în 30 de zile" next to every button, never buried in the FAQ.

A wall is still built, as a net rather than the main path: when someone falls
from a paid plan to Free while over 2 clients (cancel, failed payment, downgrade)
the app blocks and asks them to pick a plan, then to archive down to its cap —
**they choose who stays, nothing is deleted, archiving stays reversible.**

⚠️ Eleven places still promise "fără card", including the OG share image and the
meta description. Listed in the spec. `Fără card. Fără termen.` on the FREE tier
stays true — only the trial claims change.

---

Go-to-market settled 2026-08-20, after the owner asked whether to start posting on
Instagram and whether the app could take real clients.

**The decision: skip the free beta entirely.** Owner pushed back on a "gratuit în
beta" framing and was right — free-beta anchors the product at zero and makes the
first invoice feel like a betrayal. Instead: **real prices public from day one +
the universal 30-day no-card trial already spec'd in
[[pricing-monetization-plan]] + manual invoicing for the first ~10 trainers**
(payment link / transfer, flag flipped by hand in the admin dashboard). Stripe is
NOT built and deliberately stays unbuilt until ~10 people actually pay — building
billing infra for zero users is the trap.

**Why the trial is the load-bearing piece:** it buys ~30 days of runway before any
billing mechanism is needed at all (first signup doesn't pay until day 31), removes
signup friction, and avoids the free-anchor. Owner has a **PFA** — enough to invoice;
under the 300k RON VAT threshold so no TVA; e-Factura is the real compliance item
(recommended SmartBill/Oblio rather than manual SPV).

**Pricing framing shift:** competitor leads with "fără taxe per client", a frontal
attack on Velto's client-count axis. Do NOT change the model (it's Trainerize's for
good reason) — change the wording. Lead with "plătești pe măsură ce crești, nu
pentru funcții"; say "acoperă până la N clienți", never "limitat la N".

**Competitors (RO, same niche):**
- **peak-fitness.eu** — same market, same stage, open beta, free-no-card, prices
  undisclosed, zero testimonials ("Primii antrenori: vine în curând"), no SEO
  presence, web-only + WhatsApp distribution. NOT ahead of Velto. Genuinely better
  at two things worth copying: **Google/magic-link signup** (Velto still demands
  email+password+confirm) and **WhatsApp-link distribution as marketing copy**.
  Also advertises program creation from free-form text — smells like an LLM, which
  makes the parked `docs/plans/ai-integration-v1.md` newly relevant.
- **trainly.ro** — surfaced via search, looks more built out and HAS search
  presence. Never examined. Worth more attention than PeakFitness.
- **i-am-pt.com** — found 2026-09-02, a solo PFA **in Alba Iulia itself**.
  Torn down in [[competitor-teardown-2026-09]]; zero users by his own admission.

**Instagram:** `@veltofit.app` already exists (in the landing JSON-LD). Advice
given: start the account now, hold the traffic push until the funnel was fixed.
That fix is now done (below).

**INVOICING — settled 2026-08-20, runbook at `docs/plans/billing-operations-v1.md`.**
Owner reversed the manual-invoicing plan: Stripe gets built up front. **Stripe is
the invoice issuer; Solo is the bookkeeping + SPV channel** — Solo has an "Încarcă
facturi de venit" upload flow (no API/webhook integration, but none is needed).
Two traps this creates: Stripe numbers invoices PER CUSTOMER by default and must
be switched to account-level sequential BEFORE the first invoice (not rewritable
retroactively); and Checkout must be told to collect Tax ID + billing address, or
the invoices are not valid B2B documents. The routine is WEEKLY not monthly —
Stripe bills on each subscriber's anniversary, and the e-Factura clock runs from
each invoice's issuance. Owner already has the intra-community VAT code and CAEN
5829/6310, so both of those worries are closed.

**THE OPEN RISK:** the landing now promises a 30-day trial the app does not
enforce — no `trial_ends_at`, no countdown, no day-31 behavior, no limits at all.
Harmless for the first 30 days; the real deadline is **enforcement must ship before
the first signup hits day 31**. See [[landing-pricing-section-built]].
