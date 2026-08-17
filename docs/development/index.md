---
icon: lucide/code
---

# Development

## Status

**Concept stage — no application code exists yet.** This repo currently
holds only the design ([Concept](../concept/index.md)) and this docs site.
The service itself is blocked on prerequisite changes in `eliferpg-core`
(see [Ownership Map](../concept/ownership-map.md)); implementing it now
would be premature against an undesigned dependency.

Planned sequencing, each step its own brainstorming/spec/plan cycle in the
relevant repo:

1. `eliferpg-core`: NPC-owned Accounts — smallest, least dependent on
   anything else, unblocks everything.
2. `eliferpg-core`: Shop position/region, per-Character Inventory, and
   `GameServer` time/tick fields — can proceed in parallel with each
   other.
3. `eliferpg-core`: skill system — the big remaining unknown; its contract
   shape determines the trigger mechanism described in
   [Scheduler & Boundaries](../concept/scheduler.md).
4. `npc-virtual-simulation`: repo scaffolding, boundary config, tick
   engine, and admin API — can start once NPC-owned Accounts and
   `GameServer` time/tick fields exist, but the scheduler's actual trigger
   call can't be finalized until step 3 lands.

## Technology choices (planned)

- **.NET (C#), ASP.NET Core Minimal APIs** — matches `eliferpg-core`'s
  conventions.
- **PostgreSQL + EF Core**, not Marten — the remaining domain (boundary
  config, tick logs, online-player-count cache) is small, mutable
  current-state data; event-sourcing ceremony isn't warranted for it.
- **`ELifeRPG.BackendApiClient`** NuGet package (from
  [`ELifeRPG/backend-api-client-dotnet`](https://github.com/ELifeRPG/backend-api-client-dotnet))
  — a shared Kiota-generated C# client for the Central API, published to
  GitHub Packages and auto-regenerated/versioned from `eliferpg-core`'s
  OpenAPI spec. This repo consumes it as a package reference like
  `eliferpg-reforger-bridge` does, rather than generating its own client.
- Single deployable service, not a modulith — one bounded context (the
  scheduler), no need for per-module project splitting.
