---
name: landing-dark-light-branch
description: "In-progress branch feat/landing-dark-light-card — landing changes, NOT pushed yet"
metadata: 
  node_type: memory
  type: project
  originSessionId: 18f11716-6252-4229-81f7-dc1a12808b66
  modified: 2026-08-03T11:10:11.823Z
---

Working on landing-page (marketing) changes on branch **`feat/landing-dark-light-card`** (local only — user said do NOT push; verify locally + batch more changes first).

**Done + committed on the branch (commit 1112a97):** the "Mod dark & light" feature card in the `#functii` section of `src/app/(marketing)/page.tsx` now shows the real client-plan screen in BOTH themes side by side — two phones (dark left, light right), rounded + subtle border + shadow. Only that one card is special-cased (has `imgLight`); the other 3 cards keep a single dark mockup. New assets: `public/images/feature-theme-dark.png` + `feature-theme-light.png`, new SCSS classes `.featureThemePair` / `.featureThemePhone` in `src/app/(marketing)/styles.module.scss`. Verified: type-check + lint + build green + local visual capture.

**CORRECTION:** the marketing page is NOT hardcoded — it is FULLY token-based (`var(--background)`, `--foreground`, `--bg-surface`, `--neutral-*`, `--text-*`). Theme = token-remap: `:root` in `globals.scss` is dark by default; `[data-theme="light"]` remaps all color tokens. Applying `data-theme="light"` to a `<section>` cascades cleanly (verified). CLAUDE.md's "marketing hardcoded / dark mode not implemented" notes are STALE.

**Committed 2026-07-30 as `cd83842` on the branch (user approved; branch still NOT pushed). code-reviewer + design-reviewer both ran = 0 critical; lint/type-check/build green. Contents:**
- **Roadmap (`#ce-urmeaza`) + FAQ (`#intrebari-frecvente`) → light sections** (user chose ONLY these two white after rejecting a full alternating-bands + gradient-seam experiment — that was tried and fully reverted via `git checkout`). Each: `data-theme="light"` + `.fullBleed` (full-bleed so the dark page bg doesn't gutter on wide screens) + white `--bg-surface` card elevation. Added light-theme variants of `--roadmap-accent-purple`(#7C4DC4)/`-amber`(#B45309) in globals `[data-theme="light"]` (the `:root` "bright on dark" values were too pale on white). FAQ "Contactează-ne" button knockout → `var(--primary-black)` (was `--background` = near-white on the gradient → illegible in light). design-reviewer caught the gutter + knockout issues; both fixed.
- **All landing app images replaced with the NEW screenshots** (user: old ones outdated). Current images were composed/framed mockups; new captures are raw single screens, so NOT a file-swap — rebuilt each slot with CSS frames (reused `.featureThemePhone`/`.featureThemePair`; added `.heroWindow` for desktop dashboards; `.featureImage` → object-fit cover/top). New files `public/images/lp-*.png` (copied from `marketing/screenshots/`, DARK variants since sections are dark): hero=dashboard-home; client section=client home+measurements phones; feature cards = dashboard-clients (window) / client plan+progress (phones) / dark&light card (unchanged) / dashboard-client-detail (window). Old mockups (`hero-image`, `your-clients`, `card-1/2/3/4-features`.png) now ORPHANED — delete at commit after checking no other refs (OG etc.).

**Screenshots for review:** system Playwright chromium build was incomplete; drive the installed Chrome instead — Playwright `chromium.launch({ channel: 'chrome' })`. Marketing page has scroll-reveal, so `fullPage` comes up blank below the hero — scroll through / use per-element `scrollIntoView` + element/clip shots.

**Section-by-section polish pass (2026-07-30/31), COMMITTED `da9f84e`:** Benefits cards 2&3 reworked into distinct pillars (control · growth; new icons); "Ce urmează" replaced with the 4 real roadmap cards (Calendar/Clase/Nutriție/Comunicare, all "Coming soon"; dropped Web Dashboard + Explore; grid widened 1024→1356); FAQ given a muted `--neutral-300` band (distinct from the light Ce-urmează above); nav pill widened 1200→1356 to line up with content; Hero rebuilt = big dashboard window (~1.2×) + client phone floating over its top-right corner rising to the CTA level, punchier description, CTA diacritics fixed. Removed 2 orphaned roadmap SVGs. Also swept in `docs/plans/admin-dashboard-v1.md` (was untracked). Roadmap feature spec at `docs/plans/roadmap-features-v1.md` (see [[roadmap-features-plan]]). Hero image layout lives in `.heroImageWrap`/`.heroWindow`/`.heroPhone`.

**FAQ content refresh — COMMITTED `2924bb2`:** merged the 2 payment Qs into one clear "Clienții mei plătesc ceva pentru Velto?" (clients pay nothing); dropped "când se lansează"; reframed "pot testa" → "Pot încerca Velto gratuit?" (Free plan, max 2 clienți); removed the stale Explore ref from the referral Q; added 4 (nutriție, securitate date, import, limită clienți). Also committed `docs/plans/pricing-monetization-v1.md`.

**✅ MERGED TO MAIN 2026-08-03:** branch fast-forwarded main `094c241 → 2924bb2` and pushed to `origin/main`. All landing work (cd83842, da9f84e, 2924bb2) is now on main. Branch `feat/landing-dark-light-card` is fully merged (== main); can be deleted (local + remote) — not done, offer to user. Earlier DB/workout commits were already on main.

**Open / next:**
- ⚠️ Landing is HALF-PIVOTED on pricing: FAQ now says Free/Growth (launched), but hero + CTA section still say "listă de așteptare / lansare în curând / 2 luni gratuite". User deferred the pricing pivot — reconcile when doing it (see `docs/plans/pricing-monetization-v1.md`; real tiers = Free/Plus/Max, "Growth" copy is stale but currently consistent).
- OG/social image (`layout.tsx`) still the old `hero-image.png` mockup — decide keep vs swap.
- Section-by-section pass still to do: "Experiență modernă pentru clienți", "Funcții" (copy — images already refreshed), "Cum funcționează", CTA/waitlist (= pricing pivot), footer.
- OG/social image (`src/app/layout.tsx` openGraph + twitter) STILL points to old `hero-image.png` (composed mockup, old UI) — kept so the OG doesn't break, but user needs to decide keep vs swap to a new screenshot.
- CLAUDE.md route table + gaps still say marketing is a placeholder / dark mode not implemented — STALE, worth correcting.

Screenshots are captured from production via `npm run capture:marketing` (script `scripts/capture-marketing.ts`, creds in gitignored `.env.capture.local`). CAPTURE_LIGHT=1 adds a `-light` set. Output → `marketing/screenshots/` (app-*=780x1688 phones, dashboard-*=2880x1800).

STANDING: never commit `docs/database/spartan_recomp_foundation_v1_seed.sql`.
