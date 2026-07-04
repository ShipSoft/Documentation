# Developer Guide

How to develop across several SHiP packages at once: build unreleased
versions of dependencies (e.g. hack on `data-model` and `aegir`
together) and test upstream PRs (e.g. a [phlex](https://github.com/Framework-R-D/phlex)
PR) underneath our stack.

Every repo works out of the box in **binary mode**: `pixi install`
resolves released packages from
[`prefix.dev/ship`](https://prefix.dev/channels/ship) and conda-forge,
and you only build the repo you're in. That's the default, and it's
what CI uses. This guide is about the opt-in **source mode** on top of
it, built on pixi's [source
dependencies](https://pixi.prefix.dev/latest/build/getting_started/)
(the `pixi-build` preview feature) and on a local conda channel for
the cases source dependencies can't cover yet.

## Repository layout convention

All workflows here assume sibling checkouts under a common parent, plus
one shared directory for locally built packages:

```
SHiP/
├── aegir/
├── data-model/
├── geometry/
├── geometry_service/
├── field_service/
├── ship-conda-recipes/
└── ship-local-channel/     # created on demand by the build tasks
```

Relative paths in the opt-in dev blocks (`../data-model`,
`../ship-local-channel`) resolve against this layout.

## The two mechanisms

**Source dependencies** — a consumer's `dev` environment declares e.g.
`shipdatamodel = { path = "../data-model" }`; pixi builds the checkout
into a conda package on the fly (using the `[package]` section that
repo declares) and rebuilds it automatically whenever its sources
change. Used for `data-model`, `geometry` and `geometry_service`. See
[Multi-package development](multi-package.md).

**Local channel** — packages whose build can't go through a source
dependency yet (`field-service` because of its multi-output recipe,
`phlex` because it's upstream) are built into the
`../ship-local-channel` file channel, which dev environments resolve
ahead of the remote channels. See
[Testing dependency PRs](testing-dependency-prs.md).

## Requirements and caveats

- pixi ≥ 0.68 (`requires-pixi` in the manifests enforces this).
- `pixi-build` is a **preview feature**: behaviour can change between
  pixi releases. Known limitations and workarounds are tracked in the
  [pixi-build gap log](pixi-build-gaps.md).
- The opt-in blocks are commented out in the repos' `pixi.toml` and
  must stay that way in commits: pixi validates *every* environment
  against the lockfile, so an active `dev` environment would break
  `--locked` runs (CI, other developers) without your sibling
  checkouts. Never commit the uncommented block or the `pixi.lock`
  changes it causes.

## Relationship to the [Packaging](../packaging/index.md) section

Releases still flow exclusively through
[`ship-conda-recipes`](https://github.com/ShipSoft/ship-conda-recipes)
and the `prefix.dev/ship` channel — nothing in this guide publishes
anything. Two files are deliberate *dev twins* of release recipes and
must be kept in lockstep when [bumping a
version](../packaging/bump-a-version.md):

| Dev twin | Release counterpart |
| --- | --- |
| `field_service/recipe/recipe.yaml` | `ship-conda-recipes/recipes/field-service/recipe.yaml` |
| `ship-conda-recipes/recipes-dev/phlex/` | conda-forge [`phlex-feedstock`](https://github.com/conda-forge/phlex-feedstock) |

Likewise, each producer repo's `[package]` section mirrors its release
recipe (version and host dependencies).
