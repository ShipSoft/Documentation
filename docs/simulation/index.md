# Aegir

Aegir is the simulation framework for the SHiP experiment.

It is built on [Phlex](https://github.com/Framework-R-D/phlex), [ROOT](https://root.cern/),
[Geant4](https://geant4.web.cern.ch/) and [Pythia8](https://pythia.org/).

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

## Requirements

pixi installs everything for you. For reference, aegir 0.4 needs:

- [Phlex](https://github.com/Framework-R-D/phlex) 0.4.1 or later
- [ROOT](https://root.cern/) 6.40 or later, built for C++23
- [Geant4](https://geant4.web.cern.ch/) 11.3 or later, with multi-threading
- [Pythia8](https://pythia.org/) 8.312 or later
- the SHiP packages [data-model](../data-model/index.md),
  [Geometry Service](../geometry_service/index.md),
  [Field Service](../field_service/index.md) and
  [ship-random](https://github.com/ShipSoft/ship-random)

## Features

The following capabilities are already implemented in aegir or provided through
the [Geometry Service](../geometry_service/index.md) and
[Field Service](../field_service/index.md):

**Event generators**

- Configurable particle gun with Philox / Random123 RNG (`particle_gun_source`)
- Pythia8 fixed-target p–p generation, serial or `PythiaParallel` (`pythia8_source`)
- Dual-target generation with p–p / p–n interaction-point sampling based on the
  material Z/A (`fixed_target_source`)
- Pre-generated events read back from an RNTuple file (`file_source`), the
  equivalent of FairShip's MuonBackGenerator. It accepts `MCParticle` and
  `SimParticle` input, so aegir output can be fed into a new job.
- GENIE rootracker files read without linking GENIE (`genie_reader_source`)
- GENIE neutrino interactions generated in-process by the separate
  [aegir-genie](genie.md) plugin (`genie_source`)
- Every source publishes an `EventHeader` (event weight and provenance) next
  to its particles, and can record the full generator history including all
  mothers

**Geometry** (selectable per workflow)

- Built-in test geometry: W target + Si scoring planes (`geometry_builtin_provider`)
- GDML file geometry (`geometry_gdml_provider`)
- GeoModel `.db` geometry via the Geometry Service (`geometry_geomodel_provider`)
- Configurable sensitive / scoring volumes

**Magnetic field** (via the Field Service)

- Field maps in the SHiP field-map format (RNTuple), configured per magnet
  from the workflow (`field_map_provider`)
- Field-off / null provider (`field_null_provider`)

**Transport & simulation**

- Geant4 transport, single- and multi-threaded with configurable concurrency (`geant4_module`)
- Energy-cut transport and scoring-plane hit saving
- Built-in profiling: Chrome tracing, flamegraphs and benchmarks

**Output**

- RNTuple output with parallel writing (`sim_output_module`): generator
  particles (`mc_particles`) and, for full simulation, hits (`sim_hits`) and
  transported particles (`sim_particles`)
- Validation histograms (ROOT 7)

**Workflows**

- Ready-to-run Phlex jsonnet workflows for gun, Pythia8, fixed-target and
  file-input setups in single-threaded / multi-threaded / generator-only
  variants (`workflows/`)

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

## GENIE neutrino generation

Neutrino interactions come from the separate [aegir-genie](https://github.com/ShipSoft/aegir-genie)
plugin. See [GENIE neutrino generation](genie.md).
