# NPC Virtual Simulation

A backend service for the ELifeRPG ArmA Reforger mod that drives
non-player economic activity — NPCs that produce and consume goods through
the same Banking/Shops infrastructure real players use, so the in-game
market has supply and demand even when players don't.

**Status: concept stage.** No application code exists yet. See the docs
site for the full design and current status:

- [Concept](docs/concept/index.md)
- [Ownership Map](docs/concept/ownership-map.md)
- [Scheduler & Boundaries](docs/concept/scheduler.md)
- [API Reference](docs/api/index.md)
- [Development](docs/development/index.md)

This service is a Client-Credentials consumer of
[`eliferpg-core`](https://github.com/ELifeRPG/backend)'s Central API — it owns
none of the NPC domain data itself (identity, production capability,
inventory all live in `eliferpg-core`), only the pacing/scheduling logic
and its tunable boundaries. See [Concept](docs/concept/index.md) for why.
It talks to the Central API through the shared
[`ELifeRPG.BackendApiClient`](https://github.com/ELifeRPG/backend-api-client-dotnet)
NuGet package rather than generating its own client.

## Documentation site

This `README` and the in-repo docs are the source; the rendered site is
built with [Zensical](https://zensical.org/) and deployed to GitHub Pages
on push to `main` (see `.github/workflows/docs.yml`).

Requires Python `>=3.13` and [uv](https://docs.astral.sh/uv/):

```bash
uv sync
uv run zensical serve -f zensical.yml
```

A [devcontainer](https://containers.dev/) is provided (same pattern as
[`eliferpg-core`](https://github.com/ELifeRPG/backend) and
[`ELifeRPG/docs`](https://github.com/ELifeRPG/docs)) — VS Code: **Reopen in
Container**, or `devcontainer up --workspace-folder .` via the CLI.

For the shared, project-wide documentation site (architecture across all
ELifeRPG repos), see [ELifeRPG/docs](https://github.com/ELifeRPG/docs).

## License

Copyright © 2026 ELifeRPG contributors.

Licensed under the [GNU Affero General Public License v3.0 or later](LICENSE)
(`AGPL-3.0-or-later`). You may use, change and share this software. If you
distribute it, or run a modified version for other people over a network
(for example, a game server), you must make your complete source code
available to those users under the same license.

Contributions need a DCO sign-off; see [CONTRIBUTING.md](CONTRIBUTING.md).
