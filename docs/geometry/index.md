# Geometry

The SHiP detector geometry, implemented in C++23 with
[GeoModel](https://geomodel.web.cern.ch/). The simulation, digitisation and
reconstruction packages all read it, either directly or through the
[Geometry Service](../geometry_service/index.md).

Repository: [ShipSoft/Geometry](https://github.com/ShipSoft/Geometry)

API reference: [Doxygen](https://shipsoft.github.io/Geometry/)

## Building the geometry database

```bash
git clone https://github.com/ShipSoft/Geometry.git
cd Geometry
pixi run test       # configure, build and run the unit tests
pixi run geometry   # write build/ship_geometry.db
pixi run gdml       # convert it to build/ship_geometry.gdml
```

Open the `.db` file in `gmex` to browse the geometry.

## Conventions

- Origin at the target front face (first tungsten disk).
- z along the beam (positive downstream), y vertical (up), x completing a
  right-handed system.
- Lengths in mm, angles in radians. The beam axis is 1.7 m above the floor.

Each subsystem is a factory class under `subsystems/<Name>/`, placed by
`SHiPGeometryBuilder::build()`. The repository README has the
[status table](https://github.com/ShipSoft/Geometry#implementation-status)
for each subsystem and the steps to add or modify one.

## Using it

The conda package is `shipgeometry` on
[prefix.dev/ship](https://prefix.dev/channels/ship). In CMake:

```cmake
find_package(SHiPGeometry CONFIG REQUIRED)
target_link_libraries(myapp PRIVATE SHiPGeometry::SHiPGeometry)
```
