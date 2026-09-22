---
name: landing-light-redesign
description: Branch feat/landing-light — landing redesign section by section. Hero/beneficii/client/funcții/pași/prețuri/Ce urmează DONE; FAQ + footer left.
metadata:
  type: project
---

> TERMINAT SI IN MAIN. Verificat 2026-09-02: `git log main..feat/landing-light`
> e gol, iar `af4807d` se numeste "finish the landing - Ce urmeaza, FAQ, footer
> and the CTA washes". Sectiunile listate mai jos ca "next" sunt toate facute.
>
> ATENTIE: nota veche a costat deja. Pe 2 septembrie doi agenti au refuzat sa
> consolideze `FREE_SEATS` si `TRIAL_DAYS` in `(marketing)/page.tsx` fiindca au
> citit aici ca fisierul se rescrie pe o ramura. Nu se rescrie. Verifica ramura
> inainte s-o crezi.

**STATUS 2026-08-28: COMPLETE, MERGED AND LIVE.** `main` fast-forwarded over
`feat/landing-light` (33 commits, `b1ff1ae..93611ef`) and pushed; Vercel built in
~2 min. Verified on the live domain: new title, description, canonical, the
generated OG card (133 KB, was 5.7 MB), and `X-Robots-Tag: noindex` on
app./dashboard. but NOT on veltofit.app.

That push also shipped the whole feat/pricing-and-billing stack — billing UI,
health-notes consent, archive-client UI, legal rewrite — because this branch was
cut from it. Checked before pushing: the two unapplied migrations
(`trainer_private_notes.sql`, `accept_invite_seat_cap.sql`) have NO code
referencing them, and `my_billing_state()` is applied in prod, so nothing
depended on missing schema. See [[backlog-launch-blockers]], which is now partly
stale: the "Începe gratuit lands on sign-in" item is DONE.

Landing redesign in progress, decided 2026-08-26. Approach the owner chose: first flip the
existing page to the light theme (DONE), then rebuild it **section by section**, in this order:
hero → beneficii → partea clientului → funcții → cum funcționează. Pricing is already rebuilt
and is NOT part of this pass.

Branch `feat/landing-light`, cut from `feat/pricing-and-billing`. Two commits, working tree clean:
- `e51d235` flip to light (`data-theme="light"` on the `<main>` — the page is fully token-based,
  so one attribute flips it)
- `aeb3441` hero rebuild (see below)
- `87922a7` light captures (`lp-dashboard-home-light.png`, `lp-client-home-light.png`,
  `lp-client-measurements-light.png`)

**Reference direction**: a Claude Design artifact the owner picked, light theme, at
`https://claude.ai/code/artifact/8f6ce65f-b7c4-4f8f-a6ff-c8289c5a3645`. He wants its structure
and section elements, not its exact copy. My own four artboards were rejected ("urate directii") —
do not resurrect them.

**⚠️ TWO CONTAINER SYSTEMS ON THIS PAGE — the page-wide bug the hero exposed.**
`.section` caps at `max-width: 1356px` and THEN takes the 104px gutter out of it
→ **1148px** of content. The full-bleed sections (`.fullBleed`, `.benefitsSection`
— i.e. `beneficii`, `functii`, `ce-urmeaza`, FAQ) take the gutter off the viewport
first and cap the INNER wrapper → **1232px**. Above a 1356px viewport they diverge
by 84px. Their code comments claim they "re-apply the content max-width so layout
matches other sections" — **that comment is wrong, it has never matched.**
Measured at 1440: nav logo L=75, benefits cards L=104, old hero L=146.
Nothing showed it while every section was centred; a left-aligned hero does.
Hero fixed to the full-bleed order in `3bf10fb`; `--lp-max: 1356px` sits next to
`--lp-gutter` on `.page`. **`cum-functioneaza` + `preturi` + the client section
still use the narrow `.section` order — reconcile as each is rebuilt.**

Trap found the same day: a negative margin does NOT bleed a box that has
`width: 100%` — the width is pinned, so it only narrows the margin box. A bleed
claimed in `14d38ac` was silently doing nothing.

