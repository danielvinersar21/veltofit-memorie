---
name: backlog-launch-blockers
description: "Open items after the 2026-08-21/22 pricing+billing push. Branch feat/pricing-and-billing, 6 commits, nothing pushed. What's applied to prod vs not."
metadata:
  node_type: memory
  type: project
---

⚠️ **SUPERSEDED IN PART ON 2026-08-28: this branch IS deployed now.** The landing
rebuild was cut from it, so pushing `main` shipped the billing UI, health-notes
consent, archive-client UI and the legal rewrite along with it. The two migrations
below are still NOT applied, and still have no code referencing them. Everything
under "Written, reviewed, NOT applied" and "Open, not started" stands, except the
sign-up CTA item, which is done.

State at end of 2026-08-22. Branch **`feat/pricing-and-billing`**, 6 commits,
**nothing pushed, nothing deployed** (as of that date).

## Applied to production and verified live
Security write-side RLS fix · billing schema + trial ledger + grandfathering of
the 5 existing trainers · archive/reactivate. See [[rls-write-side-holes]],
[[billing-phase1-schema]], [[archive-client-feature]].

## Written, reviewed, NOT applied — in dependency order

1. **`trainer_private_notes.sql`** — moves the note to its own table.
   ⚠️ Ordering is load-bearing: apply the SQL, then deploy immediately. The
   invite route writes the note into the old column in two places and those
   writes fail after `DROP COLUMN`. Affects one flow (add client WITH a private
   note). See [[column-revoke-is-a-noop]].
2. **`accept_invite_seat_cap.sql`** — 🔴 **do not apply alone.** SQL is correct;
   the state it creates (`pending` + consent) has NO usable action in the UI:
   `activate_waiting_client()` has no caller, a reusable-link claimant leaves no
   `invites` row so `showActions` is false on mobile, and both dashboard buttons
   are broken for that row. Worse, it looks fine on application — all 5 trainers
   are `billing_exempt`, so the branch is unreachable until a new trainer's trial
   expires weeks later. Needs the service function + activate button first.

## Open, not started

- **Trial reminder emails** (day 23/28/30). No email path exists in the app at
  all (only Supabase auth mail) and no cron. Recommended deferring: the first
  trainer reaches day 23 three weeks out, and by then you know whether people
  actually drop off. This is the only item with a natural deadline.
- **Stripe.** Owner already has a Stripe account on the PFA — no verification
  lead time, which removes the reason it was sequenced early. Products/prices,
  Checkout, webhook, Customer Portal. Runbook: `docs/plans/billing-operations-v1.md`.
- ~~"Începe gratuit" lands on the SIGN-IN form~~ **DONE 2026-08-28** and live:
  `/login?mode=signup` → `initialMode` prop on `AuthForm`. Never seen
  signed-out — confirm in an incognito window.
- **Invite route CASE A** upserts an ACTIVE link via `service_role` from a typed
  email — no consent, no cap, bypasses RLS and the freeze trigger. Two steps
  (invite by email, then re-invite the same address) fabricate consent. This is
  the door that stays open regardless of the seat-cap work.
- **Client can rewrite their own measurement history** — the REVOKE lands on a
  view while the base table stays writable. Integrity, not confidentiality.
- [[backlog-health-notes-consent-proof]] · [[backlog-trainer-writes-silent-noop]]
- Six legal questions await a lawyer, listed in the header of `privacy/page.tsx`.

## Testing note that keeps biting

**The owner cannot see any billing UI in their own app** — all 5 trainer accounts
are `billing_exempt`, and exempt accounts render no banner by design. A fresh
test account still will not show the seat wall either, because a new trainer gets
30 days of effectively unlimited seats. To exercise the wall, expire that
account's trial in the DB first.
