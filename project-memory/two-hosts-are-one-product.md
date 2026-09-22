---
name: two-hosts-are-one-product
description: Owner's mental model 2026-09-03 — dashboard and app are ONE product, desktop and mobile of the same thing, and should share everything. The per-origin session is the single place reality disagrees.
metadata:
  type: project
---

Owner, 2026-09-03, after being walked through why paying from a phone landed on
a login screen: *„inteleg ce zici ca e dashboard si app si doua chestii diferite,
dar eu le vad asa: dashboard e pe desktop, app e pe mobile (cum ar veni
responsive). deci ele ar trebui sa imparta exact aceleasi lucruri, mai mult sau
mai putin."*

**Why:** this is the product intent, and it is not what the code assumes. The
two hosts are two ORIGINS to a browser, so the Supabase session — kept in
localStorage — does not cross between them. Every seam that has cost time
(billing missing on the phone, Stripe returning to the wrong host, notices that
send a phone user cross-host, `?plan=` dropped on a login bounce) is the same
seam: the product is one thing, the session is two.

Do NOT treat a feature existing on only one host as a decision. Under this
model it is a gap, and the burden is on the reason it is missing — the reverse
of how V8 was first written up, where a missing page hid behind a decision
presented as a choice.

**How to apply:** when a feature lands on one host, ask what the other one does
with it before calling it done. When something must address the other host, the
cost is one extra login, and it is the model's only real exception — say so out
loud rather than routing around it silently.

The two ways to close it properly, neither built and neither cheap: a session
cookie shared across `*.veltofit.app`, or collapsing to one host. See
[[stripe-integration-state]] for what depends on this today; the mechanics are
written out in `src/lib/app-hosts.ts`.
