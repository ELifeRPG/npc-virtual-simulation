---
icon: lucide/gauge
---

# Scheduler & Boundaries

With domain state living in `eliferpg-core` (see
[Ownership Map](ownership-map.md)), this repo's own domain is intentionally
thin — it's a governor sitting on top of `eliferpg-core`, not a second
economy model.

## What this service owns

- **Boundary configuration** — the tunable knobs staff actually operate:
    - Max active population (global, and/or scoped by whatever dimension
      the skill system exposes — e.g. per skill, per region).
    - Rate limits — how much production/consumption activity is allowed
      per tick, independent of the skill system's own per-recipe amounts.
    - Activity-scaling curve — how the *fraction* of the population that's
      active scales with the current online-player count.
- **Tick log** — a per-tick record of what was scheduled/triggered and any
  errors, for admin visibility/debugging. Not a copy of NPC economic state
  — that stays queryable from `eliferpg-core`.
- **The scheduler itself** — decides, each tick, *how many* and *which*
  eligible NPCs (queried from `eliferpg-core`'s Characters, filtered to
  `OwnerKind=Npc`) get to act, within the configured population cap and
  activity-scaling factor, then triggers their action in `eliferpg-core`.

## Tick timing is not owned here

`npc-virtual-simulation` does not define its own tick interval or
in-game-time-scale config. Ticks and in-game time are controlled by either
the core backend or the gameserver itself, and either way `eliferpg-core`'s
`GameServer` settings are the source of truth (see
[Ownership Map](ownership-map.md#eliferpg-core-prerequisites), item 5). The
scheduler reads that per-gameserver setting via the Central API and paces
itself against it, rather than owning a competing notion of time.

Since `GameServer` settings and NPC `Character`s are both scoped
per-gameserver-instance (per `eliferpg-core`'s multi-gameserver-tenancy
model), the scheduler naturally operates **per gameserver instance**, not
as one global clock.

## Population scaling by online player count

Rather than polling a new `eliferpg-core` endpoint, `npc-virtual-
simulation` subscribes to the Central API's existing event pipeline
(SignalR live push + `GET /events?since=<sequence>` catch-up — the delivery
mechanism already designed for this module) and consumes
`CharacterSessionStarted`/`CharacterSessionEnded` events to maintain a live
online-player count. That count feeds the configurable activity-scaling
function above (e.g. the active-NPC fraction rises as online players drop,
keeping the market from going quiet when few players are online) — the
exact curve is an admin-tunable boundary, not hardcoded.

``` mermaid
sequenceDiagram
    participant Core as eliferpg-core
    participant Sim as npc-virtual-simulation
    participant Staff as Staff / Admin UI

    Staff->>Sim: configure boundaries (max population, rate limits, scaling curve)
    Core-->>Sim: CharacterSessionStarted / Ended (SignalR + catch-up)
    Sim->>Sim: update live online-player count
    loop each gameserver tick (paced by Core's GameServer settings)
        Sim->>Core: query eligible NPC Characters
        Sim->>Sim: select active subset (population cap × scaling factor)
        Sim->>Core: trigger NPC action(s)
        Sim->>Sim: record TickLog
    end
```

## Open question: the trigger mechanism

Exactly *how* the scheduler triggers an NPC's action depends on the shape
of `eliferpg-core`'s future skill system:

- A single endpoint (e.g. `POST /api/npc-characters/{id}/act`) that runs
  core-side skill logic and settles it via Banking/Shops internally —
  `npc-virtual-simulation` as a pure pacer.
- `npc-virtual-simulation` reading skill/recipe data itself and driving
  Banking/Shops endpoints directly — `npc-virtual-simulation` as pacer
  *and* executor.

This can't be settled until the skill system's own design exists in
`eliferpg-core` — it's the key open follow-up for this repo.
