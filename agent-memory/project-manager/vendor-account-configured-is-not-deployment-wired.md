---
name: vendor-account-configured-is-not-deployment-wired
description: "Third-party account configured" and "production wired to it" are two different claims — never let one status line cover both, especially for payment processors.
metadata:
  type: project
---

When a status line says a third-party service is "fully configured", split it into two
questions before reporting it as done:
1. Is the **vendor account** configured (products, prices, portal, retry rules)?
2. Is the **deployment** wired to it (live credentials in the hosting env, a webhook
   endpoint registered in live mode, a redeploy after the keys changed)?

**Why:** in this project both were recorded under one phrase — "live mode configured
completely" — and only the first was true. Production ran on test credentials with no
live webhook for three days, and the *test* webhook pointed at the production host, so
test-mode events were writing into the production database. Nothing looked broken,
because everything was consistent with itself: test keys, test webhook, screens
confirming payments that did not exist. A real customer would have "paid", been given
access, and nothing would have been collected. It was caught only because the owner
noticed a "Sandbox" badge on the real app's payment page.

**The same failure recurs when code starts handling a NEW webhook event.** A webhook
endpoint has an explicit event subscription list, chosen once and then forgotten. Code
that begins listening for one more event type is inert until someone ticks that event in
the vendor dashboard — and the symptom is an empty screen, not an error. Whenever a diff
adds an event constant, check the runbook/setup doc: if it still enumerates the old set,
the endpoint almost certainly does too, and the feature will silently record nothing.

**How to apply:** treat env-var and webhook-endpoint state as its own checklist row,
separate from the vendor dashboard row, and never mark it from a memory note or a plan
doc — those describe the account. Also remember that env changes need a **redeploy** to
take effect, and that after any mode mix-up you must check for rows the wrong mode
already wrote (a stale customer id will be re-used and silently point at nothing).

Related: [[first-charge-settings-are-irreversible]] (same domain, other half: the
irreversible settings live in the dashboard), [[verify-project-status-from-git-refs]].
