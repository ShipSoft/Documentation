# Installation

## Using pixi (recommended)

[Pixi](https://pixi.sh) manages all of FairShip's dependencies via the
`pixi.toml` and `activate.sh` shipped in the FairShip clone. The
activation script sets `FAIRSHIP`, `GEOMPATH`, and similar paths to the
clone, so the pixi project root **must** be the FairShip clone itself.

1. [Install pixi](https://pixi.sh/latest/#installation).
2. Clone and build:
    ```bash
    git lfs install
    git clone https://github.com/ShipSoft/FairShip.git
    cd FairShip
    pixi run build
    ```
    The field maps in `files/*.root` are stored with Git LFS. Run
    `git lfs install` once per machine before cloning, otherwise you get
    small pointer files instead of the maps.
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
[FairShip README](https://github.com/ShipSoft/FairShip#using-the-pre-built-package)
for that flow.

### On lxplus (AFS)

A plain `pixi run build` installs the whole environment into `.pixi/`
inside your clone, which is enough to exceed a typical AFS quota. On
lxplus, use the pixi build published on CVMFS together with the
`pixi cvmfs` extension. This keeps the environment out of your AFS area.

Start from a clean shell and a FairShip clone without an old `build/`
directory:

```bash
export PATH=/cvmfs/ship-nightlies.cern.ch/pixi/0.72.0-rattlerfs-layers-d5a7568/bin:/cvmfs/ship-nightlies.cern.ch/pixi/pixi-cvmfs:$PATH
cd /path/to/FairShip
pixi cvmfs build    # build (once, and again after code changes)
pixi cvmfs          # enter the environment
```

To build a branch, check it out and run `pixi cvmfs build` again. You can
put the `export PATH=...` line in your `~/.bashrc` to have the commands
in every new shell. The pixi version in that path changes from time to
time; the [FairShip README](https://github.com/ShipSoft/FairShip#on-lxplus-afs)
has the current one.

## Legacy releases (CVMFS + aliBuild)

aliBuild on top of a CVMFS toolchain was the default up to and including
release **26.05**. From **26.06** onwards pixi is the recommended build
path. Newer CVMFS releases are still published (the latest is 26.10, with
FairShip 26.09), but they are no longer the main way to get FairShip.
The releases and their contents are listed in the
[shipdist changelog](https://github.com/ShipSoft/shipdist/blob/main/CHANGELOG.md).

Use these instructions if you work with an existing CVMFS release or with
the legacy branches (`SHiP-2018`, `muflux`). New development uses pixi;
see [Using pixi](#using-pixi-recommended) above and the
[shipdist page](../shipdist/index.md).

With CVMFS (e.g. lxplus):

```bash
source /cvmfs/ship.cern.ch/$SHIP_RELEASE/setUp.sh
aliBuild build FairShip --always-prefer-system --config-dir $SHIPDIST --defaults release
alienv enter FairShip/latest
```

In non-interactive contexts such as batch jobs, load the environment with
`eval $(alienv load FairShip/latest --no-refresh)` instead of
`alienv enter`.

Without CVMFS, install aliBuild via `pipx`/`pip`, clone
[shipdist](https://github.com/ShipSoft/shipdist) alongside FairShip, and
run `aliBuild build FairShip --config-dir ./shipdist --defaults release`.

shipdist currently provides two sets of defaults: `release` and `lcg`
(system packages from the LCG_109 view). The `fairship-2018` defaults
mentioned in older instructions for the legacy branches are not in the
current shipdist; ask on the software channel if you need to build those
branches.
