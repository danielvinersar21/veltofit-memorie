---
name: launch-readiness-plan
description: "The full path from today to public signups, written 2026-08-29 at docs/plans/launch-readiness-v1.md. Verdict: promote yes, open the tap no — and why the trial starts a 3-week clock."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-30T08:23:09.939Z
---

Owner asked on 2026-08-29 whether he can start promoting and let people begin
trials. Answer given: **promote and onboard manually, do NOT open public
self-serve signup yet.** Full plan lives in the repo at
`docs/plans/launch-readiness-v1.md` — read that, not this; this note only records
the framing so it is not re-derived.

## The insight that drives the whole plan

The trial is 30 days, no card. So Stripe is not needed on day 1 — **it is needed
on day 31.** If promotion works and 20 people sign up in week one, they all reach
day 31 in the same week. The moment the first stranger clicks "Începe gratuit", a
three-week clock starts in which Stripe, an email path AND a cron must all exist.

So the recommendation is not "wait" — it is **control the flow rate**: promote,
talk to trainers, add 2-3 per week manually and watch what they do.

## The three facts that surprised, all verified in code 2026-08-29

1. **Nobody has ever signed up as a new trainer.** All 5 accounts are
   `billing_exempt`, so `provision_trainer_subscription` has never run in
   production — and it is fail-soft (`EXCEPTION WHEN OTHERS` → `RAISE WARNING`),
   so if it breaks the trainer silently gets **2 seats** instead of 30 days
   unlimited. First contact with the product would be a wall. **Testing this with
   one fresh account is step A and costs 30 minutes with no code.**
2. **The app cannot send email at all** — grepped: no Resend/nodemailer/sendgrid
   at app level, only Supabase auth mail. And no cron (`src/app/api/` has two
   routes, no `vercel.json`). That single gap blocks trial-expiry warnings, invite
   resend (which today only re-arms the DB row and mails nobody), and the email
   variant of the consent flow.
3. **Stripe is not integrated** — the only occurrences of the word in `src/` are
   landing copy and billing copy strings.

## ✅ Applied in prod 2026-08-30 — the database owns ONE seat number

`billing_seat_defaults_free_only.sql`. `default_seat_limit_for_tier` used to
encode the dead pricing model (Plus 20 / Max 50 + metered blocks); it now returns
**only** Free = 2 and **raises 22023 for every other tier**, with a message
naming where the real number comes from (the purchased Stripe Price → written
into `trainer_subscriptions.seat_limit`).

**Why raise instead of returning the current numbers:** a seat count is a pricing
decision, not a database fact — Plus moved 20→15 precisely to fix the price
curve, so any number stored there goes stale the same way. And for Max there is
no correct single answer: six rungs (30/50/75/100/150/200), so whoever bought 200
would be walled at the entry step. Same instinct as `toClientState`'s required
argument: better a loud error than a silently wrong default.

Zero rows written, no live behaviour changed — all four callers pass the literal
`'free'`. Verified live afterwards: `'free'` → 2, both catalog comments rewritten,
`'plus'` raises.

## Also decided the same day

**Both notes fields hidden from the UI** (client health notes AND the trainer's
private note), to shrink what has to be explained to a lawyer — health notes are
GDPR special-category data. **UI only: columns, RPCs, API fields and existing
data all stay.** Shipped `7ec2831`. **To restore: flip `SHOW_CLIENT_NOTES` to
`true` in `src/config/feature-flags.ts` — that is the entire revert**; every
render site keeps its markup behind that guard.

Three things left open by it, all for the owner:
- A **third** free-text field about a person exists and was left alone:
  `client_measurements.notes` ("Notițe (opțional)"). Same flag would cover it.
- `privacy/page.tsx:212` says the client "can read them in your profile but
  cannot edit them directly" — **wrong before today** (they could edit since the
  consent feature) and now wrong the other way (they can no longer read them).
  Needs the lawyer, not a quiet edit.
- The client's only two routes to see and delete health notes written about them
  were the consent panel and their edit field. Both are hidden and the data is
  still stored, so the only erasure route left is the email address in the
  privacy policy. That is the part of the GDPR story that actually changed.

Related: [[backlog-case-a-consent-flow]] (the next code task),
[[pricing-monetization-plan]] (read before creating Stripe products — the
per-client cost must never rise as capacity grows),
[[launch-gtm-decision]], [[backlog-launch-blockers]].
