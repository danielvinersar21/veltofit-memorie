---
name: add-client-followups
description: "Open follow-ups / TODOs for the add-client feature (things deferred, not yet done)."
metadata: 
  node_type: memory
  type: project
  originSessionId: c986b52c-f9db-4113-8b45-34922f12ceb4
---

All three add-client follow-ups are RESOLVED (2026-07-22):

1. ~~**Client name in the trainer's Clients table.**~~ **DONE.** Rule reordered to prefer **full_name** first (a client's display_name may be just a first name → many clients read as "Daniel"; full_name is distinguishing): `full_name → display_name → invite.display_name → invite.email → fallback`. Applied in both `clients-api.ts` name resolvers (list + `listLatestActiveClients`).

2. ~~**Reusable-link "revoke".**~~ **DECIDED: no button (closed).** User confirmed the reusable link stays stable — no revoke/regenerate control. `rotateReusableLink` still exists in the service if a real need ever arises.

3. ~~**Trainer deletes a client.**~~ **DONE.** Decision: unlink + clear the plan (keep the client's standalone account/history/own data, re-invitable). `removeClientFromTrainer` now, BEFORE deleting the trainer_clients row, calls the existing SECURITY DEFINER RPC `unassign_plan_from_client(clientId, assignedPlanId)` — clears `profiles.assigned_plan_id` and marks `client_plan_progress` abandoned (a trainer can't write the client's profile/progress under RLS, and the RPC authorizes on the ACTIVE relationship, hence the ordering). Dashboard remove-confirm message updated to say the plan is retracted but the account/history stay + re-invitable. Removal UI is dashboard-only.

See [[add-client-production-config]] for the prod deployment checklist (still the only open add-client item).
