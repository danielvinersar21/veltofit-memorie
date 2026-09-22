---
name: pwa-viewport-shortfall
description: "2026-09-04 — the installed PWA on app.veltofit.app gets 793px of an 852px screen, so every bottom-anchored bar stops 59px short. The fix is written and pushed but UNVERIFIED in the shell that lies; what was eliminated, and why prod could not be used to check."
metadata: 
  node_type: memory
  type: project
  originSessionId: e4cdbcd6-0f93-4d8c-8bd6-f0a403e11d6d
  modified: 2026-09-04T15:03:59.693Z
---

The owner saw a band of empty page under the tab bar in the installed PWA.
Measured from his screenshots, pixel by pixel: the pill's lower edge sits
**114px above the screen bottom where the CSS asks for 54** (34 safe area + 20).

## The cause, measured twice in one screenshot

The app's content stops at **793** and the bar's lower edge is at 793 − 54.
The screen is 852. So the installed shell hands the page a fixed-positioning box
59px short — exactly the status bar — while painting all 852. The bar was never
wrong; it anchors correctly to a number that lies.

## Eliminated, each on the device, none of them the cause

`apple-mobile-web-app-status-bar-style: black-translucent` · `overflow-x: hidden`
making body the scroller · the manifest shell (display-mode standalone vs a
plain shortcut) · deleting and re-adding the icon · a redirecting `start_url`
(the app's manifest says `/`, which the middleware sends to `/login`).

A static replica of the bar, and later the REAL `<TabBar>` inside a copy of the
`(app)` layout, sit correctly in every one of those. **The same code on a Vercel
preview origin reports 852 and a shortfall of 0.** Whatever the trigger is, it
lives on the `app.veltofit.app` origin and cannot be reproduced anywhere else.

## The fix — pushed on `feat/billing-launch`, NOT verified where it matters

`--viewport-shortfall`: an inline script in `layout.tsx` measures the box
`position: fixed` actually anchors to against the visible viewport and writes the
difference to `documentElement`. Everywhere the two agree — every browser, every
desktop — it is `0px` and every `calc()` is the value it was. Verified both ways
in a browser: measured 0 changes nothing; forced to 59px it moves the bar down
by exactly 59.

Applied to all seven bottom-anchored surfaces (tab bar, active-workout action
bar, plan-detail CTA, both password footers, modal + drawer backdrops, the
archive wall's sticky footer) and to containers that CENTRE content or paint
their own background. Deliberately **not** on `html, body` or `.root`: the
canvas already paints that strip, so it bought nothing and made every page
scroll 59px for no reason.

**It cannot be confirmed until it runs on `app.veltofit.app`.** That needs
`origin/main` to move, and local `main` is 25 commits ahead of it — so a deploy
today would ship far more than this. Owner chose to park it: the symptom is
cosmetic, and the fix rides along when billing ships. See
[[billing-on-the-phone]].

## Two traps this cost hours to learn

1. **A pixel detector can lie confidently.** A sloppy white-run filter caught a
   card at the screen bottom instead of the tab bar and produced a clean,
   plausible "the reinstall fixed it — 59px". The owner said it had not changed;
   he was right. When measuring from a screenshot, verify the detector found the
   element it claims (width and height against the CSS), not just that it found
   something.
2. **iOS bakes web-app config at Add-to-Home-Screen time.** `viewport-fit=cover`
   entered the code on 2026-08-27 (`78b6b23`); an icon installed before that
   keeps the old behaviour and a force-quit does not re-read it. This was
   demonstrated live — adding a meta and relaunching changed nothing until the
   icon was recreated. It was a real effect, just not the cause here.

Deleting a route while `npm run dev` is running leaves stale generated files in
`.next/dev/types/`, and the pre-push gate fails on them. `rm -rf` the stale
directory; they regenerate.
