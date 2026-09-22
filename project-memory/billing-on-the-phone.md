---
name: billing-on-the-phone
description: 2026-09-03 — the app host got a billing screen (finding V8), Stripe now returns to the host you paid from, and four defects the owner found on his own phone. The webhook timing was MEASURED on 2026-09-05 and the two poll constants stay as they are.
metadata:
  type: project
---

A full day on V8: a trainer on a phone had NO billing page at all. Branch
`feat/billing-launch`.

## Shipped and proven

**Confirmed by the owner on his own phone:** paid 195 RON from the phone and came
back INTO the app, logged in. That path ended on a login screen the same morning.
The portal's `return_url` was seen in Stripe's own event data pointing at
`app…/trainer/profile/billing` — the fix verified from their side, not ours.

The screen is now ONE component (`features/billing/components/billing-screen/`)
rendered by both hosts; the pages are a container plus the Suspense boundary.
`sameHostUrl(origin, {dashboard, app})` decides every return address — the caller
MUST name a path for both hosts, which is what makes the old bug (a real origin
paired with a dashboard-only page) unspellable.

## Four defects the owner found by looking, that no test could have

1. `.securedBy`'s card icon sat orphaned in the corner. Cause: `globals.scss`
   resets `svg { display: block }`, so switching the row from flex to inline flow
   was NOT enough — it needed `display: inline-block` on the icon. Got this wrong
   once in front of him; check the global reset before assuming inline flow.
2. `.fact` had no `flex-wrap`, so at 390px the trial row broke the DATE between
   day and month. `min-width: 0` is inert here — the floors are never reached.
3. "Clienți activi — 2 din 2 clienți activi" said it twice. `formatSeatLimit` now
   returns a bare number; `describeSeatLimit` keeps the noun for its nine
   sentence callers. The two deliberately disagree.
4. "Velto Free" wore an "Anulat" chip. Free is not something you cancel.

## Two bugs found by tests and by the owner, both real

**The poll never re-armed.** The recheck effect depended on a BOOLEAN
(`stillChecking`) that is `true` on attempt 1 and still `true` on attempt 2, so
the dependency array never changed and React never re-ran it. One check, then
silence — and since the counter never reached its budget, the honest "it is
taking longer" message never appeared either. The trainer sat on "îți actualizăm
contul" for ever. Fix: the COUNTER must be in the deps. `exhaustive-deps` cannot
ask for it — the body increments through a functional updater and never mentions
it. Mutation-checked both ways.

**`overflow-x: hidden` on `html, body` cut the page in half.** The clause nobody
remembers: one axis `hidden` + the other `visible` makes the other `auto`, so
`body` became a SCROLL CONTAINER of window height. Chrome on iOS re-renders a
restored tab at its old height and never recomputes an inner scroller — the fixed
tab bar floated at ~70% of the screen, the sticky top bar hid above the visible
area, a reload fixed it. `overflow-x: clip` clips identically and creates no
scroll container. Owner confirmed fixed. Does NOT affect the installed PWA
(`display: standalone`, no browser bar to collapse) — it was only ever the
browser-tab experience, i.e. everyone who has not installed yet.
🔴 Four more `overflow-x: hidden` on large dashboard containers carry the same
trap. Harmless there (no collapsing browser bar on a laptop) — left alone.

## State at close of 2026-09-03

- ✅ SUPERSEDED 2026-09-04: all four lots were committed separately (owner asked
  for the split) and the branch is PUSHED. `origin/feat/billing-launch` now
  carries the billing work plus the login spacing fix and the viewport
  compensation — see [[pwa-viewport-shortfall]]. `origin/main` still untouched,
  so production has changed nothing since 30 August.
- Gate green: 904 tests, lint, type-check, build.
- Owner was offered the commit split and closed the session without answering.
  **Ask first thing.** See [[backlog-memory-has-no-backup]] — the same
  one-laptop risk he acted on that morning.

## Open, in the order they matter

1. ✅ **ÎNCHIS 2026-09-05 — numărul e măsurat, iar cele două valori RĂMÂN.**
   Nu pentru că ar fi nimerit durata, ci pentru că pe drumul normal nu se
   folosesc deloc (zero reverificări, 123 ms) — iar pe drumul anormal nicio
   valoare n-ar ajuta: prima reîncercare a lui Stripe vine abia după ~1 minut,
   deci **nicio fereastră de 16 s nu poate prinde o reîncercare**. Alegerea
   corectă nu e „mai lung", ci „renunță repede și spune adevărul". Comentariul
   din cod nu mai zice „e o judecată, nu o măsurătoare" — conține măsurătoarea.
   Cifrele: [[stripe-webhooks-need-cli-locally]].
   🔑 Și bug-ul re-armării e CONFIRMAT reparat în aplicația reală: oprind
   `stripe listen` la mijlocul unei plăți, cele opt reverificări au pornit la
   2016 ms fix, iar mesajul onest a înlocuit spinner-ul la 16,2 s.
2. A "verifică din nou" button on the slow message — `getPortalReturnCopy`
   already does exactly this for the same webhook lag. **Rămâne deschis**, dar
   e acum o comoditate, nu o gaură: textul de renunțare cere o reîncărcare de
   pagină, care chiar e soluția reală.
3. ✅ **ÎNCHIS 2026-09-05** — `stripe listen`, procedura de test și recuperarea
   unui webhook pierdut sunt scrise în `docs/plans/billing-operations-v1.md`,
   secțiunea E2.
4. `BackLink` on the four client profile subpages (same gap, four one-liners).
5. Skeleton is ~344px against a ~1054px first paint on a phone — wants measuring
   on a device, not estimating from CSS.

`BackLink` uses a FIXED href, not `router.back()`: after a payment, history holds
Stripe's consumed Checkout Session, so "back" walks out of the app.
