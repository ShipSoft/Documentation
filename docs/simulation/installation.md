# Installation

Depending on use cases, you might or might not need to build Aegir. For use cases such as updating `.jsonnet` files, it is sufficient to source the latest `ship-nightlies`.

```
export WORK_DIR=/cvmfs/ship-nightlies.cern.ch/next/sw/
source $WORK_DIR/slc9_x86-64/aegir/latest/etc/profile.d/init.sh
```

For more involved use cases, such as changing the source code, you also need to build your local version of Aegir. Currently aliBuild is recommended:

```
source $WORK_DIR/slc9_x86-64/alibuild/latest/etc/profile.d/init.sh
aliBuild build aegir --always-prefer-system -c /cvmfs/ship-nightlies.cern.ch/next/shipdist --defaults release
```
