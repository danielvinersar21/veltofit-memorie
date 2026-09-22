---
name: rls-write-side-holes
description: "Two preexisting FOR ALL policies missing WITH CHECK — trainer_clients and client_plan_progress. Personally confirmed in schema 2026-08-20. Root cause worth remembering: Postgres reuses USING as WITH CHECK."
metadata:
  node_type: memory
  type: project
---

Found during the billing Phase 1 audit ([[billing-phase1-schema]]) and **verified
by reading the schema directly** — not taken on an agent's word.

**The root cause, which will recur:** when a policy has `USING` but no
`WITH CHECK`, Postgres reuses the `USING` expression as the `WITH CHECK` for
INSERT/UPDATE. If that expression only constrains the "owner" column, the OTHER
foreign key is completely unconstrained on write. Every `FOR ALL` policy in this
schema is worth re-reading with that in mind.

**🔴 `trainer_clients` — `velto_schema_refactored.sql:566`**
```sql
CREATE POLICY "tc_trainer" ON trainer_clients FOR ALL USING (trainer_id = auth.uid());
```
A trainer can INSERT `(trainer_id = self, client_id = ANY uuid, status='active')`.
That row **is** the authorization surface for `measurements_trainer` (which is
`FOR ALL` — read AND write on body measurements), `profiles_trainer_sees_clients`,
`sessions_trainer`, `progress_trainer`. So: silently re-attach a former client and
read their health data. Limited by `uq_trainer_clients_one_active_trainer` (victim
must have no active trainer) and UUIDs not being enumerable.

**🟠 `client_plan_progress` — `velto_schema_refactored.sql:557`**
```sql
CREATE POLICY "progress_client" ON client_plan_progress FOR ALL USING (client_id = auth.uid());
```
A client can INSERT a row with ANY `plan_id`. Then `plans_client` (`:484`) grants
SELECT on that plan, and the child policies grant the whole tree. Any client can
read any plan in the DB given its UUID. Matters more now that plans are the paid
product.

Audited and clean: `plans_trainer`, `sessions_client`, `set_perf_client`,
`meal_plans_trainer`, `saved_items_own`, `invites_trainer`, `client_profiles_*`,
storage policies. `plan_weeks`/`plan_days`/`plan_day_exercises` were exposed but
fixed 2026-08-15 (`security_fix_plan_children_rls.sql`) — **re-confirm in the live
DB that that file was actually applied**, since migrations here are applied by hand.

## Fix: `docs/database/security_fix_write_side_rls.sql` — ✅ **APPLIED & VERIFIED IN PRODUCTION 2026-08-20**

Applied by hand in the Supabase SQL editor (project `wcpzdurualepyjjrracm`, branch
`main`), wrapped in `BEGIN`/`COMMIT` because the file DROPs `tc_trainer` before
creating its replacements — an un-wrapped mid-file failure would have left the
trainer app dark. Final policy set: 7 policies, **none `ALL`, none `INSERT`**, both
freeze triggers present.

**All 8 attack tests blocked, all 6 legitimate tests pass.** Verified against the
live DB, not the repo file. Two traps hit while testing, both worth remembering:
- The Supabase SQL editor does **not** show `RAISE NOTICE` output. Tests written
  around notices read as "Success" whatever happens. Rewrite them so the ERROR is
  the signal, and so `UPDATE`/`DELETE` report through `WITH ... RETURNING`.
- The first run of tests (7)/(8) was **vacuous** — the chosen client had zero
  `client_plan_progress` rows, so the UPDATE/DELETE matched nothing and "passed"
  for no reason. Always confirm the fixture has data before trusting a green.
  Same for test (11): the first trainer's clients had no measurements/progress at
  all, so it proved nothing until re-run against a trainer with 22 clients / 97
  measurements / 25 progress rows — where every count matched ground truth exactly.

Strongest result: `assign_plan_to_client()` took the `ON CONFLICT DO UPDATE` path
(twice) without the freeze trigger raising, and `accept_invite()` linked a client
`active` on two consecutive calls. Those were the only genuinely novel
interactions, and the "no INSERT policy" stance depends entirely on the latter.

The first draft was **rejected in review, and the reason is the lesson**: both of
its new INSERT policies anchored on columns the attacker writes, through the very
same bug class being fixed — `progress_client_insert` trusted
`profiles.assigned_plan_id` (self-writable via `profiles_own`), and
`tc_trainer_insert` trusted `invites.accepted_by` (`invites_trainer` constrains
only `trainer_id`, and `createInvite()` inserts straight from the browser).

**The accepted fix is subtraction, not a longer predicate:** both INSERT policies
were deleted outright. Neither table has ANY authenticated INSERT path in the app —
everything is `service_role` or `SECURITY DEFINER` — so a permissive policy written
for a hypothetical future flow was pure security debt. When that flow is built it
gets a SECURITY DEFINER RPC, like `assign_plan_to_client` already does.

Also in the final version: `progress_client_delete` removed (a client could erase
completed/abandoned history the trainer reads); `tc_trainer_update` constrained to
`status IN ('pending','inactive')` so only `accept_invite()` can set `'active'`;
`BEFORE UPDATE` triggers freezing the FK columns (a `WITH CHECK` alone does NOT
stop `UPDATE ... SET client_id = '<victim>'` — the owner column is unchanged);
trigger guard is `current_user IN ('authenticated','anon')`, fail-closed, not
`auth.uid() IS NOT NULL`, which fails open.

**Still open, same bug class, now inert:** `profiles_own` and `invites_trainer` are
both still `WITH CHECK`-deficient. Not exploitable once no policy reads their
columns, but they should be fixed.

**Keep this detector — it would have caught all of them:**
```sql
SELECT tablename, policyname, cmd FROM pg_policies
 WHERE schemaname = 'public' AND cmd = 'ALL' AND with_check IS NULL;
```
`measurements_trainer` is a false positive there (its `USING` reuse is correct).
