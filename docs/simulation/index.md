# Aegir

Aegir is the simulation framework for the SHiP experiment.

It is based on:

- [Phlex](https://github.com/Framework-R-D/phlex)
- [ROOT](https://root.cern/)
- [Geant4](https://geant4.web.cern.ch/)
- [Pythia8](https://pythia.org/)

Repository: [ShipSoft/aegir](https://github.com/ShipSoft/aegir)

API reference: [Doxygen](https://shipsoft.github.io/aegir/)

## Quick start

```bash
git clone https://github.com/ShipSoft/aegir.git
cd aegir
pixi run build
pixi run phlex -c workflows/pythia8_mt.jsonnet
```

See [Installation](installation.md) for details.

## Three steps to target simulation
1. Collide proton beam at 400 GeV on fixed protons and neutrons using Pythia8
    - Beam travels in the z direction only, but smearing and painting are applied
    - Random choice for beam to interact with protons or neutrons based on the Z/A of the material it traverses
2. Transport particles with GEANT4 (and decay them, make them interact with matter, etc.)
    - Interactions with matter is time-consuming bit of target simulation
    -  To save time, particles below a certain energy are not transported
3. Save particles which reach a sensitive plane right after the hadron absorber (HA)
    -  Only particle above a certain energy are saved

<!-- -->

This simulation contains only target complex and hadron absorber. The magnetic field is also turned off.

Once the target simulation is produced, particles (such as muons) can be further propagated through the detector. This is much less time consuming, which is why there is a separation between "target simulation" and the rest of the simulation.
