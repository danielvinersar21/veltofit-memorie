---
name: backlog-health-notes-consent-proof
description: "TO DO before real users: health-notes consent is recorded in auth user metadata — works, but is not demonstrable proof under GDPR. Owner deferred 2026-08-22."
metadata:
  node_type: memory
  type: project
---

**Status: OPEN, deferred by the owner 2026-08-22** — deliberately, because zero
clients have been through the consent flow yet, so there is nothing to prove
retroactively. Do it before real signups arrive.

## What exists now

Client control over `client_profiles.health_notes` was built 2026-08-22: the
client can edit and delete their own health notes, and on first entry a panel
shows what the trainer wrote about them with keep / edit / delete.

The "they have seen and accepted this" stamp lives in
**`auth.users.raw_user_meta_data`** (`health_notes_reviewed_at`, written via
`supabase.auth.updateUser`). Chosen because it needs no new column and survives a
device change, unlike localStorage. The stamp's value is `client_profiles.updated_at`
(server time), not the phone clock — otherwise a client with a slow clock is asked
forever.

## Why it is not enough

GDPR art. 7(1) requires being able to **demonstrate** consent. This stamp:
- is written by the data subject and can be overwritten by them,
- is invisible from inside the app (only in Supabase → user metadata),
- **keeps no copy of the text that was consented to**, so even if the timestamp
  is trusted, what they agreed to is unknowable after the trainer edits the note.

Also imprecise: the panel reappears whenever `client_profiles.updated_at` moves,
which includes the trainer editing `goal` or `phone` — nothing records *which*
column changed. It errs toward asking too often, which is the safe direction.

## The fix when it is time

`client_profiles.health_notes_reviewed_at timestamptz` plus
`health_notes_reviewed_text text` (a snapshot of what was shown). Optionally
`health_notes_updated_by uuid` so the panel only reappears when the *trainer*
touched the notes, not the client.

⚠️ Write-protect the new columns properly. **Do not use a column-level REVOKE** —
see [[column-revoke-is-a-noop]], it does nothing when a table grant exists. The
client legitimately writes `health_notes` itself via `client_profiles_owner`
(`FOR ALL`), so a trigger enforcing "reviewed_at only moves forward, and only via
the consent path" is the shape that works.

## Second, smaller item from the same work

**The trainer is never told when a client deletes their health note.** They just
see "—" next time they open the file, with no idea it was ever filled. A
notification on deletion would be honest to the trainer, whose programming may
have depended on knowing about an injury. Not built; not asked for.
