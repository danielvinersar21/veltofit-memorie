---
name: billing-phase1-schema
description: "Billing Phase 1 ✅ APPLIED 2026-08-21 — but its signup trigger and trial model were SUPERSEDED on 2026-08-31 by billing_free_has_no_trial.sql. Read the superseded-by note before quoting anything from that file."
metadata:
  node_type: memory
  type: project
---

🔴 **SUPRASCRIS PE ALOCURI — citește asta înainte de a cita fișierul.**
`docs/database/billing_free_has_no_trial.sql` a fost **aplicat în producție
2026-08-31** și a rescris `provision_trainer_subscription()`. Ce scrie în
`billing_trainer_subscriptions.sql` despre trialul de la înregistrare **nu mai e
adevărat**:

| | Phase 1 (vechi) | ACUM, din 2026-08-31 |
|---|---|---|
| cont nou de antrenor | `free` + `trialing` + 30 zile | `free`, `subscription_status = NULL`, `trial_ends_at = NULL` |
| locuri efective | 9999 cât ține trialul | **2, din prima secundă, nimic nu expiră** |
| registrul de trial | revendicat la înregistrare | **neatins** — îl revendică webhook-ul, la abonare |
| cine dă trialul | baza de date | **Stripe**, `trial_period_days` la Checkout, doar pe Plus/Max |

