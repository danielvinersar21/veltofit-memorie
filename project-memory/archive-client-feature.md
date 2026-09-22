---
name: archive-client-feature
description: "Archive/reactivate client — SQL APPLIED & verified in prod 2026-08-22, UI built. Closed a pending→inactive→active laundering bypass; trainer_clients now has NO authenticated UPDATE at all."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T11:55:20.078Z
---

Built 2026-08-22 because the seat model needs a way to free a seat that is not
"delete the person". `docs/database/reactivate_client_rpc.sql` (1109 lines) is
**APPLIED in production and verified live**; the UI is on branch
`feat/pricing-and-billing`, uncommitted.

Verified end-to-end against the live DB, inside a rolled-back transaction:
archive → status `inactive`, **name still visible**, measurements hidden, seat
22→21; reactivate → `active`, measurements back, seat 21→22; second reactivate
call is a silent no-op. Read counts for the 22-client trainer unchanged
(22/23/97/25) — no regression.

## The bypass that shaped the design

The first draft only added `reactivate_client`. Review found a deterministic
2-request laundering: `tc_trainer_update`'s `WITH CHECK (status IN
('pending','inactive'))` is a **set of destination states, not a transition**, so
a trainer could PATCH a `pending` row to `inactive` and then "reactivate" it —
gaining access to the health data of someone who never accepted anything.
`pending` rows for real people exist: `/api/clients/invite` CASE B creates the
`auth.users` account **and** a pending link.

**Fix by subtraction, again:** archiving also became an RPC, `tc_trainer_update`
was dropped, and `REVOKE UPDATE ON trainer_clients FROM authenticated, anon` was
added. `trainer_clients` now has **no authenticated UPDATE path at all**. Both
RPCs refuse `pending` explicitly.

**The general lesson, third time it has appeared:** a guard that checks *current
state* is worthless when the attacker controls current state. Anchor on something
they cannot write — or remove the write.

## Two defects the UI work exposed in the DB design

Both were invisible until someone tried to use the feature:

1. **Archiving hid the client from their own trainer.**
   `profiles_trainer_sees_clients` required `status = 'active'`, so archiving
   revoked the trainer's read on that profile. Clients who joined via the
   reusable link leave no `invites` row, so they rendered as "Client arhivat" —
   no name, no avatar. "Reversible" is meaningless if you cannot tell who is who.
   Widened to `IN ('active','inactive')`; `pending` stays excluded. Safe because
   the only path to `inactive` is `archive_client()`, which starts from `active`.
   **Only identity widened** — `measurements_trainer`, `client_profiles_trainer`,
   `sessions_trainer`, `progress_trainer` all stay `active`-only. Archiving ends
   access to health and training data; it should not end the ability to remember
   a name.
2. **An archived client could not be deleted, and at the cap that was a dead
   end.** `removeClientFromTrainer` calls `unassign_plan_from_client`, which
   authorized on an active link. Archive → want to delete → must reactivate →
   reactivation needs a free seat → at the cap, neither works. Widened that RPC
   to `active` or `inactive`. `assign_plan_to_client` deliberately stays strict.
   ⚠️ It is defined in BOTH `trainer_plan_rpcs.sql` and this file — re-running the
   older file silently reverts the fix.

## Product decisions locked

- **Archiving does NOT stop the client's plan** (`assigned_plan_id` untouched).
  The client keeps training; only the trainer's roster changes. The confirm copy
  says so explicitly rather than implying "archive = stop everything".
- The seat-limit copy now says "arhivează", not "elimină" — the earlier wording
  told a trainer on a phone to destroy a relationship using a desktop-only button.
- Archive is never offered for `state === 'invited'`; there the right actions are
  resend/revoke, and `archive_client` refuses anyway.

## Still open, preexisting, worth a ticket

`/api/clients/invite` CASE A creates an **active** link via service_role for any
existing account, from an email the trainer types — no consent, no cap check. The
consent invariant these RPCs defend is already open on that door.

🟠 **An archived client can un-archive THEMSELVES** — found 2026-08-29 during the
seat-cap audit, pre-existing and still unfixed. `accept_invite()` branches
`IF active → active / ELSIF trainer_can_add_client → active / ELSIF inactive →
inactive`, so an `inactive` person who re-opens the trainer's reusable link with
a seat free is silently flipped back to `active`, bypassing `reactivate_client()`
and every guard above. And because all 5 trainers are `billing_exempt`,
`can_add` is ALWAYS true today — so this is the only branch that ever runs, and
the seat-cap branch added on 2026-08-29 never executes in production yet.
`activate_waiting_client()` explicitly refuses `inactive` and points the trainer
at reactivate; `accept_invite()` does it silently. Closing it is a product
decision (should re-opening the link re-establish a relationship the trainer
ended?) and belongs in its own file. Documented in the state diagram of
`accept_invite_seat_cap.sql`; see [[backlog-two-migrations-to-apply]].
