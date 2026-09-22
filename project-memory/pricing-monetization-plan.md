---
name: pricing-monetization-plan
description: "Velto pricing — full phased plan (SaaS subscription, RON, 3 tiers Free/Plus/Max + landing section). Decided 2026-07-31, plan 2026-08-01. Deliberately NOT built now."
metadata: 
  node_type: memory
  type: project
  originSessionId: b32fd3dc-22e2-4b25-b4ce-d6aa81ea1002
  modified: 2026-08-20T12:30:14.989Z
---

Velto monetization decided 2026-07-31. **Pure SaaS subscription** (trainer pays
Velto; clients free — NOT the client→trainer marketplace payments that
[[roadmap-features-plan]] PARKED). Pricing axis = **active-client count, not
features**; inspired by (not copied from) Trainerize. **Romania-first, RON.**

3 tiers: **Velto Free** (2 clients, no Clase/Calendar/Feed) · **Velto Plus** 99
RON (up to 20 clients) · **Velto Max** 299 RON (up to 50 + seat add-ons +49 RON/10
via Stripe quantity). Annual = 2 months free. Same features on all paid tiers —
only Free loses roadmap premium. **Trial = universal** (every new trainer 30 days
full & unlimited, then picks a tier) — built as an app-level `trial_ends_at` flag,
NOT a Stripe trial, no card.

Numbers driven by real customer-dev: 2 trainers testing the app said original
59/109 was too low. 3× price jump (99→299) matched by widened caps (20→50) to kill
the mid-roster cliff. Held 349-Max in reserve for after roadmap ships.

**Key architecture rule:** model the limit as a **number** (`seat_limit`), never a
per-tier enum — then flat tiers and metered seats are the same code and Stripe's
Customer Portal handles seat self-serve. **Nothing built yet, deliberately parked**
— needs Stripe Billing + `plan_tier`/`seat_limit`/`trial_ends_at` columns + webhook
+ in-app seat enforcement (payments are a CLAUDE.md §6 Gap).

**Landing finding — RESOLVED 2026-08-20.** The stale "Growth" / 600 RON copy is
gone and the pricing section is built; see [[landing-pricing-section-built]].

**Launch posture decided 2026-08-20** ([[launch-gtm-decision]]): no free beta —
real prices public + the 30-day no-card trial + manual invoicing for the first ~10
trainers. **Stripe stays unbuilt on purpose** until ~10 people actually pay. The
next code to write is the *enforcement* phase only (`trial_ends_at`, `seat_limit`,
graceful lock, in-app countdown) — and it has a hard deadline: the landing now
promises the trial, so enforcement must ship before the first signup reaches day 31.

Full phased plan (DB → enforcement → Stripe → landing section spec):
`docs/plans/pricing-monetization-v1.md`.
