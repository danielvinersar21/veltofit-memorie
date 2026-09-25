---
name: landing-seo-baseline
description: Where veltofit.app ranks on Google (Aug 2026), the SEO fixes applied, and the copy work still open.
metadata:
  type: project
---

**Measured 2026-08-28 via web search, three category queries** ("aplicație pentru
antrenori personali România…", "software antrenori personali abonament lunar…",
brand). Result: **veltofit.app appears only for its own name.** Zero presence on
any category query. Google's snippet for it was still the old meta description
("platformă hibridă … venituri pasive"), which is what the LIVE site serves —
the redesign is unmerged.

**Who owns those queries** (all Romanian, all gym-oriented except one):
upfit.cloud / UPfit.team (dominant, several pages), simplegym.ro, smartgym.ro,
easyweek.io, anolla.com, softlead.ro listings, and **fitavio.app — the closest
competitor, a CRM for SOLO trainers**. Most rank for "sală de fitness"
management, not for the solo-trainer niche Velto is in: that niche is thinly
contested and is where to compete.

**Root cause of the invisibility, now FIXED (uncommitted, on `feat/landing-light`):**
- The landing is a `'use client'` page, so it could not export `metadata` and
  inherited the root layout's — **its `<title>` was the single word "Velto"**.
  Added `src/app/(marketing)/layout.tsx` (server component) with a real title,
  description and canonical.
- `metadataBase` was never set → relative OG/canonical URLs resolved against
  whatever host the build ran on. Set to `https://veltofit.app` in the root
  layout.
- ⚠️ **`openGraph`/`twitter` merge per top-level KEY, not per field** — the
  child's `openGraph` replaces the parent's entirely, so the image had to be
  repeated in the marketing layout or previews shipped with no picture.
- The FAQ answers were mounted only while open (`{isOpen && …}`), so **12
  keyword-dense answers were absent from the HTML a crawler reads**. Now always
  rendered with `hidden={!isOpen}`.
- The "venituri pasive" sentence (a claim about a feature that does not exist)
  was in the root metadata, the OG tags AND the Organization JSON-LD. All three
  replaced.

**Also done (owner approved), same day:**
- Hero eyebrow "30 de zile gratuite · fără card" → **"Aplicație pentru antrenori
  personali · România"**, so the category is in the first line of text on the
  page. The offer moved to the line beside the CTA, which now reads the full
  "30 de zile gratuite, cu clienți nelimitați. Fără card."
- **Share card is now generated**: `src/app/(marketing)/opengraph-image.tsx`
  (next/og, 1200×630, prerendered static at build). Sora + Inter TTFs live in
  `public/fonts/` — **Satori cannot read the woff2 next/font produces**, and
  `fetch(new URL('./x.ttf', import.meta.url))` 500s under Turbopack dev, so it
  uses `readFile(join(process.cwd(),'public','fonts',…))`. Diacritics verified in
  the rendered PNG. The stale hero-image.png is no longer referenced by the
  landing (root layout still points at it for other routes).

**Search Console, checked 2026-08-28 (owner had never set it up):**
- **DNS is on Vercel** (`ns1/ns2.vercel-dns.com`) → TXT records go in the Vercel
  dashboard, Domains → veltofit.app → DNS.
- 🔎 **A `google-site-verification` TXT ALREADY EXISTS** on the apex
  (`BK3hSmPA6g3V31pn-x9-tzQ2-i1yTI7o78JH3kmWPV0`), alongside MailerLite's and an
  SPF record. Tokens are per Google account, so this is either an old property of
  his or someone else's (Robert?) — check the account before adding a second.
  Multiple such TXT records are legal; do NOT delete the existing one.
- Live site is otherwise crawl-ready: robots.txt allows all, sitemap.xml serves 4
  URLs, `www` 307s to the apex. **But the LIVE title is still "Velto"** — all the
  metadata work is unmerged, so deploy BEFORE requesting indexing, or Google
  re-crawls the old page.

**✅ App hosts kept out of the index** (`src/middleware.ts`): `X-Robots-Tag:
noindex, nofollow` on app.* and dashboard.*. robots.txt cannot express it — one
static file serves all three hosts — so the rule travels as a header. Only the
two branches that return a PAGE carry it; redirects land back on one that does.
Verified per host locally. veltofit.app/login is NOT covered (deliberate, minimal
change) — offered to the owner.

**Copy placements — APPLIED 2026-08-28 (owner approved):**
- Hero subhead now opens "**Ca antrenor personal,** îți administrezi clienții,
  planurile și vezi progresul lor…" (both verbs kept — "administrezi" + "vezi" is
  a documented decision, progress is seen, not administered).
- Steps section: eyebrow "Cum funcționează" → "**Pas cu pas**", h2 "Ușor de
  folosit. Gata de acțiune." → "**Cum funcționează Velto**". Split that way
  because eyebrow + h2 both starting "Cum funcționează" read as a stutter.
  ⚠️ The replaced h2 is on the verbatim-keep list in
  `docs/plans/landing-redesign-prompt.md` — second line of that brief now stale
  (the pricing title was the first).
- **`.stepsHeader` was capped at `max-width: 30em`, i.e. 450px** — a cap written
  for a lede the header does not have. The heading's first two words alone
  measure 486px at desktop size, so ANY heading there wrapped three deep. Now
  42em like the other sections; also gave `.stepsTitle` `text-wrap: balance`.
- The h1 itself is deliberately untouched: no search term in it, and it stays
  that way.
- One page cannot rank for a category. Recommended next: a few real content
  pages, and Google Search Console (never set up as far as anything in the repo
  shows).

See [[landing-light-redesign]] for the redesign itself.

**Search Console data, 2026-09-25 (3 months, Web):** 7 clicks, 33 impressions,
avg position 3.7. Visible queries are ONLY branded: "veltofit" 3/10, "velto"
0/8; the other 4 clicks / 15 impressions are queries Google hides as too rare.
Baseline confirmed: still zero category traffic. A Google "10 clicks in 28 days"
milestone email arrived 23 sep. It doesn't match this view (7 in 3 months), so
it most likely counts other search types or another property. Don't read it as growth.
