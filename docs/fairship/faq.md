# FAQ and common issues

## Which operating systems are supported?

Linux is supported:

- RHEL9 (on lxplus and HTCondor)
- Ubuntu LTS on a best-effort basis

Other distributions may be available upon sufficient interest.

## aliBuild not using system/CVMFS packages

Use the `--always-prefer-system` flag on platforms like RHEL9 where aliBuild does not automatically use system packages:

```bash
aliBuild build --always-prefer-system FairShip
```

## Git "unsafe directory" warning

Recent versions of Git may refuse to operate in directories owned by other users. If you see this warning, add the directory to the safe list:

```bash
git config --global --add safe.directory /cvmfs/ship.cern.ch/24.01/shipdist
```

This was resolved in aliBuild 1.17.3+.

## LHCb environment interference

The `lbenv` tool can break the SHiP software stack. Configure `lbenv` to prevent automatic loading. With the right configuration, both LHCb and SHiP software can coexist in the same account.