**✅ CLIENT · FUNCȚII · CUM FUNCȚIONEAZĂ · PREȚURI DONE** `e8c8186` (+ pricing
decisions in `0c50fa2`). Remaining: CTA/înregistrare, FAQ, footer.

**✅ CE URMEAZĂ DONE 2026-08-28 (uncommitted).** Rebuilt from artifact `8f6ce65f`:
eyebrow "În lucru" → "Ce urmează?" → lede, left-aligned; `.roadmapSection` uses the
settled container order (gutter off the viewport, `--lp-max` on the inner wrapper).
Cards are now **dashed 1px `--border-medium`, `--radius-3xl`, no shadow** — an
outline of something not filled in — and the English "Coming soon" grey pill is
replaced by the artifact's status line: 6px `--warning` dot + "Nu e disponibil
încă", pinned bottom with `margin-top: auto`. **One deviation from the artifact:
it colours that label `--text-tertiary`, which is 2.6:1 on white — used
`--text-secondary` instead.** Two columns kept (not the artifact's auto-fit), the
nutrition card is three sentences and would set as a ribbon in a quarter column.
The dead `.availableBadge` / `.roadmapIconImg` rules are gone with it.

Same day, in the CTA: the third assurance "Clientul are aplicația lui, tu ai
panoul tău" → **"Tu și clientul tău, în aceeași aplicație"**. Owner's steer:
splitting it in two reads as handing work off; the point is that both are in the
same place — the "întărești relația cu clientul" claim from
[[velto-positioning-statement]] that the page still does not make elsewhere.
Also in the CTA: the card's single blue radial was replaced by the hero's PAIR
(`.ctaGlowGreen` / `.ctaGlowBlue`) — solid circles blurred, not gradients. First
pass put them inside the card; **the owner meant the SECTION background**, so
they now sit on `.ctaSection` (position: relative) behind the opaque card, green
bottom-left, blue top-right, both pinned to `left: 0` / `right: 0` so nothing
bleeds past the right edge. The card lost its `overflow: hidden` with them. The
rest of the CTA section is still unrebuilt (old container order).

**Pricing header, 2026-08-28:** "Plătești pe măsură ce crești" → **"Alege planul
potrivit pentru businessul tău"**, lede → "Începi cu 30 de zile gratuite, cu
clienți nelimitați. Fără card." Owner asked for Trainerize's framing (a chooser,
not a claim about the model). **Note the replaced title is listed as verbatim-keep
in `docs/plans/landing-redesign-prompt.md`** — that brief is now out of date on
this line. The heading wraps to "...businessul / tău" on its own, so
`.pricingTitle` got `text-wrap: balance`; hard `<span>` breaks are NOT safe here
("pentru businessul tău" at the mobile size is 357px vs a 358px column at 390).

**✅ FAQ DONE 2026-08-28 (uncommitted).** Artifact structure: heading column left
(eyebrow → "Ce te mai ține pe gânduri?" → a lede that carries the mailto), the
questions right as a **divided list** — each row draws its own `border-top`, the
list closes with a `border-bottom`, so rules never double. The bordered cards and
the separate "Mai ai întrebări? / Contactează-ne" gradient button at the bottom
are gone; the contact route now sits WITH the heading, not below eleven closed
questions. All 12 questions moved into a `FAQ` array that splices `...PRICING_FAQ` in the
middle so the price answers still have one home. Copy unchanged except one, the
owner's call: **"Ce se întâmplă în ziua 31?" → "Cum funcționează cele 30 de zile
gratuite?"** (artifact's wording) — a question someone asks BEFORE starting, not
a worry about the end. Its answer now opens with the trial (no card, 30 days,
unlimited clients) and keeps the verified ending (nothing is deleted; over the
cap you keep every client but cannot add more).

**"Pot schimba planul mai târziu?" rewritten the same day** — the old answer sold
the MECHANISM ("ne scrii pe email, plata pe bază de factură, niciun card salvat"),
all of which dies the day Stripe lands. It now answers WHEN a change takes effect:
extra seats live immediately on an upgrade, a downgrade lands at the end of the
paid period (which is the policy `docs/plans/billing-operations-v1.md` §D already
chose — Customer Portal, cancel at `current_period_end`). **Still stale on the
same assumption: "Cum se face plata?" says "pe bază de factură".** With Stripe the
card is charged and Stripe issues the invoice — owner's call whether the landing
describes the interim or the Stripe flow.
`FAQItem`'s `headingLevel` prop was dead (no call site) — removed.

