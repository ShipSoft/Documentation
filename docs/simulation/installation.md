# Installation

Aegir is built with [pixi](https://pixi.sh). Two use cases:

## Just running the framework

If you only need to run workflows (e.g. update `.jsonnet` files, drive
existing simulations), consume the pre-built `aegir` package from the
[prefix.dev/ship](https://prefix.dev/channels/ship) channel:

```bash
pixi init my-aegir-workspace
cd my-aegir-workspace
pixi project channel add https://prefix.dev/ship
pixi add aegir
pixi run phlex -c <path-to-workflow>.jsonnet
```

## Building from source

For changes to the C++ code, clone and build:

```bash
git clone https://github.com/ShipSoft/aegir.git
cd aegir
pixi run build
```

`pixi run build` depends on `configure`, so a fresh checkout becomes a
ready-to-run environment in one command. See the
[aegir README](https://github.com/ShipSoft/aegir#building-with-pixi-recommended)
for the full list of tasks (`test`, `smoke`, `clean`, …) and the
`pixi shell` vs `pixi run` workflow.
