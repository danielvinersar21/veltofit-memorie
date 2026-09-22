---
name: roadmap-features-plan
description: "Landing 'Ce urmează' roadmap features spec'd 2026-07-30 (Calendar, Clase, Nutriție, Comunicare/feed). Vision only — NOT built. Full plan in docs/plans/roadmap-features-v1.md."
metadata: 
  node_type: memory
  type: project
  originSessionId: c6314947-1834-4c77-a4df-6e66ae5feb1c
  modified: 2026-07-30T13:41:43.676Z
---

During the landing **"Ce urmează" (#ce-urmeaza)** section rework (2026-07-30) we spec'd the roadmap features. **Full detail: `docs/plans/roadmap-features-v1.md`** (committed to the repo). These are VISION only — NOT built.

Section = 4 "Coming soon" cards (2×2). Removed the "Disponibil" Web Dashboard card (it's DONE). Dropped Explore (too big + gated on payments). Payments PARKED (contradicts current FAQ "clienții nu plătesc prin Velto").

The 4 features + key decisions:
1. **Calendar & programări** — trainer schedule; ONE system, two entry types: appointment (slot + one-or-more clients, no name) and class. Recurring appts matter (~80% of PT reality). Client read-only view later.
2. **Clase de grup** — named group events; trainer creates + adds clients OR clients self-join. Same scheduling system as Calendar; client JOIN is the main extra.
3. **Planuri de nutriție** — structured in-app meal plans (NOT PDF), mirror workout-plan model (zile→mese→alimente). A+B with macros/kcal OPTIONAL to fill but framed as "când precizia contează" (NOT "opțional" — undersold them). NOT full food-tracking. `meal_plans` table already exists in DB (dormant, zero code) → most ready-to-build.
4. **Comunicare cu clienții** — ONE card = private 1-on-1 **mesagerie** + a broadcast **feed** (trainer posts tips/news to all clients). Feed idea VALIDATED by a real trainer convo.

This came out of a section-by-section landing improvement pass on branch `feat/landing-dark-light-card` — see [[landing-dark-light-branch]]. The landing cards themselves are marketing copy (implemented separately); this memory is about the feature specs.

Suggested build order: Nutriție → Calendar(+Clase) → Comunicare. All follow the 4-layer pattern + need new tables/RLS.
