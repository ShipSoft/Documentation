# HTCondor submission scripts

Job submission scripts for running SHiP workloads on the HTCondor batch system.

Repository: [ShipSoft/htcondor_submission_scripts](https://github.com/ShipSoft/htcondor_submission_scripts)

The repository has one directory per workflow: `fixedTarget/`,
`MuonBack/`, `MuonDIS/`, `NeutrinoDIS/` and `SignalEventCalc/`. The
maintained production setup is
[`fixedTarget/batch/`](https://github.com/ShipSoft/htcondor_submission_scripts/tree/main/fixedTarget/batch).
The scripts in `MuonBack/`, `MuonDIS/`, `NeutrinoDIS/` and
`SignalEventCalc/` still source the CVMFS release 24.10, and the scripts
at the top of `fixedTarget/` point to paths outside CERN. Check the paths
and options before you use them.

## Quick start -- `run_fixedTarget.py` example with ganga
For a small test you need two files: `bashScript.sh` and `gangaScript.py`.

Your `bashScript.sh` lays out what you want to run. The snippet below
sources a legacy CVMFS aliBuild release (**26.03**, as in the upstream
scripts). Substitute the release you want, or point at your own FairShip
build instead.
```bash
FS_INSTALL=/cvmfs/ship.cern.ch/26.03/
export WORK_DIR=${FS_INSTALL}/sw/
source ${FS_INSTALL}/sw/slc9_x86-64/FairShip/latest/etc/profile.d/init.sh
python ${FS_INSTALL}/sw/slc9_x86-64/FairShip/latest/macro/run_fixedTarget.py $@
```

Once you have your `bashScript.sh` ready, you then need a `gangaScript.py`. Below, we run one job with 10 subjobs; the arguments for all subjobs are passed in the `j.application`, while the arguments per subjob are passed in the `j.splitter`.

Every subjob needs its own run number and seed, and so does every
submission. Otherwise jobs repeat the same events and write files with
the same names. The script picks a random starting run number and gives
each subjob the next one, as the upstream production script does.

The script also passes the run number as `--seed`, so the run number
fixes the random sequence. (`run_fixedTarget.py` does not derive the
seed from `-r` itself; without `--seed` it uses a time-based seed.) A
random start only makes collisions unlikely. When you submit several
jobs in parallel, give each one its own range of run numbers
(`startRun` to `startRun + nSJ - 1`) that does not overlap with the
others.

```python
import os
import random
import time

evtsPerJob = 1000
nSJ = 10

random.seed(os.environ.get('USER', '') + str(time.time()))
startRun = random.randint(0, 300000000 - nSJ)

j = Job(name = f'run fixed target - {nSJ * evtsPerJob} events')
j.application = Executable(exe = File('bashScript.sh'), args = ['-o', '"./"', '-n', evtsPerJob, '-e', '5'])
j.splitter = ArgSplitter(args = [['-r', startRun + _i, '--seed', startRun + _i] for _i in range(nSJ)], append = True)
j.outputfiles = [LocalFile('pythia8_evtgen_Geant4_*.root')]
j.backend = Condor()
j.backend.cdf_options['+MaxRuntime'] = '10000'

j.comment = f'{evtsPerJob} events in each of {nSJ} subjobs'
j.submit()
```

Note that it is also possible to change the output location of your files (eg. to eos) if needed.

Finally, to submit the jobs, run the following:
```bash
/cvmfs/ganga.cern.ch/Ganga/install/ship/bin/ganga gangaScript.py
```

To keep an eye on the status of your jobs, you can open ganga with `/cvmfs/ganga.cern.ch/Ganga/install/ship/bin/ganga`.

Documentation for ganga can be found [here](https://ganga.readthedocs.io/en/latest/).

## Production submission

Large productions use the scripts in `fixedTarget/batch/` instead. On top
of the quick start, they:

- run through `wn_script.py`, which takes the CVMFS release
  (`--cvmfs_version`) and the site (`--site`);
- set `SITE` in `gangaScript.py` (`CERN`, `GRIDPP` or `GENT`), which
  gives each site its own range of run numbers;
- copy the output to EOS with `MassStorageFile`;
- check the job output for `Macro finished successfully.` and register
  the files in Rucio with `postprocessor_master.py`.

Follow the steps in
[`fixedTarget/batch/README.md`](https://github.com/ShipSoft/htcondor_submission_scripts/blob/main/fixedTarget/batch/README.md).
You need a valid VOMS proxy for the file registration, and you must
submit with `gangaSubmit.sh`, which uses the `ship` ganga installation on
CVMFS rather than the latest one.

These scripts use a legacy CVMFS release. For HTCondor, the FairShip
README recommends a container image; see its
[Docker instructions](https://github.com/ShipSoft/FairShip#docker-instructions).
