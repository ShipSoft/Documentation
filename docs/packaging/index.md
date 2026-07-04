# Packaging

SHiP distributes its software stack as conda packages on the
[`prefix.dev/ship`](https://prefix.dev/channels/ship) channel. Packages
that aren't on conda-forge are built from recipes in
[`ShipSoft/ship-conda-recipes`](https://github.com/ShipSoft/ship-conda-recipes);
everything else is pulled directly from
[conda-forge](https://conda-forge.org).

Downstream repos (`FairShip`, `aegir`, `geometry`, `geometry_service`)
consume the channel via their `pixi.toml`:

```toml
channels = ["https://prefix.dev/ship", "conda-forge"]
```

## Flow

```
upstream sources (GitHub / GitLab CERN tags)
        │
        ▼
ship-conda-recipes/recipes/<pkg>/recipe.yaml
        │   rattler-build build --recipe-dir recipes/
        ▼
ship-conda-recipes/output/{linux-64,noarch}/*.conda
        │   rattler-build upload prefix -c ship
        ▼
prefix.dev/ship channel
        │
        ▼
downstream repo's pixi.toml  →  developer / CI environment
```

## Maintainer tasks

- [Add a new recipe](add-a-recipe.md)
- [Bump a package version](bump-a-version.md)
- [CI workflows and secrets](ci-and-secrets.md)

For building *unreleased* package versions during development (source
dependencies, local channels), see the
[Developer Guide](../dev-guide/index.md) — releases themselves always
flow through the recipes here.

## Channel snapshot

A weekly job in `ship-conda-recipes` writes the current channel
contents to
[`channel-snapshot.md`](https://github.com/ShipSoft/ship-conda-recipes/blob/main/channel-snapshot.md).
The diff between snapshots doubles as a changelog and surfaces
accidental publishes.

## Scope

`ship-conda-recipes` hosts only packages that aren't (yet) on
conda-forge. When an upstream candidate lands on conda-forge, the
local recipe is dropped — see for example
[ShipSoft/ship-conda-recipes#23](https://github.com/ShipSoft/ship-conda-recipes/commit/e0b730cf6)
for the `faircmakemodules` migration.
