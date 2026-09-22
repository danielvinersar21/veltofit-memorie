---
name: instagram-content-brief
description: "The Instagram content pass: 50 posts triaged against what the product actually does — 9 keep, 19 redo, 22 delete. Approved 2026-09-02."
metadata: 
  node_type: memory
  type: project
  originSessionId: 48b3cecc-f667-4664-8bdf-84950911729d
  modified: 2026-09-02T12:26:26.943Z
---

Instagram content work for Velto, 2026-09-02. Two artifacts:

- **Brief + the prompt** — https://claude.ai/code/artifact/bcf75513-3bdf-40a2-821c-57a4981f9f9f
  ("Postările care nu mint"). Holds the full prompt written to be pasted into a
  fresh Claude Design conversation over the posts canvas, plus the findings.
- **The 50 posts** — https://claude.ai/code/artifact/f9aeaceb-674e-4c67-a9d8-51975ca4dcd7
  ("Velto IG — Direcții vizuale"), the design canvas the prompt operates on.

**State: triage done and APPROVED — 9 PĂSTREZ · 19 REFAC · 22 ȘTERG.**
Sarcina 2 (rewriting the 19) was the next step. Sarcina 3 (post-launch set on
five rotating pillars) and Sarcina 4 (the "Ce NU face Velto (încă)" post) come
after.

**Why so much got cut — the reusable part.** The set was written as if Velto
already had customers and features it does not. Deletions clustered exactly
where we have no standing: 6 of 8 "sfaturi" posts, 3 of 4 "industrie RO"
statistics posts, and all 3 "transformare" success stories. What survives is
product and honest observation.

**How to apply — the claims that must never come back:**
- Four features that DON'T exist: client↔trainer payments, in-app messaging,
  per-exercise video (the `video_url` column exists but nothing can upload to
  it), notifications (see [[notifications-never-worked]]).
- No invented numbers. The exercise library holds **55** global exercises
  (`docs/database/velto_exercises_seed.sql`), never "200+".
- No "în timp real" / "live" / "instant" for the trainer seeing client logs.
  Verified in code: zero realtime channels; sets are written ~800ms after
  typing stops, but the trainer only sees them on their own page load, served
  from a 30s–2min cache. Correct phrasing: "clientul își notează antrenamentul;
  tu vezi ce a făcut".
- No white-label / trainer branding claim. Revisit only if Robert's PDF report
  export ships with the trainer's name on it — see [[robert-contributor-tasks]].
- No beta / early access / limited seats — contradicts [[launch-gtm-decision]].
- No testimonials. There are no customers yet.
- Never address the client; the feed is for trainers.

Tone reference posts, judged closest to the right voice: 31, 37, 39, 43.
First-person rules come from [[owner-never-a-personal-trainer]]; the product
claim it all sits on is [[velto-positioning-statement]].
