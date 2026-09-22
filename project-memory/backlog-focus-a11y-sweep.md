---
name: backlog-focus-a11y-sweep
description: "Deferred 2026-08-20: 14 places outside the measurements branch where keyboard focus has NO visible indicator, with exact files and lines. Owner chose to defer; do not re-propose without asking."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6910664b-32db-48cf-aa94-0b89819b0c4d
  modified: 2026-08-20T11:40:14.724Z
---

**Owner decided 2026-08-20 to defer this as a separate accessibility task.** Do not fold it into
an unrelated branch, and do not re-raise it as urgent — he has seen the list and made the call.
Raise it again only if he asks about accessibility, or if someone touches one of these files
anyway.

## Why these were invisible until now

The `focus-ring` mixin emitted `rgba(var(--primary), .2)`, which Sass cannot composite, so the
declaration was invalid and dropped — **no element using the mixin had a focus ring at all.**
Once that was fixed (see [[backlog-review-round-2026-08-20]] item 3), rings appeared everywhere
*except* these sites, which suppress the global `:focus-visible` fallback with their own bare
`outline: none`. So they do not have an inconsistent indicator; **they have none.**

## The list (verified by design-reviewer, 2026-08-20)

**Nine — `--primary-20` box-shadow + `outline: none`** (~1.2:1, fails WCAG 2.2 SC 1.4.11):
`components/ui/date-input:36,55` · `features/…/exercise-autocomplete:40,42` ·
`features/…/recent-sessions:35,48` · `features/…/trainer-activity-feed:32,45` ·
`features/…/workout-overview-drawer:33,42` · `features/…/template-picker:39-40,82-83` ·
`features/…/set-presets:45-46` · `app/(app)/plans/[id]:44-45` · `app/(app)/client/home:129,146`

**Worse than the nine — the indicator IS the 20% tint:**
`app/(dashboard)/dashboard/styles.module.scss:78-81` — `outline: 2px solid var(--primary-20)`,
and on `:focus` not `:focus-visible`, so it also fires on mouse click.

**Colour-only indicators — these fail SC 1.4.1 as well:** `outline: none` plus nothing but a
`border-color` change. `features/plans/components/plan-exercise-chip:281+284, 298+301` ·
`app/(app)/trainer/plans/create/wizard.module.scss:177+180` ·
`app/(dashboard)/dashboard/plans/create/wizard.module.scss:218`

**Inset `--primary-20`:** `app/(app)/trainer/clients/[id]:74+84` ·
`app/(dashboard)/dashboard/clients/[id]:412+421`

**Hygiene, 12 sites** — `&:focus-visible { @include focus-ring; }` nests a mixin that already
contains its own `&:focus-visible`, compiling to a dead `outline: none` plus a doubled pseudo:
`dashboard/workouts/create:55,121` · `dashboard/workouts/[id]/edit:56,122` ·
`(app)/trainer/library:56,237` · `(app)/trainer/workouts/create:42,108` ·
`(app)/trainer/workouts/[id]/edit:43,109` · `(app)/trainer/home:188` ·
`active-plan-card:30` · `client-list-item:36`

## How to fix when the time comes

Not by hand, site by site. Use `focus-ring` for normal elements and `focus-ring-inset` (added
2026-08-20) for anything full-width and flush to its container edge, where an outset ring paints
over its neighbours. The pattern to remove is always the same: a bare `outline: none` plus a
hand-written `--primary-20` shadow. `--primary-20` is a decorative tint and must never be the
indicator.

Already fixed in the 2026-08-20 round, do not re-report: the mixin itself, Button, Input, Select,
Textarea, Badge, Checkbox (whose ring was clipped by an `overflow: hidden` wrapper and had
*never* rendered), `measurements-section`, and `top-nav`'s create-menu (fully clipped).

Related: [[backlog-review-round-2026-08-20]], [[measurements-bilateral-status]].
