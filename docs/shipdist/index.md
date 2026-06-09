# shipdist

`shipdist` holds the [aliBuild](https://alisw.github.io/alibuild/)
recipes for the SHiP software stack — the build system behind the
CVMFS releases under `/cvmfs/ship.cern.ch/`.

New development uses pixi and the conda-forge ecosystem instead:

- [FairShip — Using pixi](../fairship/index.md)
- [ship-conda-recipes](https://github.com/ShipSoft/ship-conda-recipes)
  — rattler-build recipes for the SHiP-specific packages, published to
  the [prefix.dev/ship](https://prefix.dev/channels/ship) channel.

For users on existing CVMFS releases, see
[FairShip → Legacy releases](../fairship/installation.md#legacy-releases-cvmfs--alibuild-2604).

Repository: [ShipSoft/shipdist](https://github.com/ShipSoft/shipdist)
