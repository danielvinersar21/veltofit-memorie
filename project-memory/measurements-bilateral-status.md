---
name: measurements-bilateral-status
description: "Bilateral measurements (16-field set) — SHIPPED: migration APPLIED and verified 2026-08-20, code merged into origin/main. Only demo re-seed remains."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6910664b-32db-48cf-aa94-0b89819b0c4d
  modified: 2026-08-20T07:41:06.818Z
---

**Built 2026-08-20 as `f7e8ca7` on branch `feat/measurements-bilateral`. Merged into
`origin/main` (confirmed 2026-08-29 — the branch is an ancestor of origin/main, 0 unpushed
commits). Spec: `docs/plans/measurements-bilateral-v1.md`.**

**✅ MIGRATION APPLIED AND VERIFIED 2026-08-20.** `measurements_full_set.sql` ran successfully
after `measurements_clear_legacy_unilateral.sql`. Verified end to end, not assumed:
- Save works from the dashboard with ONLY one side of a pair filled (partial completion is
  valid), which also proves PostgREST reloaded its schema cache — the INSERT names `arm_left`
  explicitly and would fail otherwise.
- RLS isolation holds: impersonating a client via `SET LOCAL ROLE authenticated` +
  `set_config('request.jwt.claims', …)` returned **3** own rows and **0** of another client's,
  and `pg_class.reloptions` shows `security_invoker=on`. The July RLS-bypass fix did NOT
  reopen — the thing most at risk from a DROP+CREATE VIEW.
- The positive control mattered: two zeros would have meant "impersonation failed", not
  "isolation works". Always assert both directions when testing RLS this way.

**The abort guard earned itself on the first run:** it found 96 rows still holding
`arms`/`quads`/`calves` and rolled the whole transaction back — the owner's "all data is
fictional" was true, but the data was still there. `measurements_clear_legacy_unilateral.sql`
was written to clear them, scoped to `is_demo` and aborting if any legacy row fell outside that
set. No carry-over into the new columns, deliberately: a unilateral value holds no side
information, and copying it into both sides fabricates a perfect symmetry inside the very
feature built to show asymmetry.

**Still open:** demo data. `velto_mock_clients_seed.sql` creates SIX NEW mock clients with fresh
UUIDs — it does not refresh the accounts whose legacy values were cleared, so those keep their
weight/height/chest and show em dashes for `neck` and all eight bilateral fields permanently.
Owner deferred running it; decide before any `capture:marketing` run.

Three traps live in that SQL, all called out in its header:
1. `DROP VIEW` before `ALTER TABLE` — `CREATE OR REPLACE VIEW` cannot remove columns, and we
   remove three (`arms`, `quads`, `calves`).
2. `security_invoker = on` must be re-stated in the new `CREATE VIEW`. It is NOT inherited.
   Losing it reopens the CRITICAL RLS bypass fixed 2026-07-27 — see
   [[backlog-supabase-db-health]].
3. `DROP VIEW` discards all GRANTs. Symptom is "permission denied for view
   client_measurements", which does not look like a privileges problem.

**Destructive on purpose:** `arms`/`quads`/`calves` values are dropped, no compatibility
layer, because the owner confirmed all measurement data is fictional (see
[[backlog-owner-priorities-2026-08]]). If that ever stops being true, this migration is
wrong as written. Demo data needs re-populating afterwards for
`npm run capture:marketing` and `is_demo` accounts.

**Design note worth keeping:** `MEASUREMENT_GROUPS` (single | pair) is now the source of
truth in `measurements-api.ts`, and the flat `MEASUREMENT_FIELDS` is derived from it. Form
state, insert payload and save handler all map over the list rather than spelling out the
fields — the old hand-written triplication is how `calves` once reached the service but
never the view. Add a field in the groups list and the rest follows.

**Layout — settled after two rounds of owner feedback 2026-08-20. Do not re-litigate:**
a pair takes ONE half-width cell, like any single field, with Stânga/Dreapta **side by side**
inside it. The first pass gave each pair a full-width row; the owner rejected it because on a
wide dashboard card it tore the 50/50 rhythm of the singles above and left a canyon between
the sides. A second pass stacked the sides vertically inside the half cell; he then asked for
horizontal, correctly — measured, a pair cell is 492px on dashboard and 172px at phone width,
against a 67px worst-case value, so two columns fit even on mobile (the delta chip wraps under
its value there). Adjacency is also the point: left vs right is only comparable side by side.

**History-list fixes that came out of the same review (2026-08-20).** Three separate bugs in
`MeasurementsSection`'s history, all owner-reported from a real dashboard screenshot:
the expanded body was a `1fr 1fr` grid, so once FieldGrid brought its own grid the field set
rendered at half width; an open row bled 12px outward per side and so grew wider than its
neighbours; and the row button had no horizontal padding, so the hover fill ran flush against
the date. The list is now square-cornered on purpose — its dividers are straight edge-to-edge
lines and a rounded fill leaves a wedge of gap where they meet. Don't reintroduce
`border-radius` there.

Preview without a session: `/design-system/measurements-section` seeds the zustand store
directly (the component is data-connected and never fetches when logged out). It has three
frames — phone, dashboard width, and history — because the layout bug only showed at width.
