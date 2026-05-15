# Samples

Current samples are located on CERN EOS:

```
/eos/experiment/ship/simulation/bkg/
```

## Content of the different subfolders

- Mbias2025:
    - Description: Series of files generated at CERN in December 2025. Note that the parent of the particles cannot be accessed in these samples, due to a bug in FairShip 25.12
    - Geometry: default geometry (`$FAIRSHIP/geometry/target_config_Jun25.yaml`) version from FairShip 25.12 -- example file on `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2025/CERN_combined_weighted/geofile_full.root`
    - Stack: FairShip 25.12 (`/cvmfs/ship.cern.ch/25.12/setUp.sh`)
    - Number of events:
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2025/CERN_combined_weighted/33`: small production with energy cut at 1.0 GeV (0.526E8 PoTs)
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2025/CERN_combined_weighted/34`: small production with energy cut at 1.0 GeV (1.8E9 PoTs - 600k PoTs failed at submission), dimuon processes boosted by a factor of 100
- Mbias2026:
    - Description: Series of minimum bias files generated at CERN, GRIDPP and GENT in 2026
    - Geometry: default geometry (`$FAIRSHIP/geometry/target_config.yaml`) corresponding to "intermediate target" version from FairShip 26.03 and 26.04 (no change between the FairShip versions but information now saved in the FileSummary when `$FAIRSHIP/macro/run_fixedTarget.py` is run) -- example file on `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/CERN/merged/geofile_full.root` --
    - Stack: FairShip 26.03 and 26.04 (`/cvmfs/ship.cern.ch/26.03/setUp.sh` and `/cvmfs/ship.cern.ch/26.04/setUp.sh`)
    - Number of events: the number of events is growing as we speak, with currently several billion PoTs for all energy cuts combined. Files are present in the following folders
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/CERN/merged/ecut1GeV_dimuonboost100` (~1.8 billion PoTs)
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/CERN/merged/ecut5GeV_dimuonboost1` (~2.8 billion PoTs)
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/CERN/merged/ecut5GeV_dimuonboost100` (~2.8 billion PoTs)
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/GridPP/merged/ecut30GeV_dimuonboost1` (~12.0 billion PoTs)
        - `root://eospublic.cern.ch//eos/experiment/ship/simulation/bkg/Mbias2026/GridPP/merged/ecut70GeV_dimuonboost1` (~8.5 billion PoTs)
- MuonBack_2024helium:
- MuonDIS:
- MuonDIS_2024helium:
- NeutrinoDIS_2024helium:
- NeutrinoDIS_2024helium_noCavern:
- NeutrinoDIS_2024helium_noCavern_v2:
