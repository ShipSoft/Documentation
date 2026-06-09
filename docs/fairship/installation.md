# Installation

## Using pixi (recommended)

[Pixi](https://pixi.sh) manages all of FairShip's dependencies via the
`pixi.toml` and `activate.sh` shipped in the FairShip clone. The
activation script sets `FAIRSHIP`, `GEOMPATH`, and similar paths to the
clone, so the pixi project root **must** be the FairShip clone itself.

1. [Install pixi](https://pixi.sh/latest/#installation).
2. Clone and build:
    ```bash
    git clone https://github.com/ShipSoft/FairShip.git
    cd FairShip
    pixi run build
    ```
3. Run commands inside the environment:
    ```bash
    pixi run python macro/run_simScript.py --tag my-simulation
    ```
   Or start a shell:
    ```bash
    pixi shell
    python macro/run_simScript.py --tag my-simulation
    ```

Pre-built FairShip packages are also published to the
[prefix.dev/ship](https://prefix.dev/channels/ship) channel; see the
[FairShip README](https://github.com/ShipSoft/FairShip#using-pixi) for
that flow.

## Legacy releases (CVMFS + aliBuild, ≤26.04)

These instructions are kept for users on existing CVMFS releases up to
**26.04** and for legacy branches (`SHiP-2018`, `muflux`). New
development uses pixi — see [Using pixi](#using-pixi-recommended)
above and the [shipdist page](../shipdist/index.md).

With CVMFS (e.g. lxplus):

```bash
source /cvmfs/ship.cern.ch/$SHIP_RELEASE/setUp.sh
aliBuild build FairShip --always-prefer-system --config-dir $SHIPDIST --defaults release
alienv enter FairShip/latest
```

Without CVMFS, install aliBuild via `pipx`/`pip`, clone
[shipdist](https://github.com/ShipSoft/shipdist) alongside FairShip, and
run `aliBuild build FairShip` with the same defaults.
