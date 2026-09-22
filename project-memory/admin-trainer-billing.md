---
name: admin-trainer-billing
description: "Admin trainer page shows package/status/trial/renewal/payments — SQL `admin_trainer_billing.sql` APPLIED + VERIFIED in prod 2026-09-22 (verify 23 OK; row 8 was a script bug, fixed). LIVE: pushed 2026-09-22 (e897e47..ef6e1c0)."
metadata:
  type: project
---

The billing half of the owner's admin request (backlog "Zona de admin" (a)), built
2026-09-21/22 right after [[trainer-signup-source]].

**SQL APPLIED in prod 2026-09-22** by the owner, guided: pre-flight 12/12 OK (11
trainers, 6 exempt, 1 active sub, 1 canceled, 1 payment; 5 exempt rows still carry
the expired legacy trial date 2026-09-20), whole file "Success", verify script
23 OK + 3 INFO + row 8 FAIL — the FAIL was the SCRIPT comparing `format('%s', bool)`
(t/f) to 'true'/'false'; the data matched exactly. Script fixed the same day
(`livemode::text`, row numbers rounded). The owner then eyeballed the card locally:
"pare totul în regulă".

Two NEW read-only RPCs (`admin_trainer_billing`, `admin_trainer_payments`), gated by
`is_platform_admin()`. `admin_trainer_detail` was deliberately NOT re-created again —
so there is no re-run-order trap between the two files.

Stripe live/test mode is not stored per subscription; inferred from the latest
payment, else the card shows both links. Durable fix (webhook-written `livemode`
column) not built.

**Code PUSHED 2026-09-22** as `e897e47` (feature) `e6200bc` (privacy §2/§4/§8)
`ef6e1c0` (backlog), together with the owner's social commit `ab4d35c` (he said
"urcă tot"). Pre-push green, 1594 tests.
