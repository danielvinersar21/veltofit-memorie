---
name: backlog-middleware-proxy-deprecation
description: "Next deprecated the `middleware` file convention in favour of `proxy` — and Velto's entire subdomain routing runs through it. Surfaced by a build warning 2026-08-29, deferred."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T06:56:04.810Z
---

Every `npm run build` prints:

> ⚠ The "middleware" file convention is deprecated. Please use "proxy" instead.

**Why this one is not just noise:** the middleware file is what routes
`veltofit.app` → `(marketing)/`, `app.veltofit.app` → `(app)/` and
`dashboard.veltofit.app` → `(dashboard)/`. If it stops being honoured on a Next
upgrade, all three domains serve the wrong route group at once — the whole
product, not one page. That makes it a thing to schedule deliberately rather
than discover halfway through an upgrade.

Still only a **warning** today: the convention works, nothing is broken, there is
no deadline beyond "the major that removes it".

**When it is done:** its own branch, and verify by actually loading all three
subdomains — this is exactly the kind of change a green build does not prove.
Related trap already learned: local origins usually have live sessions, so a
signed-out check needs an incognito window (see [[landing-light-redesign]]).

Deferred by the owner 2026-08-29; it surfaced during the private-notes migration
and did not belong in that deploy. Also from that same build: a repeated
`baseline-browser-mapping` "data is over two months old" notice — cosmetic, fixed
by bumping that dev dependency.
