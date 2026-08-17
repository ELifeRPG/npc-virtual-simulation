---
icon: lucide/table
---

# Ownership Map

| Concern | Owned by |
|---|---|
| NPC identity (Account/Character, no Bohemia ID) | `eliferpg-core` |
| What an NPC can produce/consume (skills, recipes) | `eliferpg-core` — **future skill system, not yet designed** |
| NPC's shop location / "region" | `eliferpg-core` (likely a position field on `Shop`) — **exact mechanism not yet decided** |
| NPC's held-goods inventory | `eliferpg-core` — new per-Character Inventory concept |
| Money, banking, listings, purchases | `eliferpg-core` (existing Banking/Shops, unchanged) |
| How many NPCs are active, how fast, scaled by online-player count | `npc-virtual-simulation` |
| Rate-limit / max-population / activity-scaling config | `npc-virtual-simulation` |
| Tick scheduling and simulation control (pause/resume/manual tick) | `npc-virtual-simulation` |

## `eliferpg-core` prerequisites

Each of the following is substantial enough to deserve its own design pass
in `eliferpg-core` (following that repo's `docs/superpowers/specs/` habit)
before implementation. They're listed here so the dependency is explicit,
not designed in full:

1. **NPC-owned Accounts.** `Account` gets an `OwnerKind` (`Player` |
   `Npc`) discriminator. An NPC-owned account has no Bohemia ID and no
   Keycloak user — provisioned directly by a new endpoint (e.g.
   `POST /api/accounts/npc`), authorized by a new `npc:write` scope.
   Downstream modules (`Characters`, `Banking`, `Shops`) already only key
   off `AccountId`/`CharacterId`, so they need no structural change — only
   their existing scope checks (currently `gameserver:*`) need to also
   accept `npc:read`/`npc:write` on the specific endpoints an NPC needs
   (create character, open bank account, create shop, create/update
   listing, purchase listing). A new Keycloak client (e.g.
   `npc-simulation-dev`) gets those scopes, without the `impersonation`
   realm-management role — it's never needed, since there's no player to
   impersonate.
2. **Shop position / region.** Not yet designed. Most likely shape: a
   position/region field on `Shop`, applying to player- and NPC-owned
   shops alike, used to group NPC activity geographically without
   `npc-virtual-simulation` needing its own location concept.
3. **Per-Character inventory.** A real Inventory concept (`ItemId` →
   quantity) tied to a `Character`, so an NPC (and eventually players) has
   queryable held goods — crafted-but-unsold, bought-but-unconsumed.
4. **Skill system (future, explicitly deferred).** Determines what a given
   Character/NPC can produce and consume — supersedes any notion of a
   simulation-side "archetype" or "recipe" definition. `npc-virtual-
   simulation` depends on this existing before it can fully drive NPC
   production/consumption decisions.
5. **Time/tick settings on `GameServer`.** Extends the existing
   per-instance `GameServer` settings (`GET`/`PATCH
   /api/game-servers/{clientId}`) with whatever time/tick fields the
   simulation needs (e.g. in-game-time-per-real-time ratio) — exact fields
   TBD, but the mechanism (per-gameserver settings on the existing entity)
   already exists and is the right place for this, not a new concept.
