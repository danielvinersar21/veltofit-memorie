---
name: backlog-seo-content-pages
description: Backlog — three content pages to rank for category searches, not just the brand name. Un-parked 2026-09-25, owner wants them built.
metadata:
  type: project
---

**UN-PARKED 2026-09-25** — owner decided to build them, in a separate thread, after
Search Console showed 3 months of branded-only traffic (see [[landing-seo-baseline]]).
(Was parked 2026-08-28: "posibil să ajungem la el la un moment dat.")

**Why it exists:** measured that day — veltofit.app appears on Google only for its
own name, on none of the category queries a trainer would type. On-page SEO for
the landing is now fixed (see [[landing-seo-baseline]]), but **one page can only
rank for one subject**, and the competitors owning those queries have dozens.

**The three, in priority order:**
1. `/aplicatie-antrenori-personali` — a page whose whole subject is that phrase.
   The landing sells Velto; this answers "what app do personal trainers use".
   **If only one gets written, this is it.**
2. `/alternativa-la-excel-si-pdf` — catches the moment someone searches "cum țin
   evidența clienților", "program antrenament excel", i.e. before they know
   products like this exist.
3. A guide, e.g. `/ghid/cum-iti-organizezi-clientii` — top of funnel, slowest to
   pay off, best at attracting links.

**Honest cost, as told to the owner:** 600–1000 words of real content each, from
his knowledge, and results in 2–3 months rather than weeks.

**Cheap win that is NOT part of this** and can be done any time: get Velto listed
in Romanian software directories (softlead.ro ranks on page one for these
queries). Gets a link, referral traffic, and presence on a page that already
ranks. An hour of work.

**Measuring it:** Search Console IS set up (data read 2026-09-25). Steps are in [[landing-seo-baseline]].

**2026-09-25 — page 1 BUILT, UNCOMMITTED.** `/aplicatie-antrenori-personali`: server
component, article layout, 4 screenshots in `public/images/aplicatie-antrenori/`,
own share card via shared `src/app/og-card.tsx`, sitemap entry, landing footer link.
Text in `docs/plans/seo-aplicatie-antrenori-personali-text.md` (content-reviewed).
`SourceCarryingLink` / `carrySignupSource` carry signup attribution from content
pages to the landing — reuse for pages 2 and 3. Traps: openGraph must NOT declare
`images` on a page inside `(marketing)` (route-group hash suffix; Next auto-attaches
the colocated card only when `images` is absent). Don't `npm run build` while the
owner's dev server runs (shared `.next`). Next after publish: request indexing in
Search Console, then the softlead.ro directory listing.
