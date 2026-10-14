# Field Service

Framework-agnostic C++20 library exposing the SHiP magnetic field maps to
simulation ([aegir](../simulation/index.md)) and reconstruction
([Trout](../trout/index.md)).

Repository: [ShipSoft/field_service](https://github.com/ShipSoft/field_service)

API reference: [Doxygen](https://shipsoft.github.io/field_service/)

## Field maps

Maps are stored in the
[SHiP field-map format](https://github.com/ShipSoft/field_service/blob/main/docs/field_map_format.md),
a versioned ROOT RNTuple layout, and evaluated with
[covfie](https://github.com/acts-project/covfie) (trilinear interpolation on
a regular grid). A bare map filename is resolved against
`$SHIPFIELD_ROOT/share/field/`.

## Components

| Component | Conda package | CMake target | Needs |
|-----------|---------------|--------------|-------|
| Core: evaluators from in-memory maps | `field-service-core` | `SHiP::FieldService` | nothing from ROOT or Geant4 |
| MapIO: read and write map files, `FieldMapSource` | `field-service-mapio` | `SHiP::FieldServiceMapIO` | ROOT |
| G4Adapter: `G4MagFieldAdapter` | `field-service-g4` | `SHiP::FieldServiceG4Adapter` | Geant4 |
| ActsAdapter: `ActsFieldAdapter` | `field-service-acts` | `SHiP::FieldServiceActsAdapter` | ACTS |

Request the components you need, e.g.
`find_package(SHiPFieldService REQUIRED COMPONENTS G4Adapter MapIO)` in aegir
or `COMPONENTS MapIO ActsAdapter` in Trout. All packages are on
[prefix.dev/ship](https://prefix.dev/channels/ship).

## Tools

The `field-service-tools` package ships:

- `fairship_to_fieldmap` converts FairShip's legacy ROOT field maps (use
  `--symmetry quadrant_dipole` for quadrant-symmetric maps);
- `fieldmap_dump` prints map metadata or evaluates a map at points read from
  stdin;
- `generate_constant_fieldmap` writes a uniform map for closure tests;
- `plot_field_map` plots a map as ROOT histograms and PDFs.

## Building

```bash
git clone https://github.com/ShipSoft/field_service.git
cd field_service
pixi run test
```