**Cum s-a greșit, 2026-09-17:** am citit `billing_trainer_subscriptions.sql`, am
descris trialul de la înregistrare ca fiind în vigoare, și i-am dat owner-ului
date de expirare inventate pentru doi antrenori reali. El a corectat din
observație proprie („Stripe mi-a facturat 0 lei pe trial"). Fișierul care
suprascrie **își scrie în antet că e aplicat în producție** — era acolo.

**Regula:** înainte de a cita orice din `docs/database/`, caută dacă alt fișier
redefinește aceeași funcție —
`grep -rn "FUNCTION public.<nume>" docs/database/` — și crede antetul care spune
„APLICAT ÎN PRODUCȚIE", nu ordinea în care ai deschis fișierele. Vezi și
[[two-migrations-to-apply]] pentru tiparul de pre-flight.

---

Phase 1 of [[pricing-monetization-plan]]:
**`docs/database/billing_trainer_subscriptions.sql`** (915 lines) — ✅ **APPLIED
2026-08-21**, plus `billing_grandfather_existing_trainers.sql`. Verified live: all
5 trainers on free/cap 2/trialing to 2026-09-20, all in the ledger, all
`billing_exempt = true` with effective cap unlimited. Profile editing,
`my_billing_state()` and own-row-only visibility all confirmed working.

**Two blockers were caught in review before applying — both the same class as the
security migration's, one level down.** Yesterday a guarantee rested on a column
the attacker could WRITE; here it rested on a row the attacker could DELETE:
- `trainer_subscriptions` cascades from `profiles`, and `profiles_own` is
  `FOR ALL` — which includes DELETE. So: delete own profile → subscription
  cascades away → the app itself routes to `/role-selection` → `setProfileRole()`
  recreates the row → **brand new 30-day trial**, repeatable forever. Cost to the
  attacker was small: clients, plans, sessions, progress all hang off
  `auth.users`, not `profiles`. Fixed with `REVOKE DELETE ON public.profiles`
  (verified: zero `.delete()` on profiles in `src/`) plus `billing_trial_ledger`
  keyed on `auth.users`, which no browser client can delete.
- The provisioning trigger sits on `profiles`, so any exception in it would kill
  trainer signup — and `DROP TABLE trainer_subscriptions` does NOT drop the
  trigger, so a future re-apply could open that window silently. Now fail-soft:
  `EXCEPTION WHEN OTHERS → RAISE WARNING → RETURN NEW`.

**Correction worth keeping:** "12 FKs point at auth.users, none at profiles" is
WRONG — it came from `for_debug_db.sql`, which predates `add_client_flow.sql`.
`invites.trainer_id`, `invites.accepted_by` and `client_profiles.client_id` all
reference `profiles(id)`. Count against the live DB, never that dump.

**Table `trainer_subscriptions`**, PK = `trainer_id` → FK to `public.profiles(id)`
(NOT `auth.users`, so PostgREST can embed it). Columns: `plan_tier`, `seat_limit`,
`trial_ends_at`, `subscription_status`, `billing_interval`, `stripe_customer_id`,
`stripe_subscription_id`, `current_period_end`, `cancel_at_period_end`,
`billing_exempt`.

## The three decisions that matter

**1. Billing state must NOT live on `profiles`** — the written plan says it should,
and the plan is wrong. `velto_schema_refactored.sql:453` has
`profiles_own FOR ALL USING (id = auth.uid())`, so any user can PATCH their own
profiles row via PostgREST and grant themselves `seat_limit: 9999`. Service-layer
checks are not on that path; RLS is the only boundary. Same reason `billing_exempt`
is its own column and is NEVER derived from `profiles.is_demo` at runtime.

**2. `seat_limit` stores the PAID entitlement (2 on Free); the trial boost is
DERIVED** in `trainer_seat_limit()`. The plan said store 9999 for 30 days — but
then something must claw it back on day 30 (cron/edge function/login hook), there
is no `pg_cron` in the project, and if that thing is late or never built, every
trial account is unlimited forever and the conversion moment never arrives.
Derived means day 30 happens by clock, to the second, with nothing scheduled.

**3. No policy is not enough — REVOKE too.** An UPDATE with no matching policy
does not error in Postgres: it matches zero rows and PostgREST returns `200` with
an empty body. With `REVOKE INSERT,UPDATE,DELETE FROM authenticated` it becomes
`42501 permission denied` — a loud failure, and it survives someone later adding a
permissive policy by mistake. Same technique as `trainer_clients.trainer_notes`.

## Signup hook — there is no `handle_new_user`

The `profiles` row is NOT created by a GoTrue trigger. It is upserted **from the
browser** by `setProfileRole()` (`src/features/profiles/services/profiles-api.ts`),
or by the invite route with `service_role`. At `auth.users` INSERT the role is
still null, so a trigger there would fire blind. The correct hook is the moment
the profile *becomes* a trainer: `AFTER INSERT OR UPDATE OF role ON profiles
WHEN (NEW.role = 'trainer')`, with `ON CONFLICT DO NOTHING` so flipping role back
and forth cannot re-issue a 30-day trial.

## auth-store integration — zero extra round-trip

`auth-store.ts:111` `.from('profiles').select('*')` becomes
`.select('*, trainer_subscriptions(...)')`. Works because the FK is on
`profiles(id)` and `trainer_id` is also the PK, so PostgREST treats it one-to-one
and returns an object, not an array. **Requires `NOTIFY pgrst, 'reload schema'`**
(it is at the end of the migration) or the embed errors with "Could not find a
relationship". Keep `Profile` mirroring the profiles table 1:1 and add
`subscription` as a SIBLING in `AuthState`. After Stripe Checkout returns, Phase 3
must `apiClient.invalidate('auth:')` + `fetchUser()` or the trainer pays and sees
the old tier for 5 minutes.

## Decisions locked 2026-08-20

- Existing trainers' trial starts today; owner's own trainer account is
  `billing_exempt` permanently.
- Cancel → seats drop at `current_period_end`, not immediately. `past_due` → keep
  seats while Stripe retries (~2wk) + banner. Metered = `50 + 10 × quantity`.
- **Client claiming an invite link when the trainer is at cap → lands `pending`**,
  trainer activates after archiving someone. Needs `accept_invite()` changed +
  activation UI in Phase 2. `trainer_can_add_client()` must be wired into BOTH
  chokepoints — the invite API route AND `accept_invite()`. Guarding only the
  route means a capped trainer just shares their link and the cap never applies.
- **Trial reminders (day 23/28/30) → Phase 2, via Vercel Cron** (no pg_cron).
- "Active client" = `trainer_clients.status = 'active'`. The enum already exists
  (`pending|active|inactive`), so archive is one write — only the UI action is
  missing. `admin_trainer_overview()` already counts identically.
