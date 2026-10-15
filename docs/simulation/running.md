# Running

If you have an aegir clone built with pixi, run any workflow with:

```bash
pixi run phlex -c workflows/gun_st.jsonnet
```

Or drop into a pixi shell first for an interactive session:

```bash
pixi shell
phlex -c workflows/gun_st.jsonnet
```

`activate.sh` sets `PHLEX_PLUGIN_PATH`, `LD_LIBRARY_PATH`,
`SHIPGEOMETRY_ROOT`, `SHIPFIELD_ROOT` and `AEGIR_ROOT` automatically in both
modes, so you don't need to set a prefix by hand.

If you do not need to modify the source, you can also consume the
pre-built `aegir` package from the
[prefix.dev/ship](https://prefix.dev/channels/ship) channel. See
[Installation](installation.md) for that flow.

## Workflows

Information regarding the simulation to generate is located in the
`.jsonnet` files. The `workflows/` folder has examples for the particle gun,
Pythia8, fixed-target and file-input setups; `ls workflows/` shows them all.
Most of them build on the shared blocks in `workflows/lib.libsonnet`.

Phlex requires every job to name its processing stage. The aegir workflows
set `stage: 'simulation'` at the top level, and each generator source needs
the same `stage` in its own block, because Phlex does not pass the top-level
value on to sources. The `lib.libsonnet` blocks already include it; set both
if you write a workflow from scratch.

Generators and Geant4 draw a random seed for each run unless you give one.
The seed is logged at startup, so a run can still be reproduced. Workflows
that take a `seed` argument, such as `fixed_target_mt.jsonnet`, accept it on
the command line:

```bash
pixi run phlex -c workflows/fixed_target_mt.jsonnet --tla-code seed=12345
```

## Geometry

Choose the geometry provider in the `geometry` block of the workflow.

The built-in test geometry (a tungsten target with silicon scoring planes)
needs no setup. `gun_st.jsonnet`, `pythia8_mt.jsonnet` and most of the gun
workflows use it:

```jsonnet
geometry: { cpp: 'geometry_builtin_provider' },
```

A GeoModel database comes from the [Geometry](../geometry/index.md) package.
The `shipgeometry` package ships a `build_geometry` tool but no pre-built
database, so generate one before running a GeoModel workflow such as
`fixed_target_mt.jsonnet`:

```bash
pixi run build_geometry          # writes ship_geometry.db to the current directory
pixi run phlex -c workflows/fixed_target_mt.jsonnet
```

```jsonnet
geometry: {
      cpp: 'geometry_geomodel_provider',
      db_file: 'ship_geometry.db',
      sensitive_volumes: ['ScoringPlane'],
    }
```

The provider looks for a bare `db_file` name in the current directory first,
then in `$SHIPGEOMETRY_ROOT/share/geometry/`.

A GDML file needs an absolute path in `gdml_file` (see
`workflows/gun_st_gdml.jsonnet`):

```jsonnet
geometry: {
      cpp: 'geometry_gdml_provider',
      gdml_file: '/path/to/ship_target_only.gdml',
      sensitive_volumes: ['ScoringPlane'],
    }
```
