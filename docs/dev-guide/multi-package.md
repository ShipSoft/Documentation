# Multi-package development

The worked example is hacking on `data-model` and `aegir` together;
the same recipe applies to `geometry` → `geometry_service`,
`geometry` → `aegir`, etc.

## One-time setup

1. Check out the repos as siblings (see the
   [layout convention](index.md#repository-layout-convention)).
2. In `aegir/pixi.toml`, uncomment the *Multi-package development*
   block: the `[feature.dev]` tables and the
   `dev = { features = ["dev"] }` line in `[environments]`. Trim the
   source-dependency list to the packages you actually want from
   source; everything else keeps coming from the released channel.

The uncommented manifest and the resulting `pixi.lock` changes are
**local state** — never commit them (pixi validates every environment
against the lock, so they'd break `--locked` runs for everyone
without your checkouts). When you're done:

```console
$ git checkout pixi.toml pixi.lock
```

## The loop

```console
$ vim ../data-model/include/SHiP/MCParticle.hpp   # hack
$ pixi run -e dev build                           # pixi rebuilds shipdatamodel, then aegir
$ pixi run -e dev test
$ pixi run -e dev smoke
```

All regular tasks work in the dev environment — they are the same
tasks, just run with `-e dev`. pixi notices changed sources
automatically: on the next `pixi run -e dev ...` it rebuilds the
source dependency's conda package, reinstalls it into the `dev`
environment, and your build picks it up through the usual
`find_package` machinery. Measured on `data-model` → `aegir`: an
edit costs ~20–30 s end to end (dep package rebuild + incremental
aegir rebuild); a no-op `pixi run -e dev build` returns in under a
second.

The default environment is untouched throughout: `pixi run build`
keeps using the released binary packages.

## How it works

Each producer repo declares a `[package]` section in its `pixi.toml`
(preview feature `pixi-build`), mirroring its release recipe:

```toml
[package]
name = "shipdatamodel"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-cmake", version = "0.*" }

[package.host-dependencies]
root_base = "*"
root_cxx_standard = "==23"
```

A consumer's `dev` feature then replaces the binary spec with a source
spec — the path form builds a local checkout, the git form builds a
remote ref (handy for [testing a colleague's
PR](testing-dependency-prs.md) without checking it out):

```toml
[feature.dev.dependencies]
shipdatamodel = { path = "../data-model" }
# or: shipdatamodel = { git = "https://github.com/ShipSoft/data-model", branch = "my-feature" }
```

## Rules

- **Version lockstep.** A source-built package carries the version
  from its `[package]` section. Binary packages pin each other via
  `run_exports` (e.g. `shipgeometryservice` requires
  `shipdatamodel 0.x`), and those pins must stay satisfiable when you
  mix a source package into a binary environment. So `[package].version`
  must always match the latest release — it's part of the
  [version-bump checklist](../packaging/bump-a-version.md).
- **ABI caution.** The binary packages in your dev environment were
  compiled against the *released* headers of the package you're now
  building from source. Breaking ABI in the source checkout (changing
  class layouts used across package boundaries) without rebuilding the
  affected consumers from source too will crash at runtime, not at
  configure time. When in doubt, add the downstream package to the
  source list as well.
- **field-service is different.** Its multi-output recipe can't be a
  source dependency yet (see the [gap log](pixi-build-gaps.md));
  co-develop it via the local channel instead:
  `pixi run build-local-channel` in `field_service/`, then
  `pixi update field-service-core field-service-g4` in the consumer.
