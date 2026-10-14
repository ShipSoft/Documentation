# Developer Guide

How to develop across several SHiP packages at once: build unreleased
versions of dependencies (e.g. hack on `data-model` and `aegir`
together) and test upstream PRs (e.g. a [phlex](https://github.com/Framework-R-D/phlex)
PR) underneath our stack. For the shared CI, release and repository
setup see [CI & repository conventions](ci-and-conventions.md).

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

```text
SHiP/
├── aegir/
├── aegir-genie/
├── data-model/
├── geometry/               # ShipSoft/Geometry, cloned in lower case
├── geometry_service/
├── field_service/
├── Trout/
├── ship-conda-recipes/
└── ship-local-channel/     # created on demand by the build tasks
```

Relative paths in the opt-in dev blocks (`../data-model`,
`../geometry`, `../ship-local-channel`) resolve against this layout.
The repository is called `ShipSoft/Geometry`, but the dev blocks and
CI expect a lower-case `geometry/` directory, so give the clone an
explicit name:

```console
$ git clone https://github.com/ShipSoft/Geometry geometry
```

## The two mechanisms

**Source dependencies** — a consumer's `dev` environment declares e.g.
`shipdatamodel = { path = "../data-model" }`; pixi builds the checkout
into a conda package on the fly (using the `[package]` section that
repo declares) and rebuilds it automatically whenever its sources
change. The producers are `data-model`, `geometry` and
`geometry_service`. Consumers with a dev block are aegir (all three),
aegir-genie (`geometry_service`) and Trout (`data-model`). See
[Multi-package development](multi-package.md).

**Local channel** — packages whose build can't go through a source
dependency yet (`field-service` because of its multi-output recipe,
`phlex` because it's upstream) are built into the
`../ship-local-channel` file channel, which dev environments resolve
ahead of the remote channels. See
[Testing dependency PRs](testing-dependency-prs.md).

## Requirements and caveats

- pixi ≥ 0.68. The repositories that use `pixi-build` enforce this
  with `requires-pixi`; field_service, Shannon and FairShip don't set
  it.
- `pixi-build` is a **preview feature**: behaviour can change between
  pixi releases. Known limitations and workarounds are tracked in the
  [pixi-build gap log](pixi-build-gaps.md).
- The opt-in blocks are commented out in the repos' `pixi.toml` and
  must stay that way in commits: pixi validates *every* environment
  against the lockfile, so an active `dev` environment would break
  `--locked` runs (CI, other developers) without your sibling
  checkouts. Never commit the uncommented block or the `pixi.lock`
  changes it causes. Trout differs slightly: its `dev` environment is
  committed and active because it adds `gdb`, and only the
  `shipdatamodel` path line is commented out.

## CI coverage

Three scheduled workflows in `ship-conda-recipes` exercise this setup
every day, so a regression usually turns up there first:

| Workflow | Schedule (UTC) | What it builds |
| --- | --- | --- |
| `dev-build-fairship.yml` | daily 05:00 | FairShip `master`, lock deleted, resolved against the current channels |
| `dev-build-aegir.yml` | daily 06:00 | the same for aegir `main` |
| `dev-source-build-aegir.yml` | daily 07:00 | aegir's `dev` environment, with data-model, Geometry and geometry_service built as source dependencies from their `main` branches |

The source-build job pins pixi (currently 0.68.1) because it is the
tripwire for regressions in the `pixi-build` preview feature; regular
CI uses the latest pixi. All three workflows can also be started by
hand to try out a pull request, see
[Testing dependency PRs](testing-dependency-prs.md#in-ci-without-a-local-setup).

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
