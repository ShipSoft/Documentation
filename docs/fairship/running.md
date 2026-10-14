# Running

All commands below assume the FairShip environment is active, either
through `pixi shell` (or `pixi run ...`) or, for a legacy CVMFS release,
through `alienv`. In both cases `$FAIRSHIP` points to the FairShip
installation. See [Installation](installation.md).

## Simulation, reconstruction and analysis

The basic chain simulates events, reconstructs them, and runs the example
analysis on the result:

```bash
python $FAIRSHIP/macro/run_simScript.py --tag my-simulation
python $FAIRSHIP/macro/ShipReco.py -f sim_my-simulation.root -g geo_my-simulation.root
python -i $FAIRSHIP/macro/ShipAna.py -f sim_my-simulation.root -r sim_my-simulation_rec.root -g geo_my-simulation.root
```

`run_simScript.py` writes `sim_<tag>.root`, `geo_<tag>.root` and
`params_<tag>.root`. Without `--tag`, the identifier is a generated UUID.
`ShipReco.py` writes the digitisation and reconstruction output to a
separate `sim_<tag>_rec.root` and leaves the simulation file unchanged.
`ShipAna.py` reads both files as friend trees, so it has access to the MC
truth and the reconstruction.

To look at the events, run the event display with the same three files:

```bash
python -i $FAIRSHIP/macro/eventDisplay.py -f sim_my-simulation.root -r sim_my-simulation_rec.root -g geo_my-simulation.root
```

### Reusing a geometry

On current `main` (not yet in release 26.09), `-g`/`--geofile` makes
`run_simScript.py` transport particles through the
geometry stored in a geometry file from an earlier simulation, for
example a standard geometry shared between productions:

```bash
python $FAIRSHIP/macro/run_simScript.py -g geo_my-simulation.root --tag my-second-simulation
```

The geometry configuration then comes from the file, so geometry options
such as `--vacuums`, `--SND` or `--target-yaml` cannot be given.

> [!WARNING]
> In 26.09 and earlier, `-g` set a muon shield geometry file that
> nothing read any more. Check old scripts that pass `-g`.

### Other useful options

- `-n` sets the number of events, `-s`/`--seed` the random seed and
  `-o` the output directory.
- `--validation` prints an extended validation summary after the run.
- `--reproducible` reduces nondeterministic log output, which helps when
  comparing runs.
- `--dry-run` stops after initialisation, `--check-overlaps` runs the
  geometry overlap check.
- `python $FAIRSHIP/macro/run_simScript.py --help` lists everything else,
  including the signal generators and particle gun options.

## Running fixed target simulation (`run_fixedTarget.py`)

To run the fixed target simulation with 100 PoTs and an energy cut of 5 GeV (only particles with an energy above that value will be propagated and saved), use the following command:

```bash
python $FAIRSHIP/macro/run_fixedTarget.py -n 100 --ecut 5
```

In general, you might want additional options. For example, the dimuon production cross-section can be increased, and so can the branching fraction of light particles to dimuons:

```bash
python $FAIRSHIP/macro/run_fixedTarget.py -n 100 --ecut 5 --boostDiMuon 100 --boostFactor 100
```

Other options you may need:

- `-r`/`--run-number` and `--seed` set the run number and random seed.
  Give each job of a production its own values.
- `--pythia8-tune FTFT` selects the FTFT tune for the primary
  interaction instead of the default (Monash 2013). This option is on
  `main` but not yet in release 26.09.
- `--charm` and `--beauty` generate charm and beauty decays.
- `--TARGET_YAML` selects the target configuration (see
  [Configuration](configuration.md)).

Run `python $FAIRSHIP/macro/run_fixedTarget.py --help` for the full list.

Note that the target simulation only propagates particles up to after the hadron absorber. This is because
1. The target simulation is the most time-consuming part of the simulation due to the number of interactions with matter
2. This allows us to change detectors downstream and test various configurations.
In addition, the field is turned off in the target simulation, and is added in the next steps of the simulation.

### Output

Unless you pass `-o`, the output goes to a new directory named
`<hostname>_run_fixedTarget_<run>` in the current directory. With
`--charm`, `--beauty` or a non-default `--pythia8-tune`, the directory
name gets `_charm`, `_beauty` or `_<tune>` appended. The events are in
`pythia8_evtgen_Geant4_<run>_<ecut>.root` (`pythia8_Geant4_...` if EvtGen
is not used). With the defaults (`-r 1`, `--ecut 0.5`) that is
`pythia8_evtgen_Geant4_1_0.5.root`.

## Running `run_simScript.py` after obtaining samples with `run_fixedTarget.py`
Once the target simulation is produced, particles (such as muons) can be further propagated through the detector.
Currently, we are using the production point of particles, rather than their hit position at the hadron absorber. This means that the lack of a magnetic field in the fixed target does not matter.
Depending on studies, this is where parts of the detector can be changed to understand effect on (muon) background.

Pass the output of `run_fixedTarget.py` with `-f` and select the muon
background generator with `--MuonBack`:

```bash
python $FAIRSHIP/macro/run_simScript.py --MuonBack -f <hostname>_run_fixedTarget_1/pythia8_evtgen_Geant4_1_5.0.root -n 100
```

`-f` can be repeated to read several input files. For files on EOS or
another remote storage, add `--remote-input` so the paths are passed to
ROOT as they are instead of being expanded as local globs. `--FollowMuon`
and `--FastMuon` are commonly used with `--MuonBack`; see `--help` for
what they do.

## Signal events with EventCalc

[EventCalc](https://github.com/ShipSoft/EventCalc) produces long-lived
particle decay events. Convert its output and simulate it with:

```bash
python $FAIRSHIP/macro/convertEvtCalc.py -f test_input.dat -o test_folder
python $FAIRSHIP/macro/run_simScript.py --evtcalc -n 1 -o test_folder -f test_folder/test_input.root
```
