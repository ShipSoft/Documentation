# Data Model

Event data model for the SHiP experiment: plain C++23 structs with ROOT
dictionaries, stored with RNTuple. Every stage, from simulation through
reconstruction to the event display, reads and writes these classes.

Repository: [ShipSoft/data-model](https://github.com/ShipSoft/data-model)

API reference: [Doxygen](https://shipsoft.github.io/data-model/)

## Classes

| Header | Class | Category |
|--------|-------|----------|
| `SHiP/EventHeader.hpp` | `SHiP::EventHeader` | Event metadata |
| `SHiP/MCParticle.hpp` | `SHiP::MCParticle` | MC generation |
| `SHiP/SimHit.hpp` | `SHiP::SimHit` | Simulation |
| `SHiP/SimParticle.hpp` | `SHiP::SimParticle` | Simulation |
| `SHiP/SimResult.hpp` | `SHiP::SimResult` | Simulation |
| `SHiP/RecHit.hpp` | `SHiP::RecHit` | Reconstruction |
| `SHiP/RecParticle.hpp` | `SHiP::RecParticle` | Reconstruction |
| `SHiP/TrackFitResult.hpp` | `SHiP::TrackFitResult` | Reconstruction |

`SHiP/Units.hpp` holds the unit definitions, and `SHiP/detectors/` the
per-detector hit types.

## Using it

The conda package is `shipdatamodel` on
[prefix.dev/ship-experiment/stable](https://prefix.dev/channels/@ship-experiment/stable). In CMake:

```cmake
find_package(SHiPDataModel REQUIRED)
target_link_libraries(your_target PRIVATE SHiP::DataModel)
```

## Backward compatibility

Files written by released versions must stay readable. CI reads a frozen
reference file for every release with the current code, and a committed
schema snapshot makes any schema change fail CI until the snapshot is
regenerated in the same pull request. See
[`tests/data/README.md`](https://github.com/ShipSoft/data-model/blob/main/tests/data/README.md)
for the policy and what to do when these tests fail.
