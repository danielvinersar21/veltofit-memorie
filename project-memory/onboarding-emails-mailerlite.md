---
name: onboarding-emails-mailerlite
description: "Owner decided 2026-09-22: friendly onboarding emails for new trainers (welcome, week-1 check-in…) live in MailerLite automations; the app only adds new trainers to a group with name, email, package. In progress."
metadata:
  type: project
---

Owner's words: emails „prietenoase ca trainerul să se simtă bine, inclus, ca într-o comunitate" —
not newsletters. Decided 2026-09-22 (AskUserQuestion):
- **Where:** MailerLite automations (already linked to the domain; owner edits emails visually).
- **Sender:** „Daniel de la Veltofit", replies go to the owner.
- **Data to MailerLite:** name, email, package only (no city, provenance, client counts).
- **Who:** only trainers who become trainers from now on; not exempt friends, not demo; the three
  organic ones get a personal email from the owner instead.
- Copy: first person only about building, never about coaching ([[owner-never-a-personal-trainer]]);
  content-reviewer before activation. Privacy policy must add MailerLite (it wasn't listed; the old
  `/api/subscribe` MailerLite route was dead and a spam vector — removal proposed).

Also 2026-09-22: the owner's test account's DB subscription + 99 RON payment are Stripe TEST-mode
(written by local testing into the prod DB) — nothing real to invoice; the live one is a trial he
cancels in the Stripe Dashboard, not via the app. See [[billing-payments-ledger]].

## Built 2026-09-22 (uncommitted at the time of writing)
`src/lib/api/mailerlite.ts` + `src/features/onboarding/onboarding.server.ts`, hooked into the
trainer-created route and the webhook's after(). Dead `/api/subscribe` deleted. API behaviour
checked against developers.mailerlite.com the same day (unsubscribed can't be reactivated via API
without `resubscribe:true`, never sent; PUT with `groups` removes unlisted groups — fields-only).
Privacy policy names MailerLite (§6, §7, §8, §9) + §1 / Terms §13 operator lists.

**Before the owner switches it on (env vars MAILERLITE_API_KEY + MAILERLITE_TRAINERS_GROUP_ID,
Production only, AFTER the privacy page is live):** in MailerLite check (1) open/click tracking —
if on, turn it off or add two sentences to §7/§8 (content-reviewer gave the wording); (2) every
automation email has the unsubscribe link; (3) emails are welcome + app tips, not upsell —
through content-reviewer before activation; (4) double opt-in for API subscribers OFF; (5) old
waitlist subscribers from `/api/subscribe` in the account — decide (backlog); (6) MailerLite DPA
in force for the PFA account, hosting region, non-EEA sub-processors. Create the group, the text
field key `pachet`, automation trigger "joins group" without re-entry.
Runbook: to keep someone out, UNSUBSCRIBE them in MailerLite — never delete the subscriber.
