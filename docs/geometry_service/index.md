# Geometry Service

Framework-agnostic C++20 library that wraps the SHiP GeoModel geometry behind
a stable interface. It builds the GeoModel tree from the
[Geometry](../geometry/index.md) sources or loads it from a `.db` file, and
converts it to Geant4 on demand.

Repository: [ShipSoft/geometry_service](https://github.com/ShipSoft/geometry_service)

API reference: [Doxygen](https://shipsoft.github.io/geometry_service/)

## API

```cpp
#include <GeometryService/SHiPGeometryService.h>

auto svc = ship::SHiPGeometryService::fromSource();              // build from SHiPGeometry
// or: ship::SHiPGeometryService::fromFile("ship_geometry.db");
const GeoVPhysVol* world = svc->geoModelWorld();
```

The GeoModel tree is built eagerly, so geometry errors show up at start-up.
The Geant4 conversion is lazy: call `geant4WorldLogical()` from
`G4VUserDetectorConstruction::Construct()`, because it needs a
`G4RunManager` on the calling thread. After that,
`getLogicalVolume(name)` looks up Geant4 logical volumes by name.

## Building

```bash
git clone https://github.com/ShipSoft/geometry_service.git
cd geometry_service
pixi run test
```

## Using it

The conda package is `shipgeometryservice` on
[prefix.dev/ship](https://prefix.dev/channels/ship). In CMake:

```cmake
find_package(SHiPGeometryService CONFIG REQUIRED)
target_link_libraries(myapp PRIVATE SHiPGeometryService::SHiPGeometryService)
```
