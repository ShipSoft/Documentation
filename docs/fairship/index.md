# FairShip

FairShip is the simulation and reconstruction framework for the SHiP experiment.

It is based on:

- [FairRoot](https://fairroot.gsi.de/)
- [ROOT](https://root.cern/)
- [Geant4](https://geant4.web.cern.ch/)
- [Pythia8](https://pythia.org/)

Repository: [ShipSoft/FairShip](https://github.com/ShipSoft/FairShip)

API reference: [Doxygen](https://shipsoft.github.io/FairShip/)

## Quick start

```bash
git clone https://github.com/ShipSoft/FairShip.git
cd FairShip
pixi run build
pixi run python macro/run_simScript.py --tag my-simulation
```

See [Installation](installation.md) for the full details, including the
legacy CVMFS + aliBuild path for releases ≤26.04.