**Dead SCSS removed with it:** `.sectionHeader` `.sectionTitle` `.sectionSubtitle`
`.fullBleed` `.bandMuted` — the FAQ was the last user of the centred-header /
full-bleed-band pattern. `.section` + `.sectionContent` STAY: Beneficii is still
on them. Verified Beneficii unchanged after the delete.

**✅ FOOTER DONE 2026-08-28 — the landing rebuild is complete.** Light like the
rest (it carries `data-theme="light"` ITSELF: it sits outside `<main>`, so it was
inheriting the root DARK palette — that is why it was a dark slab under a light
page). Brand + tagline left, three link columns right (Produs / Legal / Contact —
the old Social column became Contact and holds the email + Instagram), legal PFA
block and the ANPC/SOL badges under a hairline.

**⚠️ THE CONTAINER VARS WERE ON `.page` ONLY.** `--lp-gutter` / `--lp-max` were
declared on `.page`, so the footer — a SIBLING of `<main>`, not a child —
resolved them to nothing and ran edge-to-edge. They are now a `@mixin
landing-metrics` in the module, included by both. Anything else added outside
`<main>` needs the same include.

Tagline replaced: "Platforma hibridă care permite antrenorilor…" → "Clienții,
planurile și progresul lor, într-un singur loc. Construit pentru antrenorii
personali din România." (his own triad, per [[velto-positioning-statement]]).

**Trick for testing narrow widths when the extension will not resize the window**
(it silently keeps outerWidth): inject a 390px `<iframe src="/">` into the page
and measure inside it — viewport media queries key off the iframe's width. At 386
the page has **zero** horizontal overflow now, so the old 390px pricing-table
overflow bug is gone with the table.

**✅ CTA TARGETS SPLIT 2026-08-28.** Every "Începe gratuit" pointed at `/login`,
i.e. a sign-in form asking for a password the visitor had never set. Now:
`?mode=signup` on all of them, plain `/login` only for "Intră în cont". The
landing keeps ONE desktop-vs-mobile decision (`authUrls = {signup, login}` off
the same matchMedia); `src/app/login/page.tsx` compares `searchParams.get('mode')`
and passes `initialMode` to `AuthForm`, which grew that optional prop (default
'login'; a pending invite still forces signup). The value is compared, never
passed through.

⚠️ **Not seen signed-out.** All three local origins (root, app., dashboard.) had
live sessions, so `/login` redirected every time; signing the owner out to look
was not worth it. Landing hrefs were verified in the DOM. Confirm the signup form
opens in an incognito window.

