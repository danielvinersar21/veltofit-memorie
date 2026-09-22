---
name: trainer-signup-source
description: "Where each trainer came from — BUILT 2026-09-21. SQL `trainer_signup_source.sql` APPLIED + VERIFIED in prod 2026-09-21 (17/17 checks OK). Frontend PUSHED 2026-09-21 (8a29935..5afff65). LIVE."
metadata: 
  node_type: memory
  type: project
  originSessionId: 93a5f689-8aef-49ab-bdcb-53f72a7d8d01
  modified: 2026-09-21T09:00:40.810Z
---

Owner decided 2026-09-21: capture BOTH automatically (referrer hostname + utm tags,
carried in the link landing → signup form, passed as `signup_source` user metadata at
signUp; nothing new stored in the browser) AND an optional question at the trainer's
first complete-profile. City ("Localitate") required for trainers at first
complete-profile only — still optional in profile edit (deliberate, the owner's
wording was "la prima completare"). Shown on the admin trainer page.

**SQL APPLIED in prod 2026-09-21** by the owner, guided: pre-flight 12/12 OK
(11 trainers, 5 with no city), whole file "Success", then a combined verification
script (scratchpad, not in repo — built from the file's 9 commented checks, with
probes rolled back) 17/17 OK: RLS + zero policies, no browser grants, FK CASCADE to
auth.users, trainer can't read table/admin fn/cleaner, role=admin self-set still
refused, answer RPC cleans input, 15/15 normaliser cases incl. diacritics, all 11
trainers have a 'backfill' row, trigger captures + resists forgery, write-once holds
even as postgres, client can't answer, admin reads 16 columns and never a client.
**Frontend PUSHED 2026-09-21** as 4 commits `8a29935` (feature) `08c9fc7` (error
contrast) `712a73f` (legal) `5afff65` (backlog/state), pre-push green. Before that the
owner tested end to end locally with a fresh account (campaign → signup → confirm →
complete-profile → admin card showed verificare2 / test / proba / Google). Both test
accounts deleted with delete_test_user.sql.
Local trap found: the Supabase Redirect URLs lacked `http://dashboard.localhost:3000/**`,
so a local dashboard signup confirmed onto PRODUCTION app (Site URL) and the owner
completed the profile there on the old code. Owner added the entry 2026-09-21. Prod
was never affected (`https://dashboard.veltofit.app/**` is listed).

**Why the order matters:** SQL first → answers stored from day one. Frontend first →
auto-capture waits safely in metadata (backfill catches it) but answers given before
the SQL are lost.

**How to apply:** pre-flight (select "SELECT * FROM (" … "ORDER BY nr;", run alone,
expect 12× OK) → whole file (BEGIN/COMMIT, all-or-nothing) → verification 1–9 incl.
the diacritics row in 5. Commit only after the owner says so; stage by path — the
`.claude/skills/velto-social/**` changes in the tree are unrelated work.

Crăciun and Dumitru (the two organic trainers) will show "Necunoscut" forever.
Instagram links need utm tags or most Instagram signups read "Direct".
The 13 deferred follow-ups are in `docs/plans/backlog.md` (21 sep entries).
Related: [[legal-pages-hardening]], [[trainer-accounts-who-is-who]], [[free-has-no-trial]].
