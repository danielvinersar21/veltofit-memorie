---
name: owner-alerts
description: "Email alerts to the owner (new trainer, payments, cancel, failed) — SQL `owner_alert_log.sql` APPLIED + VERIFIED in prod 2026-09-22 (18 OK + 1 INFO). Env vars set in Vercel by the owner. LIVE: pushed 2026-09-22 (4fb55e4..65f1433)."
metadata:
  type: project
---

Built 2026-09-22. Recipient `contact@veltofit.app` (owner's choice; Gmail filter on
sender `alerte@veltofit.app`), sender default `Veltofit <alerte@veltofit.app>` — no
mailbox needed for the sender. Owner created a Resend key ("Sending access",
veltofit.app) and set `RESEND_API_KEY` + `OWNER_ALERT_EMAIL` in Vercel, Production
only, 2026-09-22.

**Owner decision: alerts carry NO name, email, city or provenance** — only event,
package, amounts, dates, Stripe status/document ids, links to admin. Chosen because
rich alerts would make the mailbox a personal-data store (policy in 5 places, Google
Workspace as processor, inbox purges on every deletion/objection). The privacy page
deliberately says "fără nume sau adrese de email", NOT "fără date personale" (ids and
amounts are pseudonymous) — don't "improve" it.

SQL applied: pre-flight 5/5, file Success, verify 18 OK + 1 INFO after fixing a
script bug (format('%s', bool) prints t/f — SAME trap as admin_trainer_billing_verify
the day before; use ::text).

Not covered on purpose: a trainer role set directly in the DB (no app path) sends no
alert; no trial-ending / seat-wall alerts (no cron).

**Code PUSHED 2026-09-22** as `4fb55e4` (feature + webhook) `407c68e` (privacy) `65f1433`
(backlog); pre-push green. First
real-world test planned right after deploy: a fresh trainer signup on prod → expect
„Antrenor nou pe Veltofit" at contact@ (DMARC IS set since 7 Sep, p=none; a Gmail filter on the sender also marks "never spam"), then delete it.
