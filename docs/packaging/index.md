# Packaging

SHiP distributes its software stack as conda packages on the
[`prefix.dev/ship-experiment/stable`](https://prefix.dev/channels/@ship-experiment/stable) channel. Packages
that aren't on conda-forge are built from recipes in
[`ShipSoft/ship-conda-recipes`](https://github.com/ShipSoft/ship-conda-recipes);
everything else is pulled directly from
[conda-forge](https://conda-forge.org). The
[package table](https://github.com/ShipSoft/ship-conda-recipes#packages)
in the repository README lists every recipe, plus the packages that
have since moved to conda-forge.

Every active SHiP pixi repository (FairShip, aegir, aegir-genie,
data-model, Geometry, geometry_service, field_service, Shannon, Trout,
sea-cucumber, ship-random, ship-ci-metrics) consumes the channel via
its `pixi.toml`:

```toml
channels = ["https://prefix.dev/ship-experiment/stable", "conda-forge"]
```

## Flow

```text
upstream sources (GitHub / GitLab CERN tags)
        │
        ▼
ship-conda-recipes/recipes/<pkg>/recipe.yaml
        │   rattler-build build --recipe-dir recipes/
        ▼
ship-conda-recipes/output/{linux-64,linux-aarch64,noarch}/*.conda
        │   rattler-build upload prefix -c ship-experiment/stable
        ▼
prefix.dev/ship-experiment/stable channel
        │
        ▼
downstream repo's pixi.toml  →  developer / CI environment
```

## Platforms

Every recipe is built for linux-64. The Amaru stack is also built for
linux-aarch64: shipdatamodel, shipgeometry, shipgeometryservice,
field-service, mp-units and the GENIE chain (genie, pythia6,
rootegpythia6). osx-arm64 is planned.

- aegir, aegir-genie, shannon and trout skip linux-aarch64 for now.
  Their current releases need `shipdatamodel <0.6`, which has no
  linux-aarch64 build. Each recipe drops the skip when it moves to a
  release that supports data-model 0.6.
- The FairShip stack (fairship, fairroot, geant3, genfit, photospp,
  acts-ship) and npdb-client stay linux-64 only.
- noarch packages are built once and uploaded from the linux-64 job.

See the
[ship-conda-recipes README](https://github.com/ShipSoft/ship-conda-recipes/blob/main/README.md#platforms)
for the current list.

## Maintainer tasks

- [Add a new recipe](add-a-recipe.md)
- [Bump a package version](bump-a-version.md)
- [CI workflows and secrets](ci-and-secrets.md)

For building *unreleased* package versions during development (source
dependencies, local channels), see the
[Developer Guide](../dev-guide/index.md). Releases themselves always
flow through the recipes here.

## ROOT C++ standard variants

conda-forge splits ROOT builds by C++ standard, so most ROOT-linking
recipes here build two variants. Two keys in the recipe's
`variants.yaml` control this:

- `root_cxx_standard` (`"20"` or `"23"`) selects the matching
  `root_base` build in `host:` and is part of the package hash. The
  cxx20 variant serves consumers of the default ROOT builds, such as
  FairShip. The cxx23 variant serves the phlex/aegir stack.
- `root_base` is pinned explicitly, to the same version in every
  `variants.yaml` under `recipes/`. Without the pin, the solver can
  fall back to a legacy ROOT build from before the C++ standard split.

The `variant-lockstep` prek hook (`ci/check-variant-lockstep.py`)
checks that all `variants.yaml` files under `recipes/` carry the
same `root_base` pin. The development recipes in `recipes-dev/` are
not checked; `recipes-dev/phlex` pins `root_base` to `"6.40"`.
When conda-forge moves ROOT past the pin, the weekly
[channel-drift job](ci-and-secrets.md#channel-driftyml-weekly-drift-check)
opens a `variant-drift` PR that advances it everywhere at once.

The C++23-only recipes (aegir, aegir-genie) build a single variant and
constrain ROOT directly in `host:`. The
[README](https://github.com/ShipSoft/ship-conda-recipes#recipe-conventions)
has the full rationale and the upstream workarounds shared by several
recipes.

## Channel snapshot

A weekly job in `ship-conda-recipes` writes the current channel
contents to
[`channel-snapshot.md`](https://github.com/ShipSoft/ship-conda-recipes/blob/main/channel-snapshot.md).
The diff between snapshots doubles as a changelog and surfaces
accidental publishes.

## Scope

`ship-conda-recipes` hosts only packages that aren't (yet) on
conda-forge. When an upstream candidate lands on conda-forge, the
local recipe is dropped. See for example
[commit e0b730cf6](https://github.com/ShipSoft/ship-conda-recipes/commit/e0b730cf6)
for the `faircmakemodules` migration.

Two dependencies are built from ShipSoft forks:
[`pythia6`](https://github.com/ShipSoft/pythia6) (Pythia 6.4.28 with
the PDFLIB dummy routines restored) and
[`ROOTEGPythia6`](https://github.com/ShipSoft/ROOTEGPythia6), a
standalone copy of ROOT's EGPythia6 library, which ROOT removed in
6.30.
