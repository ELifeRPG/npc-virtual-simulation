---
icon: lucide/plug
---

# API Reference

!!! note "Not implemented yet"
    This service doesn't exist as running code yet — see
    [Development](../development/index.md) for status and blockers. The
    endpoints below are the planned admin/staff-facing surface, as agreed
    during design; they'll be filled in with real request/response shapes
    (and generated OpenAPI docs, matching `eliferpg-core`'s own `/docs`)
    once implementation starts.

`npc-virtual-simulation` exposes a real REST API of its own, consumed
directly by staff tooling/Admin UI — not by `eliferpg-core`. It is *also* a
Client-Credentials client of `eliferpg-core`'s Central API (`npc:read` /
`npc:write` scopes); that outbound direction is documented in
[Scheduler & Boundaries](../concept/scheduler.md), not here.

## Planned endpoints

| Resource | Purpose |
|---|---|
| `Boundaries` | Get/update max population, rate limits, activity-scaling curve parameters (likely scoped per gameserver instance) |
| `SimulationState` | Read-only, per gameserver instance: current online-player count, current activity-scaling factor, how many NPCs were active in recent ticks |
| `TickLog` | List/inspect recent tick history and errors |
| `SimulationControl` | Pause/resume, trigger a manual tick (per gameserver instance) |
| `/health` | Liveness/readiness |

## Auth

Staff/Admin UI tokens are validated the same way `eliferpg-core` validates
them — same Keycloak realm, JWT bearer, role/scope-based authorization.
This service is a resource server for staff *and* a Client-Credentials
client of the Central API simultaneously.
