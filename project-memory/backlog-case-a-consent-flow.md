---
name: backlog-case-a-consent-flow
description: "CLOSED 2026-08-30 (7b092d8): manual add now REFUSES an existing account and points to the reusable link. The in-app consent prompt planned here was dropped — do not build it."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-30T12:42:14.553Z
---

## ✅ CLOSED 2026-08-30 — but NOT the way this note planned

Shipped in `7b092d8`. **The in-app consent prompt below was dropped.** Manual add
now simply **refuses** an address that already has a Velto account and points the
trainer at their reusable link — which already worked for a logged-in existing
account and already records consent, checks the cap and activates on the normal
path. CASE A creates no `trainer_clients` row in any state; the `link` verdict was
deleted from the TYPE, so restoring it fails to compile.

**Why the plan below died:** it needed an in-app notification, and the owner
deleted the notification system the same day (see [[notifications-never-worked]]).
Rebuilding it for this would have contradicted that decision. And the deeper
reason it was never good: the app cannot reach someone who is not looking at it —
no email path for existing accounts, no push. A card is seen only by someone who
opens Velto having no trainer, no plan and no reason to. A WhatsApp message from
their own trainer arrives.

**What the trainer lost, deliberately:** manual add now serves exactly one
purpose — inviting someone with no account yet. Written at the route header, on
the refusal constant and on `decideCaseA` so nobody "restores" it.

The delete-then-re-add bypass below is closed with it: there is no longer any
path in that route that creates a link, so deleting one and retyping the address
lands on the same refusal.

---

**Historical — the plan that was NOT built.** Owner's decision 2026-08-29: when a
trainer types the email of someone who already has a Velto account, that person
must be **asked**, not silently linked.
Today `/api/clients/invite` CASE A creates an `active` link on a typed email
alone — no consent, nothing sent to the person — and from that moment the trainer
reads their measurements, health notes, sessions and progress.

## ⚠️ The trap that makes this bigger than it sounds

The obvious implementation — "send them an invite email like CASE B does" —
**cannot be built with what exists.** CASE B uses Supabase's
`inviteUserByEmail`, which creates the account AND sends the mail as one
operation; it does not apply to an account that already exists. And the app has
**no other email path at all** — the only mail Velto sends today is Supabase's
own auth mail. So that route needs a whole email integration first.

**Do it in-app instead.** The person has an account, so the honest channel is a
notification: create the link as `pending` with `consented_at` NULL, notify them
("X vrea să te adauge ca și client") with Accept / Refuse, and on accept run the
same path the reusable link already uses — consent is stamped, the seat cap is
checked, the link goes active. No new infrastructure, and better than an email
they would miss.

Scope: notification with actions, an accept/refuse endpoint, the client-side
screens, and a decision about what refusing does (delete the link? mark it, so
the trainer cannot re-ask forever?).

## Why it was NOT done on 2026-08-29

Four hours of verified work was sitting uncommitted in a working tree shared with
another Claude session. Closing the day beat starting a feature. The residual
hole is documented in a comment at the CASE A write, and
`accept_invite_seat_cap.sql:849-856` already prescribes the eventual fix (write a
non-active link so `activate_waiting_client()` refuses it).

## The bypass that must be closed with it

`tc_trainer_delete` (`security_fix_write_side_rls.sql:273-275`) is
`FOR DELETE USING (trainer_id = auth.uid())` with **no status predicate**, so a
trainer can DELETE their own `pending` link over the REST API and then re-add the
same email — landing on CASE A with no existing row to refuse against. Any fix
that only guards "a link already exists" is defeated by this in one extra
request. See [[backlog-two-migrations-to-apply]] for the consent model this
protects, and [[archive-client-feature]] for the same class of bug.

Related, found the same day and still open: `.ilike('email', …)` treats `%` and
`_` as wildcards (they pass `EMAIL_RE`), so a typed pattern can match an address
the trainer never knew — being fixed in the 2026-08-29 round, verify it landed.
