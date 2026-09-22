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
