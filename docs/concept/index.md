---
icon: lucide/lightbulb
---

# Concept

`eliferpg-core`'s architecture has anticipated an "NPC Simulation Module"
as a future consumer of its Central API from early on (Client-Credentials
OAuth, subscribing to events, writing results back) but its shape was
never designed and no code exists for it yet. The immediate motivation is
an upcoming in-game **market feature**: for a player-driven market to feel
alive, it needs non-player supply and demand, not just whatever players
happen to be buying or selling at a given moment. `npc-virtual-simulation`
is that subsystem.

NPCs are simulated as real `Character`s in `eliferpg-core` — each with its
own bank account and shop — not a faceless aggregate. An NPC's economic
activity is just normal Banking/Shops traffic, so the market doesn't need
a bespoke pricing/ledger engine of its own. NPCs don't move or render
in-game; this is a pure backend economic simulation whose only visible
effect is listings, prices, and stock shifting in Shops.

## What this repo does and doesn't own

The domain nouns — who NPCs are, what they can produce, where their shop
is, what they hold — all live in `eliferpg-core`. `npc-virtual-simulation`
does not keep its own copy of any of that. It owns only **the pacing logic
and its tunable boundaries**: how many NPCs are active, how fast they act,
and how that scales with how many real players are currently online.

This split exists because recipes/production belong in `eliferpg-core`
(eventually via a **skill system** that determines what a Character can
produce — not yet designed), NPC location is a per-Shop position in core,
and NPC inventory is a real per-Character concept in core. See the
[Ownership Map](ownership-map.md) for the full breakdown.

`npc-virtual-simulation` is a **governor/scheduler** service sitting on top
of `eliferpg-core`, not a second domain model.

## Architecture

``` mermaid
graph TB
  subgraph GameserverHost["Gameserver Host"]
    GS[ArmA Reforger Gameserver]
    Bridge[Bridge Service<br/>eliferpg-reforger-bridge]
    GS --> Bridge
  end

  subgraph CoreGroup["eliferpg-core (Central API)"]
    CoreBox[Owns: NPC identity, skills/recipes,<br/>shop position, inventory, banking, shops]
  end

  Bridge -- "real players" --> CoreBox

  NPC[npc-virtual-simulation<br/>owns: boundaries, tick/scheduling engine]
  Staff[Staff / Admin UI]

  NPC -- "Client Credentials<br/>npc:read / npc:write<br/>no token exchange" --> CoreBox
  Staff -- "configure boundaries,<br/>inspect simulation state" --> NPC
```

`npc-virtual-simulation` never talks to the ArmA game server, the Bridge,
or players directly — only to `eliferpg-core`'s Central API (as a client)
and to staff/Admin UI (as a server, for boundary configuration and
inspection).
