---
name: backlog-owner-priorities-2026-08
description: "Owner's 2026-08-15 backlog re-ordering — offline PWA first, archive-client deliberately deprioritized. As of 2026-08-20 all five have specs and #4 has shipped."
metadata: 
  node_type: memory
  type: project
  originSessionId: 4858c9de-810d-4580-afee-bf08c6ca7d05
  modified: 2026-08-15T18:15:24.406Z
---

On 2026-08-15 the owner re-ordered the whole Velto backlog himself. Top 5, in his order:
**1. mod offline real (PWA)** → **2. media pentru exerciții (video demo)** → **3. editare plan
existent prin wizard** → **4. upload poză profil** → **5. raport exportabil (PDF/share)**. Everything
else (landing polish, nutriție/calendar/comunicare, Stripe) sits below these.

**Explicitly deprioritized: arhivare clienți** — "aplicația e în testare, nu o să fie neapărat
folosit". Do not re-propose it as important.

**AI added the same day, scope chosen, deliberately left unprioritized.** Owner parked it in the
backlog with no rank — a settled decision, so don't ask where it goes again. He picked three
from a shortlist:
**generare plan cu AI**, **insights pe progres**, **auto-progresie greutăți**. He **declined the
client-facing chat assistant** — don't re-propose it without a new signal. Note the third one
doesn't need an LLM at all (arithmetic over `actual_weight`); build it rule-based first.
Repo has zero LLM infra, but `src/app/api/` already has two routes, so a server-side AI route is
an established pattern rather than a new one.

**Why:** the app is in validation, so the owner wants depth for the people actually using it rather
than revenue plumbing or public-site polish. Offline is #1 because every gym incident since July
(freeze on resume, lost set writes, the 25kg→2kg bug — see [[backlog-set-write-ordering]] and
[[backlog-active-workout-freeze]]) traces back to the tracking screen being network-dependent at the
worst moment.

**All measurement data in the app is fictional** (owner, 2026-08-15) — and by extension treat
demo/seed data as disposable unless he says otherwise. This unblocks destructive schema changes
that would be unacceptable against real data: the bilateral-measurements plan drops the old
`arms`/`quads`/`calves` columns outright instead of carrying a compatibility layer. Ask before
assuming the same about `workout_set_performance` — the gym-logged weights may be his own real
training. Spec: `docs/plans/measurements-bilateral-v1.md`, also parked without priority.

**STATUS 2026-08-20 (verified against the repo, supersedes the "no specs yet" note below).** All
five now have spec docs in `docs/plans/`: `offline-workout-v1.md` (#1), `exercise-media-v1.md` (#2),
`plan-editing-v1.md` (#3), `profile-photo-upload-v1.md` (#4), `report-pdf-export-v1.md` (#5).
**#4 profile photo is SHIPPED** — merged to main as PR #16 (`8340e40`). **#5 raport PDF is
IN PROGRESS — Robert owns it** (owner, 2026-08-20); don't schedule it as open work. #2 has a started branch,
`feat/exercise-media` (`c71a828`, "phase 1, UNVERIFIED"), the only feature branch unmerged from
main. So the live top of the queue is **#1 offline PWA**, with #2 half-built.

**How to apply:** the top 5 were plan-writing candidates when this was written; that is now done. Plan #2 and #4 together (one Supabase Storage bucket + RLS serves both).
Scope #1 as an offline *write* path for the active workout (queue + replay + conflict rules,
overlapping the existing sessionStorage backup in `use-active-session.ts`), NOT as "turn on
next-pwa" — `next-pwa` is installed but unwired, and enabling it alone buys nothing for this.
Consequence to keep in mind: deprioritizing archive leaves the Stripe seat-enforcement blocker in
[[pricing-monetization-plan]] open by choice. Full list lives in the repo at
`.claude/memory/velto-project-state.md` under "Backlog (prioritized) — reordered 2026-08-15".
