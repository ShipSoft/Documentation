# Running

For a series of use cases (eg. updating `.jsonnet` files), it is not needed to build Aegir. Instead, if you have your local version of Aegir, you can simply execute the following commands

```
export WORK_DIR=/cvmfs/ship-nightlies.cern.ch/next/sw/
source $WORK_DIR/slc9_x86-64/aegir/latest/etc/profile.d/init.sh
phlex -c workflows/fixed_target_mt.jsonnet
```

If you need to change the source code, remember to build Aegir.

Information regarding the simulation to generate is located in the `.jsonnet` files. A few examples are in the `workflow` folder.

You can choose to use `geometry_geomodel_provider` or `geometry_gdml_provider` for the geometry: 

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
