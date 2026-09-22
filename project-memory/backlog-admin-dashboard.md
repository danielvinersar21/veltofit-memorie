---
name: backlog-admin-dashboard
description: "Platform-owner (super-admin) dashboard — BUILT and extended with demo/real split + dedicated admin role."
metadata:
  node_type: memory
  type: project
  originSessionId: 84202102-dd0c-43e7-b5c8-30c33256d5a9
  modified: 2026-08-04T13:47:42.778Z
---

Read-only founder/super-admin dashboard. **Built and shipped to main** 2026-08-04: `0ea14ea` (v1, by Varvara Robert) + `844ce07` (demo/real split + admin role). Fast-forward merge, pushed.

Full spec: `docs/plans/admin-dashboard-v1.md`. DB: `docs/database/admin_dashboard.sql` (applied live).

**Two superseded decisions from the original brainstorm:**
- ~~Admin = Daniel's existing trainer account~~ → now a **dedicated account** with its own `role = 'admin'`. The CHECK constraint on `profiles.role` was widened to `('trainer','client','admin')`.
- ~~Hidden route only~~ → admins get `ADMIN_SIDEBAR_ITEMS` (Platformă / Date demo). Still absent from the trainer sidebar.

**Account setup (not derivable from code):**
- `daniel.vinersar.ux@gmail.com` = super admin, `role='admin'`, in `admin_users`. Was previously the gym test-client; that account was renamed off it.
- `d.vinersar21@gmail.com` = the gym test-client (`ac027227-47cd-429c-bf1b-cf470a472b59`), `is_demo=false` so it still counts in real stats — open question whether to flag it demo.
- Renaming a user's email is only possible via the GoTrue Admin API — the Supabase panel exposes no edit control. Script kept at scratchpad `change-email.mjs` (dry-run by default). `admin_users.user_id` is a UUID FK, so it follows the ACCOUNT through a rename, not the address.

**Security invariant:** `profiles.role='admin'` is NAVIGATION ONLY. `profiles` has a `FOR ALL USING (id = auth.uid())` self-policy, so any user can write their own role — authorization must stay in `admin_users` behind `is_platform_admin()`. Never gate data on role.

**Follow-ups:** signups sparkline, client drill-down, per-trainer Plans/Activity. See also [[backlog-supabase-db-health]].
