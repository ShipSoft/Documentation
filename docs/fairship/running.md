# Running

## Running fixed target simulation (`run_fixedTarget.py`)
To run the fixed target simulation with 100 PoTs and an energy cut of 5 GeV (only particles with an energy above that value will be propagated and saved), use the following command:

`python FairShip/macro/run_fixedTarget.py -n 100 --ecut 5`

In general, you might want additional options. For example, the dimuon production cross-section can be increased, and so can the branching fraction of light particles to dimuons:

`python3 FairShip/macro/run_fixedTarget.py -n 100 --ecut 5 --boostDiMuon 100 --boostFactor 100`

Other options exist, but they are used less frequencly for now.

Note that the target simulation only propagates particles up to after the hadron absorber. This is because 
1. The target simulation is the most time-consuming part of the simulation due to the number of interactions with matter
2. This allows us to change detectors downstream and test various configurations.
In addition, the field is turned off in the target simulation, and is added in the next steps of the simulation.

## Running `run_simScript.py` after obtaining samples with `run_fixedTarget.py`
Once the target simulation is produced, particles (such as muons) can be further propagated through the detector. 
Currently, we are using the production point of particles, rather than their hit position at the hadron absorber. This means that the lack of a magnetic field in the fixed target does not matter.
Depending on studies, this is where parts of the detector can be changed to understand effect on (muon) background.

`python3 FairShip/macro/run_simScript.py -f file from_target_production -n 100 --MuonBack`
