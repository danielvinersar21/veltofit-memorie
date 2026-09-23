---
name: e2e-suite-and-db-upgrade
description: Playwright e2e suite built 2026-08-10/11 on branch test/e2e-suite (UNCOMMITTED). It found 3 real app bugs (fixed) and proved the DB was the binding constraint — Supabase upgraded Free/NANO → Pro/MICRO.
metadata: 
  node_type: memory
  type: project
  originSessionId: 942ae820-f0a5-4369-aaa1-70945c3bdfd1
  modified: 2026-09-23T14:40:42.959Z
---

Built on branch `test/e2e-suite`, **still uncommitted**. Includes app fixes in `src/` that belong in main regardless of the suite.

## The suite
**Recounted 2026-09-22: 8 specs, 17 runnable tests + 1 `test.fixme`, plus 3 setup steps and 1 teardown** (the older "20 tests" figure counted differently). The superset work on 2026-08-15 added four specs:
`client/superset.spec.ts` (grouped workout + finishing one — nothing had ever completed a
workout before), `trainer/superset-builder.spec.ts`, `trainer/superset-copy.spec.ts` (clone +
copy-week), `trainer/plan-wizard.spec.ts` (the whole build-a-plan flow + publish guard, which
closes the old TODO below). Helpers added: `support/plan-fixture.ts` (seeds a prefixed plan
over PostgREST, reads it back) and `support/superset.ts`.

Original 13: auth gating (3), client home progress (2), workout persistence (2), trainer
chrome + publish guard (2), plus setup/cleanup. `npm run test:e2e`.

**Assert against the STORED rows, not the screen**, for anything that writes. Local state
renders happily whether or not the write landed — that is how the superset toggle's silent
failure and the wizard's silent add-failure were both caught.

**Two hard-won config facts:**
- **Must run against a PRODUCTION build**, not `next dev`. StrictMode double-invokes effects, so the first `load()` bails via `ignore` and never sets `restoreDoneRef` — which disables the sessionStorage auto-save. The specs then fail on behaviour real users never see. Config runs `npm run build && next start -p 3100`.
- Test timeout raised to 90 s: the workout flow does three full page loads, and the 30 s default expired mid-flow and reported it as a missing value — indistinguishable from data loss.

**Data lifecycle:** `e2e/support/reset.ts` resets `testclient` (deletes sessions + set performance, plan back to W1D1); `e2e/support/fixtures.ts` deletes anything created, matched on the `[e2e]` title prefix AND the test trainer, both conditions always. Both refuse to touch a non-`is_demo` account. Verified adversarially: a plan named "Plan REAL nu sterge" survived the sweep.

Accounts: `testtrainer@veltofit.app` / `testclient@veltofit.app` ("Alex M."), password `parola`, both `is_demo`. Vitest is now scoped to `src/` — without that it collects the Playwright specs and reports 4 failed files.

## Three app bugs it found (all fixed)
1. **`load()` in use-active-session had no error handling.** Any rejection abandoned it mid-flight: the workout stayed on screen, but the set restore never ran and `restoreDoneRef` stayed false, silently disabling crash recovery. Now caught and surfaced.
2. **The load effect depended on the `user` OBJECT**, which `fetchUser` replaces on every auth refresh (incl. the 30 s notification poll) — so a refresh cancelled the in-flight load. Now `user?.id`. This was code-review warning 3, deferred as "just perf". It was not.
3. **The workout screen fetched the ENTIRE plan (~21–30 kB) to render one day.** Replaced with `getPlanStructure` (day grid + exercise counts, ~2 kB) + `getPlanDayExercises` (one day). New cache keys `plans:structure:` / `plans:day:`.

## The DB was the real constraint
Measured with the test client's own JWT (RLS included), idle: heaviest query 539 ms — nowhere near a timeout. Yet under suite load Postgres returned **"canceling statement due to statement timeout"**, and the app showed that raw message to the user.

Diagnosis: **Free plan = NANO (`t4g.nano`, SHARED CPU, 0.5 GB)**, RAM 54% at idle, CPU averaging 64% over 7 days. Not disk (14%), not data volume (30 MB), and **not geography — region is `eu-central-2` Zurich, ideal for Romanian users**.

**Upgraded 2026-08-11 to Pro + MICRO.** Micro was a **"Free Upgrade"**: on Pro, NANO and MICRO both bill at $0.01344/h, so the resize was +$0.00 — on NANO you paid the same for shared CPU and half the memory. Monthly stays ~$25 (compute $9.68 is covered by the Pro credit).

Also discovered on the way, and the stronger reason to upgrade: **the project had NO BACKUPS at all** on Free, with months of real client data. Pro gives daily backups, 7-day retention.

Result: idle latency down 20–30%; suite went from 2 failures every run to **1 failure in 3 runs**, and the failing run took 2.0 min vs 52 s for the green ones — same "slow ⇒ fails" signature, so some contention remains. Worth re-running a few times across a day to characterise.

