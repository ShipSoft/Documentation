# HTCondor submission scripts

Job submission scripts for running SHiP workloads on the HTCondor batch system.

Repository: [ShipSoft/htcondor_submission_scripts](https://github.com/ShipSoft/htcondor_submission_scripts)


## Quick start -- `run_fixedTarget.py` example with ganga
You will need 2 files in order to run your script: `bashScript.sh` and `gangaScript.py`.

Your `bashScript.sh` lays out what you want to run. The snippet below
sources the CVMFS aliBuild release **26.04** as an example — substitute
the release you want, or point at your own FairShip build instead.
```
FS_INSTALL=/cvmfs/ship.cern.ch/26.04/
export WORK_DIR=${FS_INSTALL}/sw/
source ${FS_INSTALL}/sw/slc9_x86-64/FairShip/latest/etc/profile.d/init.sh
python ${FS_INSTALL}/sw/slc9_x86-64/FairShip/latest/macro/run_fixedTarget.py $@
```

Once you have your `bashScript.sh` ready, you then need a `gangaScript.py`. Below, we run one job with 10 subjobs; the arguments for all subjobs are passed in the `j.application`, while the arguments per subjob are passed in the `j.splitter`.

```
evtsPerJob = 1000
nSJ = 10

j = Job(name = f'run fixed target - {nSJ * evtsPerJob} events')
j.application = Executable(exe = File('bashScript.sh'), args = ['-o', '"./"', '-n', evtsPerJob, '-e', '5'])
j.splitter = ArgSplitter(args = [['-r', nSJ + _i] for _i in range(nSJ)], append = True)
j.outputfiles = [LocalFile('pythia8_evtgen_Geant4_*.root')]
j.backend = Condor()
j.backend.cdf_options['+MaxRuntime'] = '10000'

j.comment = f'{evtsPerJob} events in each of {nSJ} subjobs'
j.submit()
```

Note that it is also possible to change the output location of your files (eg. to eos) if needed.

Finally, to submit the jobs, run the following: 
```
/cvmfs/ganga.cern.ch/Ganga/install/ship/bin/ganga gangaScript.py
```

To keep an eye on the status of your jobs, you can open ganga with `/cvmfs/ganga.cern.ch/Ganga/install/ship/bin/ganga`.

Documentation for ganga can be found [here](https://ganga.readthedocs.io/en/latest/).
