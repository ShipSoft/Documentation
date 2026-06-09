# Running

If you have an aegir clone built with pixi, run any workflow with:

```bash
pixi run phlex -c workflows/fixed_target_mt.jsonnet
```

Or drop into a pixi shell first for an interactive session:

```bash
pixi shell
phlex -c workflows/fixed_target_mt.jsonnet
```

`activate.sh` sets `PHLEX_PLUGIN_PATH`, `LD_LIBRARY_PATH`,
`SHIPGEOMETRY_ROOT`, and `AEGIR_ROOT` automatically in both modes — no
manual prefix needed.

If you do not need to modify the source, you can also consume the
pre-built `aegir` package from the
[prefix.dev/ship](https://prefix.dev/channels/ship) channel — see
[Installation](installation.md) for that flow.

Information regarding the simulation to generate is located in the
`.jsonnet` files. A few examples are in the `workflows/` folder.

You can choose to use `geometry_geomodel_provider` or
`geometry_gdml_provider` for the geometry:

```
geometry: {
      cpp: 'geometry_geomodel_provider',
      db_file: 'ship_geometry.db',
      sensitive_volumes: ['ScoringPlane'],
    }
```

```
geometry: {
      cpp: 'geometry_gdml_provider',
      gdml_file: 'ship_target_only.gdml',
      sensitive_volumes: ['ScoringPlane'],
    }
```
