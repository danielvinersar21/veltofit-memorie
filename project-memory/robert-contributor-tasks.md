---
name: robert-contributor-tasks
description: "Robert (owner's brother) is a second contributor on Velto — his skill level, and the two plans written for him 2026-08-15."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4858c9de-810d-4580-afee-bf08c6ca7d05
  modified: 2026-08-15T08:57:26.838Z
---

**Robert (Varvara Robert, `varvararobert41@gmail.com`) is a second contributor on Velto** —
the owner's brother. He uses Claude Code too. Git history shows two commits, and they are
**not** trivial: `feat(admin): add read-only platform dashboard v1` (a whole feature) and
`fix(admin): sidebar highlights only the current subroute`.

**How to apply:** write briefs for him at the level of goals + constraints + acceptance
criteria, not step-by-step instructions — he can handle a real feature. The owner explicitly
wants him to **learn infrastructure work** (Supabase buckets, storage policies), so don't
strip that out of a plan to make it easier, and don't hand him copy-paste SQL — give the
security requirement and the reasoning and let him work it out.

Two plans written for him on 2026-08-15:
- `docs/plans/profile-photo-upload-v1.md` (S) — includes the Supabase bucket + policies as
  the deliberate learning half.
- `docs/plans/report-pdf-export-v1.md` (M) — print CSS + `window.print()`, owner's choice
  over jsPDF/Puppeteer. **IN PROGRESS as of 2026-08-20** — the owner confirmed Robert has
  picked this up, so treat #5 in the backlog as taken, not open. Profile photo (the other
  plan) shipped as PR #16.

**Two corrections worth remembering, both from assuming a backlog line was still true:**
1. "Profile photo upload was never built" was **wrong** — it is ~95% built and orphaned
   (service + hook + a full `ProfileImageUploader` component that is not exported from the
   barrel and used nowhere). Always grep the codebase before repeating a stale backlog entry.
2. The owner said we could drop Romanian diacritics from the PDF — that concern only applied
   to the jsPDF option we did **not** pick. With browser print-to-PDF, Inter/Sora render
   ă/â/î/ș/ț for free, so the plan keeps them.

See [[backlog-owner-priorities-2026-08]] for where these sit in the overall order.
