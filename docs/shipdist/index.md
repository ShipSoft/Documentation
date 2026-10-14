# shipdist

`shipdist` holds the [aliBuild](https://alisw.github.io/alibuild/)
recipes for the SHiP software stack — the build system behind the
CVMFS releases under `/cvmfs/ship.cern.ch/`.

New development uses pixi and the conda-forge ecosystem instead:

- [FairShip — Using pixi](../fairship/installation.md#using-pixi-recommended)
- [ship-conda-recipes](https://github.com/ShipSoft/ship-conda-recipes)
  — rattler-build recipes for the SHiP-specific packages, published to
  the [prefix.dev/ship-experiment/stable](https://prefix.dev/channels/@ship-experiment/stable) channel.

shipdist is in maintenance mode, but it is still tagged and released.
Each tag (for example
[26.10](https://github.com/ShipSoft/shipdist/releases/tag/26.10), with
FairShip 26.09) corresponds to a CVMFS release, and the
[CHANGELOG](https://github.com/ShipSoft/shipdist/blob/main/CHANGELOG.md)
lists what changed in each one. Use it as the release notes for CVMFS
releases from 26.01 on.

Two sets of defaults are available: `release` (the usual one) and `lcg`
(system packages from the LCG_109 view).

For users on existing CVMFS releases, see
[FairShip → Legacy releases](../fairship/installation.md#legacy-releases-cvmfs-alibuild).

Repository: [ShipSoft/shipdist](https://github.com/ShipSoft/shipdist)
