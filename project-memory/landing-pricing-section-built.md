---
name: landing-pricing-section-built
description: "Landing waitlist→signup pass — COMMITTED 2026-08-20 as 1dd353c on main (not pushed). Pricing section live, Growth copy killed. OG image still stale + metadataBase unset."
metadata:
  node_type: memory
  type: project
---

Built and **COMMITTED 2026-08-20 as `1dd353c` on main** (NOT pushed yet) in
`src/app/(marketing)/page.tsx` + `styles.module.scss`, per
[[launch-gtm-decision]]. Gate re-verified at commit time: lint 0, type-check 0,
**168/168 tests**, build OK (95 pages).

What landed:
- New `#preturi` section between "Cum funcționează" and the CTA — 3 cards
  (Free/Plus/Max, real prices from `docs/plans/pricing-monetization-v1.md`), Plus
  featured with the only solid button in the row. Below it, the day-31
  reassurance line (pick a plan or stay on Free, data intact).
- **Waitlist deleted.** All 6 CTAs (nav, mobile menu, hero, 3 cards, final CTA)
  point at `APP_SIGNUP_URL` = `NEXT_PUBLIC_APP_URL` + `/login`, fallback
  `https://app.veltofit.app`.
- The rejected **"Growth" / 600 RON copy is gone** from all 3 spots + FAQ; FAQ now
  carries real caps, the 30-day trial, and a new "Cum se face plata?" entry.
- `/api/subscribe` + MailerLite left wired but unused — kept on purpose.

**Two conventions worth reusing:**
1. `.linkButton` — anchor styled to mirror `ui/Button`'s base + variantDefault.
   Conversion CTAs leave for the app subdomain, so they must be real `<a href>`
   (middle-click, open-in-new-tab, crawlers); `<button>`-based `ui/Button` can't.
2. **Pricing grid deliberately skips a 2-column stage** (1 col → 3 col at `lg`).
   Caught visually at 820px: three tiers in two columns orphans the third and reads
   as "two plans and a footnote".

Capture trick for tall landing sections: the sticky `nav` composites into the
middle of an element screenshot — inject `nav { display: none }` before shooting.
`fullPage: true` + `clip` from `boundingBox()` silently produces a blank image.

**Still open:**
- OG image is still `/images/hero-image.png`, a raw screenshot — needs a designed
  asset, not code.
- **`metadataBase` is NOT set** in `src/app/layout.tsx` (build warns ×4). OG image
  URLs are relative, so they resolve against `VERCEL_URL` (the per-deployment host)
  instead of `veltofit.app`. One-line fix: `metadataBase: new URL('https://veltofit.app')`.
- FAQ "Cum se face plata?" says "pe bază de factură" — written under the
  manual-invoicing assumption. Owner has since decided to build Stripe up front
  ([[pricing-monetization-plan]]), so this copy needs a card-payment rewrite when
  Stripe ships.
