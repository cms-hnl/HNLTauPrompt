## How to run miniAOD->nanoAOD skims production with HNLProd

### How to install
Connect to CentOS8 machine and clone the central repository.
```sh
git clone --recursive git@github.com:cms-hnl/HNLProd.git
```

### Loading environment
Following command activates the framework environment:
```sh
source $PWD/env.sh nano_prod

## How to run miniAOD->nanoAOD skims production

Production should be run on the server that have the crab stageout area mounted to the file system.
1. Load environment on CentOS8 machine
   ```sh
   source $PWD/env.sh nano_prod
   source /cvmfs/cms.cern.ch/common/crab-setup.sh
   voms-proxy-init -voms cms -rfc -valid 192:00
   ```

1. Create crab configs
   ```sh
   python3 NanoProd/createCrabConfigs.py --samples config/samples_2018.yaml --output crab/Run2_2018
   ```

1. Check that all datasets are present and valid:
   ```sh
   cat crab/Run2_2018/all_samples.txt| xargs python3 RunKit/checkDatasetExistance.py
   ```

1. Modify output and other site-specific settings in `config/overseer_cfg.yaml`. In particular:
   - site
   - crabOutput
   - localCrabOutput
   - finalOutput
   - renewKerberosTicket

1. Test that the code works locally (take one of the miniAOD files as an input). E.g.
   ```sh
   python3 RunKit/nanoProdWrapper.py customise=HNLTauPrompt/NanoProd/customiseNano.customise skimCfg=HNLTauPrompt/config/skim_v10.yaml maxEvents=100 sampleType=mc storeFailed=True era=Run2_2018 inputFiles=file:/eos/cms/store/group/phys_tau/kandroso/miniAOD_UL18/TTToSemiLeptonic.root
   ./RunKit/nanoProdCrabJob.sh
   ```
   - check that output file `nano.root` is created correctly

1. Test a dryrun crab submission
   ```sh
   python3 RunKit/crabOverseer.py --work-area crab_test --cfg HNLTauPrompt/config/overseer_cfg.yaml --no-loop HNLTauPrompt/config/crab_test.yaml
   ```
   - NB. Crab estimates of processing time will not be accurate, ignore them.
   - After the test, remove `crab_test` directory:
     ```sh
     rm -r crab_test
     ```

1. Test that post-processing task is known to law:
   ```sh
   law index
   law run CrabNanoProdTaskPostProcess --help
   ```

1. Submit tasks using `RunKit/crabOverseer.py` and monitor the process.
   It is recommended to run `crabOverseer` in screen.
   ```sh
   python3 RunKit/crabOverseer.py --cfg HNLTauPrompt/config/overseer_cfg.yaml crab/Run2_2018/FILE1.yaml crab/Run2_2018/FILE2.yaml ...
   ```
   - Use `crab/Run2_2018/*.yaml` to submit all the tasks
   - For more information about available command line arguments run `python3 RunKit/crabOverseer.py --help`
   - For consecutive runs, if there are no modifications in the configs, it is enough to run `crabOverseer` without any arguments:
     ```sh
     python3 RunKit/crabOverseer.py
     ```