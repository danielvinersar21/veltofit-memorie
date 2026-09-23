---
name: qa-gate-blind-spots
description: "The QA gate does NOT catch unused imports or unused locals in this repo — eslint has no no-unused-vars, tsconfig has no noUnusedLocals. Verified 2026-08-21."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-09-23T19:33:28.931Z
---

`npm run lint && type-check && test && build` passes with **unused imports and
unused local variables still in the file**. Confirmed 2026-08-21 after deleting a
`useBilling()` call and leaving its import behind: all four commands went green.

Why: `eslint.config.mjs` extends only `eslint-config-next/core-web-vitals`, which
does not enable `@typescript-eslint/no-unused-vars`; and `tsconfig.json` has
`strict: true` but neither `noUnusedLocals` nor `noUnusedParameters`.

**How to apply:** after removing a hook call, a prop, or a destructured value,
grep for the symbol yourself — the gate will not tell you. This bites hardest
when acting on a review finding, because the edit is usually "delete one usage"
and the import is a separate line.

Enabling `noUnusedLocals` would close it, but that is a repo-wide change that
would surface existing violations — not something to switch on mid-task.

**The debt is now MEASURED, 2026-08-29:**
`npx tsc --noEmit --noUnusedLocals --noUnusedParameters --incremental false -p tsconfig.json`
returns **51 findings** (48 × TS6133 unused variable/import, 3 × TS6196 unused
type) and **zero real type errors** — consistent with the plain type-check
passing, so this is pure cleanup, not hidden breakage. Concentrated in dev
tooling, led by `ExerciseCard.stories.tsx` (12) and several `design-system/`
demo pages. That command is also the honest way to check your own edit: run it
and read only the lines for the files you touched.

Owner deferred the cleanup 2026-08-29 — it was surfaced mid-migration and mixing
it into that deploy would have been wrong. The durable fix, when it is done, is
to clear the 51 and THEN turn the flag on, so it cannot regrow.

## A second blind spot, in the TESTS: a `vi.fn()` swallows unhandled rejections

Found 2026-09-23 while testing the `last_seen_at` heartbeat, which is deliberately
fire-and-forget (`void touchLastSeen(...).catch(() => {})`). A test written to prove
the `.catch` net is needed — by deleting it and expecting Vitest to report an
unhandled rejection — **passed with the net removed**. It caught nothing.

Why: `vi.fn()` attaches its own handler to the promise it returns (that is how it
records the outcome), which marks any rejection as "handled". Vitest then never sees
an unhandled rejection and never fails the run.

**How to apply:** a test that asserts "the suite would fail on an unhandled
rejection" proves nothing when the rejecting call is a `vi.fn()`. Re-wrap the
promise inside the mock and register `process.on('unhandledRejection')` by hand in
the test. This matters for any deliberately-discarded promise — and this repo has
several, since fire-and-forget is the house pattern for telemetry-ish writes
(the heartbeat, the owner alerts, the MailerLite onboarding).

**The general lesson, worth more than the mechanism:** the way to know a test bites
is to break the source and watch it fail. The `last_seen_at` round did that for four
separate mutations (including `=== undefined` → `!lastSeenAt` and adding an `await`),
restored the sources byte-identically, and only then trusted the suite. Without it,
one of the four tests was quietly worthless.
