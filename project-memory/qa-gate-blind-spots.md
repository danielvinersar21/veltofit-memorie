---
name: qa-gate-blind-spots
description: "The QA gate does NOT catch unused imports or unused locals in this repo — eslint has no no-unused-vars, tsconfig has no noUnusedLocals. Verified 2026-08-21."
metadata: 
  node_type: memory
  type: project
  originSessionId: dbc30056-f825-4104-b501-ac93e3096fc5
  modified: 2026-08-29T06:55:56.470Z
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
