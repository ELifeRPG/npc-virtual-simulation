---
icon: lucide/house
---

# NPC Virtual Simulation

A backend service that drives ELifeRPG's non-player economic activity —
NPCs that produce and consume goods through the same Banking/Shops
infrastructure real players use, so the in-game market has supply and
demand even when players don't.

This site documents the subsystem in depth. For a short, high-level
summary alongside the rest of ELifeRPG's architecture, see the
[NPC Simulation](https://elife-rpg.github.io/docs/development/npc-simulation/)
page on the main docs site.

**Status: concept stage.** No application code exists yet — see
[Concept](concept/index.md) for the design and
[Development](development/index.md) for what's blocking implementation.

## Where to start

- [Concept](concept/index.md) — why this exists, how it fits with
  `eliferpg-core`, and what this repo does vs. doesn't own.
- [Ownership Map](concept/ownership-map.md) — which repo owns which piece
  of NPC-related state.
- [Scheduler & Boundaries](concept/scheduler.md) — how the simulation
  paces NPC activity and scales it with online player count.
- [API Reference](api/index.md) — the admin/staff-facing endpoints this
  service will expose.
- [Development](development/index.md) — setup, dependencies, and current
  implementation status.
