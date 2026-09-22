---
name: feedback-never-start-dev-server
description: "Owner 2026-08-31: do NOT start `npm run dev` — not even when asked to look at a page. next dev eats 3–5 cores and 5GB within tens of minutes on this machine. He starts it himself when he wants it."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 671c6857-3813-43fe-928d-27deea8601cd
  modified: 2026-08-31T14:43:44.038Z
---

**Standing instruction: never run `npm run dev` (or otherwise start a server).**
Owner's words, after the second time in one session: *"nu le mai porni, imi
strici laptopul."*

## Why

`next dev` on this repo reliably becomes a resource hog — measured twice on
2026-08-31 alone: 520% CPU / 5.4 GB after ~70 minutes, then 337% CPU / 4.5 GB
just 15 minutes after a clean `rm -rf .next` restart. System load climbed past
8 on a laptop the owner is also using for everything else. See
[[dev-server-runaway-cpu]] for the measurements and for what has been ruled out.

The second measurement matters: it happened with **zero builds** run against it,
which disproved the tidy explanation offered an hour earlier. The cause is still
unknown, so there is no "safe way" to leave one running.

## How to apply

- **Do not start a dev server, even to verify a change.** Run the parts of the
  gate that need no server — lint, type-check, tests — and say plainly that the
  visual check needs him to start it.
- **When he asks to look at a page**, give the URL and let him start the server.
  Remember the dashboard lives on `dashboard.localhost:3000`, not bare
  `localhost` — middleware redirects.
- **If one IS running and misbehaving**, killing it is welcome; starting it is
  not. Measure before claiming: `ps aux` for the process, `uptime` for load.
- Same restraint for anything else long-lived on his machine.

⚠️ He asked once, explicitly, and it had already been started twice. Do not
treat a later "yes" to some other question as permission to start one again —
this instruction outlives the task that prompted it.