## The latency flake — recognise it, don't debug it as logic
Roughly **once per full run** (3 of 5 runs on 2026-08-15), whichever spec happens to be running
when the database stalls fails — a different one each time. Signature: a 15–30 s wait expiring,
and the page showing the app's own **"Conexiune lentă. Datele tale sunt în siguranță"** screen,
i.e. a read past `DEFAULT_REQUEST_TIMEOUT_MS` (15 s in `src/lib/api/timeout.ts`). Every one of
them passes standalone. Timings in a bad run give it away: a spec that normally takes 6 s takes
23–32 s and still passes.

Each superset spec adds workout starts, so a fuller suite makes the window likelier. The cause
is DB response time, not the code under test.

## Still open
- App shows raw Postgres error text to clients — should be a friendly message.
- The `user` → `user?.id` fix was applied ONLY to use-active-session; the other 8 hooks the review listed still re-run on every auth refresh.
- Trainer specs: **create-plan is now covered** (2026-08-15). Still missing: assign to client,
  add client via link, and the create-WORKOUT `test.fixme` — though the picker selectors it was
  blocked on are now proven in `plan-wizard.spec.ts`, so it is cheap to write.
- Signup untestable until it's known whether email confirmation is on. Owner also reports the role selector sometimes appearing when it shouldn't — `auth-form/index.tsx:92-95` documents a matching race.
- Chromium only, though the app is a PWA used mostly in mobile Safari.

## CI — written, cannot run (TODO)
`.github/workflows/quality.yml` (`1b7473b`) runs lint + type-check + unit tests + build on PRs and pushes to main. **It has never executed.** Every run shows "Startup failure": GitHub refuses to allocate a runner because *"Your account's billing is currently locked"*.

The billing state is contradictory and looks like a stuck flag: the account is on **GitHub Free**, all usage is **$0**, both subscriptions are Free — and `Settings → Billing → Payment information` says both *"Invalid payment method - authorization hold failed"* AND *"You have not added a payment method"*. So the flag is for a card that is not attached.

**Re-checked 2026-09-23 — unchanged after six weeks, 90 runs, all "Startup failure"**,
including the push that shipped `/deschide`.

🔴 **CAUSE FOUND 2026-09-23, and it is the bank, not GitHub.** The owner retried that
day and got an SMS from ING: *"cardul 1796 a fost blocat in urma unor tranzactii
suspecte, efectuate la GITHUB\* CARD VERIFY"*. ING's fraud filter refuses the
authorization hold and blocks the whole card with it. So the six-week-old "contact
your bank" message was literal advice, not boilerplate. Fix: unblock at ING (on the
number printed on the card, never the one in the SMS), ask them to allow
international online payments, retry. A Revolut card usually passes where a Romanian
personal card does not.

⚠️ **A WRONG GUESS OF MINE, recorded so nobody repeats it:** I saw the account's
billing address is the old *Str. Gheorghe Șincai, Alba Iulia* — not the PFA's
Bucharest address from the legal pages — and concluded an address/card mismatch was
declining the hold, advising "fix the address first". **The owner corrected me: it is
his PERSONAL card, used everywhere, so that address already matches what the bank
holds. Do not change it.** The address on those fields serves the bank check, not the
PFA identity. The PFA name belongs in "Additional information", and is moot while the
account is Free and nothing is invoiced.

**Owner's plan (2026-08-13): retry tomorrow; if still blocked, attach a card** — the first successful authorization clears the flag, and with $0 owed on Free nothing gets charged. Alternative is GitHub Support (slower). The workflow needs no changes; once Actions runs, open either failed run → **Re-run jobs**, then add repo secrets `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` (build-time — `supabase.ts` throws at module scope without them; neither is sensitive, both already ship in the client bundle).

**Correction worth remembering:** the billing lock blocks ONLY Actions. A `git push` failed once with "Permission denied (publickey)" and I attributed it to the same flag — wrong. Read worked, write didn't, and it looked causal, but the owner changed nothing and a plain retry succeeded. Treat a one-off push failure as transient; retry before diagnosing.

## Meanwhile: local pre-push gate (`7dc60df`)
`.githooks/pre-push` runs `npm run verify` (lint + type-check + 106 unit tests) in ~9s and blocks the push on failure. Wired via `core.hooksPath`, so hooks are versioned rather than hidden in `.git/`; install once per clone with `npm run hooks:install`. `next build` is excluded on purpose — Vercel builds every push and refuses to deploy a broken one.

This is genuinely sufficient here, not a compromise: the owner is the only one who pushes, and his brother's work reaches main through the same machine (Vercel Free means the deploy must come from the owner's account). Escape hatch: `--no-verify` or `SKIP_VERIFY=1`.