**⚠️ PRICING CHANGED — read `docs/plans/pricing-monetization-v1.md` first.** It now
opens with "Before you change any price": the per-client cost must never rise as
capacity grows, with a runnable check. Plus is **15 seats** (was 20 — at 20 it was
cheaper per client than Max's first steps, which is backwards). Max is a **six-step
slider**: 30/50/75/100/150/200 at 195/299/399/499/699/899. The comparison table and
the "over the ceiling, write to us" branch are both gone.

**Landing patterns now settled** (apply to the remaining sections, do not re-derive):
- Container: **gutter off the viewport FIRST, cap on the inner wrapper**. Four
  sections had it backwards and sat 41px further in. Still wrong in the CTA
  section and the FAQ — fix as each is rebuilt.
- Every section: eyebrow (uppercase, `--text-secondary`; green only on the client
  band) → title → lede. Left-aligned, never centred.
- Screenshots are **cropped, not scaled**, and crop offsets are **percentages** —
  in px they break the moment the container width changes.
- Roadmap features under a price need BOTH a group heading saying they do not
  exist and a hollow marker instead of a tick.

**✅ BENEFICII DONE** `135f66c` — left heading + hairline list + ghosted 01/02/03,
from artifact `8f6ce65f`. Copy traps learned: (1) benefit-level titles, not
mechanics — "Planul se scrie o dată" is a feature, "Mai mulți clienți, cu mai
puțin efort" is a benefit; (2) **"chiar" smuggles in an accusation** ("clienții
tăi *chiar* urmează planul" blames someone for today); (3) **reflexive passive
("planurile se refolosesc") is the register of official Romanian** — the owner
reads it as plastic every time. Next: partea clientului.

**✅ HERO DONE — commits `aeb3441` `14d38ac` `3bf10fb` `e25a7b3` `7e791f7`
`6dd4613` `b2dfd45` (2026-08-26/27).** Final shape: badge → 2-line headline
(68px desktop) → subhead → CTA + reassurance → 3 facts | dashboard capture right,
even halves, no chips, no disclaimer. **Owner rejected, do not retry:** cropped
UI fragments (A/B/D experiments), 3 floating notes over the shot ("prea
aglomerat" — hero already carries a lot of text), the product bleeding past the
content edge ("arată urât ieșită").

**⚠️ IMAGE TRAPS learned the hard way, all cost real time:**
- `sizes` describes the width the `<img>` is PAINTED at. Under-declare it and
  the browser fetches a thumbnail and stretches it — that was the "pixelat".
- `quality={90}` is REJECTED (400) — next.config declares no `images.qualities`.
- Next serves stale optimized images from **`.next/dev/cache/images`** (NOT
  `.next/cache/images`). Change a PNG → purge that dir AND restart dev.
- A negative margin cannot bleed a grid item that has `width: 100%`.

**Hero — earlier state:** The first pass came back
"arata un pic ciudat, e mai mic decat" the reference — correct: I had sized the
headline from a GUESS at Sora's width and capped it at 44px. **Measured: the
longest line ("Bifează fiecare serie.") renders at exactly 10× the font size.**
So the max non-wrapping size is one tenth of the column — use that, never an
estimate. Now fluid via `clamp()` per layout, 64px on desktop (raised from 60 once the container fix widened the copy column from 594px to 710px).

**How to read the reference artifact's real CSS** (do this instead of guessing):
`WebFetch` the artifact URL — it saves the full bundled HTML to the session's
tool-results dir. The page's actual markup is JSON inside
`<script type="__bundler/template">`; extract and `json.loads` it. The reference
hero: `clamp(38px,5.6vw,68px)` / lh 1.03 / tracking -1.8px, 1240px container,
40px gutters, product with `margin-right: clamp(-160px,-6vw,0px)` so it bleeds
off the edge, facts inline in the copy column, blue `--primary` CTA.
NOTE: at 1440 its own h1 would wrap — it is a generated design, not engineered.

**Hero — first pass, superseded:** Split layout (copy left /
product right from lg, stacked below), eyebrow pill, the three-line headline
below, subhead, CTA + reassurance side by side, the dashboard capture with the
client phone overlapping its bottom-right corner, an example activity-feed chip
at the bottom-left stepping into the grid gap, and a three-fact strip underneath
("0 lei / atât plătesc clienții tăi", "Fără App Store / se instalează direct din
browser", "Web + telefon / tu pe desktop, clientul pe telefon"). Gate green,
verified at 390 / 768 / 1100 / 1440.

The line breaks in the headline are structural, not styling: each sentence is a
`<span>` set to `display: block` at EVERY width. Measured — the longest sentence
is 320px at 32px Sora bold, so it holds down to a 360px phone. Letting them wrap
naturally turns the loop into prose and loses the argument.

**Type scale — FIXED.** `--font-size-display-sm` (36px) / `-md` (44px),
`--line-height-display`, `--tracking-display` (em, so it holds across sizes) and
`--tracking-eyebrow` now live in `globals.scss`, with `display-1` + `eyebrow`
mixins in `_mixins.scss`. 44px is the cap because the 1356px container leaves
1148px of content, so the copy column stops growing past lg. Also added
`--lp-gutter` on `.page` (16 / 32 / 104px) — the other sections still hardcode
104px and should adopt it as each gets rebuilt.

**Hero headline — REPLACED 2026-08-26 (`e25a7b3`), current:**

> Mai mulți clienți.
> **Aceleași ore.**   ← accent on line 2
> Îți administrezi clienții, planurile și progresul lor dintr-un singur loc.
> Fără Excel, fără agendă, fără PDF-uri pe WhatsApp.   (`7e791f7`)

**How the owner describes Velto, in his own words — use this framing:** "practic
aplicația asta face, îi ajută să administreze mai ușor clienții, planurile și
progresul lor." He is comfortable with the word "administrare" even though it is
slightly bureaucratic; the triad **clienți · planuri · progres** is his, and it
has now survived three rewrites. The replaced-tools list is **Excel · agendă ·
WhatsApp** — the paper agenda matters, a lot of this audience is still on it.

Rejected subhead, and WHY (do not reintroduce): "Faci planul o dată și îl dai
mai multor clienți" — it explains the headline by implying everyone gets the
same plan, which for a trainer reads as the accusation he is trying to avoid.
Never explain efficiency with anything that sounds like cookie-cutter coaching.

Accepted trade-off: "dintr-un singur loc" says WHERE the work happens, not why
it takes less time, so "Aceleași ore." is asserted, not explained. **The
benefits section is where that argument has to land.**

Owner rejected the previous one ("nu-mi place directia si cum suna, nici tonul").
The diagnosis that landed with him: **"Bifează fiecare serie" is an imperative**,
so the trainer reads it as an order to himself when it describes his client —
subject jumps you → him → you. And **"raportul" is not a gym word** ("văd ce a
făcut", not "văd raportul"), which is where the corporate tone came from.
Brainstormed 6 directions; he picked the shortest, business-benefit one.
Rejected directions, do not resurrect: "Nu mai întrebi", "Planul tău ajunge în
telefonul lui", "Fără Excel/PDF-uri" as the HEADLINE, sec/descriptive.

**⚠️ ROMANIAN LEADING.** `--line-height-display` was 1.03, copied from the
reference. Romanian sets **ș and ț with a comma BELOW the baseline** — at 80px
the comma under "mulți" landed on top of "Aceleași". **1.12** is the floor.
Never copy a Latin-only design's display leading for RO copy.

**⚠️ THE 8.15× RULE IS PER-STRING.** "Mai mulți clienți." sets at 8.15× the font
size; the old "Bifează fiecare serie." was 10×. The clamp() expressions in
`.heroH1` are derived from whichever headline is live — **re-measure in the
browser if the copy changes.** Now 34px floor → 80px cap.

**Hero headline — superseded, do not restore:**

> Îi dai planul. Bifează fiecare serie. Tu vezi raportul.
> Clienții, planurile și progresul lor — într-un singur loc. Fără Excel, fără PDF-uri trimise pe WhatsApp.

It came from asking him to describe Velto out loud, and his answer had a better shape than the
artifact's headline did: a **loop** (you assign → client logs rep by rep → you get the report).
The artifact's headline only claimed "clients will actually open it", which is the middle of the
loop, not the payoff. His most concrete phrase was "fiecare repetare, fiecare greutate" — that
detail is what separates Velto from a PDF, and it belongs in the copy. See [[landing-dark-light-branch]].

**Open on the hero, for the owner to call:**
- The desktop dashboard capture is unreadable at phone width (~7px text). Inherited from
  the old hero, not new. A mobile-only composition (phone alone) would fix it.
- That capture is the demo account — it shows "22 CLIENȚI / 26 PLANURI". A real product
  screenshot, but a visitor reads those as Velto's numbers.
- CTA pill is `--foreground` (near-black), not `--primary`. The reference uses blue for
  BOTH the nav and hero CTAs; left alone because `.linkButton` is shared with the nav and
  pricing, so it is a page-wide decision. **This is the last visible difference from the
  reference — ask before the next section.**
- 🐛 **The pricing comparison table overflows the viewport at 390px** (scrollWidth 526 vs
  390) — a horizontal scrollbar on the whole page. Pre-existing, in the pricing section,
  NOT the hero. Needs an `overflow-x: auto` wrapper.
- ~~Pre-existing dead SCSS: `.availableBadge`, `.roadmapIconImg`~~ — both removed
  with the Ce urmează rebuild.

**How he wants copy written**: plain, spoken Romanian. He rejected a subhead containing
"notează ce a ridicat" as "plastic, cam scris de AI". Test every line against "would a trainer
say this out loud?" See [[feedback-plain-language]].
