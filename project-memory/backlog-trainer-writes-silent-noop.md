---
name: backlog-trainer-writes-silent-noop
description: "approvePlanForClient was a fully dead button — FIXED and proved in production 2026-09-06. The notification half stays open, and is parked. The pattern itself is the lasting lesson: RLS drops the write, PostgREST answers 200."
metadata:
  node_type: memory
  type: project
---

**Status: OPEN, to fix.** Owner asked for this as a ticket 2026-08-20. Found
during the RLS audit ([[rls-write-side-holes]]), preexisting, unrelated to the
billing work. Every claim below was verified by reading the code and the policies.

## The mechanism (this is the reusable part)

PostgREST `UPDATE`/`INSERT` that matches **zero rows because of RLS** is **not an
error**. It returns `200` with an empty body. Supabase-js reports `error === null`,
so a service that only checks `if (error) return {error}` returns **success** for a
write that never happened. Any trainer-side write to a table whose trainer policy
is `FOR SELECT` only is silently dead.

This is the same failure mode `trainer_plan_rpcs.sql` was written to fix for
assign/unassign — the lesson just wasn't applied to the neighbouring functions.

## What is actually broken

**1. `approvePlanForClient()` — `progress-api.ts:1038`. Completely dead.**
The trainer's "mark plan as completed" button. Both writes are made as the trainer:
- `client_plan_progress` UPDATE → `progress_trainer` (`velto_schema_refactored.sql:558`)
  is `FOR SELECT` only → 0 rows.
- `profiles` UPDATE on the CLIENT's row → `profiles_own` only matches the trainer's
  own row, `profiles_trainer_sees_clients` is SELECT only → 0 rows.

Returns `{ ok: true }`. Worse, `use-client-detail.ts:287` then does an **optimistic
local update** — moves the plan into the completed list in the UI. So the trainer
watches it work, and on the next refresh the plan is back. Live on BOTH
`(app)/trainer/clients/[id]` and `(dashboard)/dashboard/clients/[id]`.

**2. Trainer→client notifications never arrive.**
`notifications_own` (`:579`) is `FOR ALL USING (user_id = auth.uid())`. With no
`WITH CHECK`, Postgres reuses `USING`, so an INSERT is allowed only where
`user_id = auth.uid()`. Every trainer-authored notification carries
`user_id: clientId`, so it fails — and all of them are fire-and-forget, so nothing
surfaces:
- `progress-api.ts:987` `plan_assigned` — **a client is never told a plan was assigned to them**
- `progress-api.ts:1064` `plan_completed` (trainer path, inside the dead function above)
- `progress-api.ts:1161` `plan_cancelled`

Working by comparison: `completePlanForClient()` (`:1087`) is called by the CLIENT
on their own row, so `clientId = auth.uid()` and both the writes and the
notification succeed. The invite route (`route.ts:318`) uses `service_role`.

## The fix

A `SECURITY DEFINER` RPC, exactly the pattern already in
`docs/database/trainer_plan_rpcs.sql` (`assign_plan_to_client` /
`unassign_plan_from_client`) — same authorization check (caller must be the
client's ACTIVE trainer), same `SET search_path = public, pg_temp`.

- Add `complete_plan_for_client(p_client_id, p_plan_id)` doing both writes.
- **Move the notification INSERT inside the RPCs**, or it keeps failing even after
  the plan writes are fixed. This is the easy half to forget.
- Consider a shared `notify_client(...)` SECURITY DEFINER helper, since three
  separate call sites need the same thing.

## Guard against the next one

Grep for trainer-side `.update(`/`.insert(` on `client_plan_progress`, `profiles`,
`notifications`, `measurements` that don't go through an RPC. The service layer's
`if (error)` check does NOT prove a write landed — to prove it, request the
affected rows back (`.select()`) and assert a non-zero count.

---

## ✅ ÎNCHIS 2026-09-06 — `approvePlanForClient`

Reparat prin RPC `complete_plan_for_client` (SECURITY DEFINER, autorizat pe o
legătură ACTIVĂ din `trainer_clients`, ambele scrieri într-o tranzacție,
întoarce numărul de rânduri — orice altceva decât un număr pozitiv e eroare).
Migrarea aplicată în producție de owner pe 2026-09-06.

**Era mort de două ori, nu o dată.** Nu doar scrierea pe `profiles`: și cea pe
`client_plan_progress`. Antrenorul are doar SELECT acolo, iar singura politică de
scriere cere `client_id = auth.uid()`.

**Probat pe viu**, nu doar în teste: buton apăsat pe fișa unui client, apoi citit
din bază — un rând `completed`, `completed_at` scris, plan golit din profil.

Două lucruri găsite chiar în timpul probei:
1. **Butonul e practic inaccesibil pe drumul normal.** Apare doar când progresul
   e `completed` ȘI profilul încă ține planul — dar auto-finalizarea clientului
   le face pe amândouă deodată, deci starea aia nu apare de la sine. E un buton
   pentru o stare intermediară. Merită regândit.
2. **Duplica intrarea în „Planuri finalizate"** la fiecare apăsare, fiindcă
   planul era deja în listă prin chiar statusul care scoate butonul. Reparat în
   același commit.

Jumătatea cu notificările (`plan_assigned` / `plan_cancelled`) rămâne deschisă și
e **parcată deliberat** de owner din 2026-08-30.
