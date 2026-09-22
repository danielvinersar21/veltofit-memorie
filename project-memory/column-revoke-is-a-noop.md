---
name: column-revoke-is-a-noop
description: "Column-level REVOKE does NOT work in Postgres when a table-level grant exists. The trainer_notes hole it left is CLOSED in prod 2026-08-29; the audit of the other REVOKE sites is still open."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T07:12:08.407Z
---

**The technique this repo uses to hide a column does not work.** Verified against
the live database, not reasoned about: as an `authenticated` client,
`SELECT trainer_notes FROM trainer_clients WHERE client_id = auth.uid()`
**succeeds**. `tc_client` grants SELECT on the row; the column REVOKE does not
subtract from it.

Why: in PostgreSQL a table-level privilege implicitly covers every column, and a
column-level `REVOKE` **cannot subtract from it** — `relacl` and `attacl` are
additive and there is no deny. Postgres answers with a WARNING
("no privileges could be revoked for column"), not an error, so it looks applied.
Supabase grants ALL on `public` tables by default, so the table-level grant is
always there.

## ✅ CLOSED IN PRODUCTION 2026-08-29

`trainer_private_notes.sql` **applied and verified live**. Four checks passed:
`trainer_clients.trainer_notes` no longer exists, the new table has RLS on, it
carries exactly four policies (`tpn_trainer_select/insert/update/delete`) and
**none for the client**, and 0 notes needed migrating — so the fix landed before
anyone wrote a note, exactly as intended. Deploy order was code-first (see
[[backlog-two-migrations-to-apply]]); the route change shipped in `ee61fb2`.

The lesson below stands and is the reason the fix took the shape it did — it is
now also written into `getTrainerNote`'s docblock in `clients-api.ts`, which is
the only place in `src/` where it can be read by whoever next reaches for a
column REVOKE.

**Historical, for context.** `add_client_flow.sql:166` did
`REVOKE SELECT (trainer_notes)` while the UI promised *"Doar tu vezi aceste note.
Clientul nu le poate citi."* That promise was false. Measured 2026-08-22: **0 of 25
links had a non-empty `trainer_notes`**, so nothing ever leaked.

**What actually works instead**, in rough order of preference:
- Move the column to its own table with its own RLS (trainer-only policy).
- A `BEFORE` trigger for write protection (this is what `consented_at` uses in
  `accept_invite_seat_cap.sql`, deliberately, instead of trusting a column REVOKE).
- A view that omits the column, with the base table unreadable.

**Audit result — all `REVOKE .* (` sites checked 2026-08-22:**

| Site | Verdict |
|---|---|
| `add_client_flow.sql:166` `trainer_notes` | ✅ **CLOSED in prod 2026-08-29** by `trainer_private_notes.sql` (own table, real RLS) |
| `accept_invite_seat_cap.sql:204` `consented_at` | ⚪ same broken technique, but harmless — the real guard there is a trigger, and the file says so |
| `measurements_full_set.sql:234` `client_measurements` UPDATE | 🟠 **table-level, so it DOES apply — but to a VIEW.** See below |
| `billing_trainer_subscriptions.sql:218,246,283` | ✅ table-level, correct technique |

**The measurements one is a different bug worth knowing.** The revoke lands on
the `client_measurements` view, while the base table `public.measurements` stays
exposed through PostgREST with `authenticated` holding UPDATE and
`measurements_own` being `FOR ALL`. **Verified live 2026-08-22: a client can
UPDATE 9 of their own measurement rows via `/rest/v1/measurements`.** So the
history is not append-only, and a trainer's progress report can change under them
silently. Integrity problem, NOT a confidentiality one — nothing leaks. The file's
own verification tested the view, not the base table, which is why it looked
checked. Not fixed; no ticket raised yet.

Also noted in the same audit, unfixed: `authenticated` holds TRUNCATE on `public`
tables from Supabase's default privileges, and TRUNCATE bypasses RLS. Not
reachable through PostgREST (it exposes no TRUNCATE), so it needs a direct DB
connection — low urgency, but new tables should revoke it as a matter of form.

Related: [[rls-write-side-holes]], [[archive-client-feature]] — same family of
"the guard looked applied but wasn't".
