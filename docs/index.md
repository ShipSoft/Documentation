# SHiP Software

Welcome to the documentation for the [SHiP experiment](https://ship.web.cern.ch/) software.

## First Steps:

If you're only joining SHiP and/or CERN, please start from **[Onboarding](first-steps/index.md)** page.

## Projects

- **[FairShip](fairship/index.md)** — Simulation and reconstruction framework
- **[Simulation](simulation/index.md)** — aegir, the Phlex-based simulation framework, and its GENIE plugin aegir-genie
- **[Shannon](shannon/index.md)** — Digitisation
- **[Trout](trout/index.md)** — Reconstruction
- **[sea-cucumber](sea-cucumber/index.md)** — Event display
- **[Geometry](geometry/index.md)** — Detector geometry description
- **[Geometry Service](geometry_service/index.md)** — Geometry access library
- **[Field Service](field_service/index.md)** — Magnetic field map library
- **[Data Model](data-model/index.md)** — Event data classes
- **[ship-random](ship-random/index.md)** — Shared random number generator
- **[shipdist](shipdist/index.md)** — aliBuild recipes (CVMFS releases); new development uses [ship-conda-recipes](https://github.com/ShipSoft/ship-conda-recipes) and pixi
- **[cvmfs_release](cvmfs-release/index.md)** — CVMFS release management (archived)
- **[HTCondor](htcondor/index.md)** — Job submission scripts for the HTCondor batch system

## Contributing

- **[Developer Guide](dev-guide/index.md)** — pixi workflows, linting, working across several packages
- **[Packaging](packaging/index.md)** — Conda recipes and the prefix.dev/ship channel
