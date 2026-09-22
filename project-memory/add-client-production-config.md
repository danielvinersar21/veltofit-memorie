---
name: add-client-production-config
description: "Production deployment checklist for the add-client invite flow (env vars, Supabase config, email) — must be done before it works in prod."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

**STATUS: DONE (2026-07-22).** All production config completed + invites verified working live (both channels). Kept as reference. Summary of what was set: Vercel env (`NEXT_PUBLIC_APP_URL=https://app.veltofit.app`, `SUPABASE_SERVICE_ROLE_KEY`); Supabase Auth Site URL = `https://app.veltofit.app` (fixed the "localhost" text in invite emails) + Redirect URLs for all 3 subdomains (app/dashboard/veltofit.app); custom SMTP via Resend (domain veltofit.app verified via the Vercel auto-configure, sender noreply@veltofit.app) — removes the ~3-4/hr built-in limit (now 30/hr, adjustable in Auth → Rate Limits). Original checklist below.

The **add-client invite flow** (branch `feat/add-client-flow`, merged fixes through commit `b3d81af`) is verified working LOCALLY but needs this configuration before it works in **production**. Do not forget these:

1. **`NEXT_PUBLIC_APP_URL=https://app.veltofit.app`** (the app subdomain). Critical: the claim/set-password page + `/client/home` must be on the SAME origin — the Supabase auth session is localStorage-scoped per origin, so landing the claim on a different subdomain than the app runs on causes a session loss + login bounce. Locally this is `http://app.localhost:3000`.
2. **`SUPABASE_SERVICE_ROLE_KEY`** — add on Vercel (Project Settings → Environment Variables), server-only, NEVER `NEXT_PUBLIC_`. Used by the `/api/clients/invite` route handler's admin client (`inviteUserByEmail`).
3. **Supabase Auth → URL Configuration → Redirect URLs** — add `https://app.veltofit.app/invite/set-password` (and the `/**` wildcard). Without it the invite email link's redirect is rejected / session not attached.
4. **Email delivery** — `inviteUserByEmail` uses Supabase's built-in email, which has low default rate limits (~3-4/hour). Configure **SMTP or Resend** in Supabase before real use.

DB migration `docs/database/add_client_flow.sql` is already applied to the live Supabase (done during local testing).

Related: [[velto-project-state]].
