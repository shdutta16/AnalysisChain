# Analysis Chain
Full analysis chain starting from microAOD production (from miniAOD) to limit plots.

The miniAOD datasets are centrally produced. They have different parameters associated with them. Currently, I can remember the following terms that I don't fully understand. Campaign name(?), Eras(?), MetaConditions(?), ReReco(?), UL (ultra legacy)(?), EOY(end of year)(?), Global Tag. When ID value is upgraded only the cut value is changed in case of cut-based and weight file in case of MVA ID. MVAestimatorV2(?)
The different

V2 IDs are absent, only V1 IDs are present. PostRecoTools were recommended to created V2 IDs for the microAOD. Scales and smearings (?).

We do not use flashgg for SM higgs. Our main analyzer is diPhoMoriond_2017.py. beside flashgg we also have MonoHiggs where the analysis is done. microAOD is used to produce diphoton objects from falshgg. MicroAOD is used to analyze the diphoton objects. CMSSW analyzer written by us. After ntuplizer standard C scripts can be run on it. For analysis then ROOT is enough. MiniAOD uses EDM data format and hence, flashgg libraries are required. Usage of flashgg means that it is being used. 

microAOD campaign and metaconditions. Campaign changes and metaconditions changes will affect subsequent steps too. different metaconditions will make different versions of microAOD productions on the same dataset. 

CMSSW_10_6_29, flashgg dev II, MonoHiggs and then scram. 





# Setting up flashgg dev_legacy_runII with MonoHiggsToGG
1. Get flashgg (for more details [link](https://github.com/cms-analysis/flashgg/tree/dev_legacy_runII) )
   ```
   export SCRAM_ARCH=slc7_amd64_gcc700
   cmsrel CMSSW_10_6_29
   cd CMSSW_10_6_29/src
   cmsenv
   git cms-init
   cd $CMSSW_BASE/src 
   git clone -b dev_legacy_runII https://github.com/cms-analysis/flashgg 
   source flashgg/setup_flashgg.sh
   ```
   
2. Get MonoHiggsToGG (after following above steps)
   ```
   cd $CMSSW_BASE/src
   cp -r /afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/src/MonoHiggsToGG ./
   ```
   
3. Before building, change the location of the header file `LeptonSelection.h` to `MonoHiggsToGG/analysis/addfiles/LeptonSelection.h` in the analyzer plugin `MonoHiggsToGG/analysis/plugins/DiPhoAnalyzer_Legacy18.cc`

4. Now build
   `scram b -j 10`
   

## Brief Summary of Issue as of 22/03/2023

The MonoHiggsToGG analyzer `DiPhoAnalyzer_Legacy18.cc` has a method `flashgg::Muon::selectLooseMuons()` which has no definition in the `flashgg/Taggers/interface/LeptonSelection.h` header. The MonoHiggsToGG setup has a header file `LeptonSelection.h` in `MonoHiggsToGG/analysis/addfiles`. When this file was copied in the directory `flashgg/Taggers/interface` (with the tag `_MHgg`), a new error was encountered while compilation. The error was that the MonoHiggsToGG version of the header file was expecting the presence of the member `reco::HitPattern::numberOfHits()`. This class `reco::HitPattern` (`reco` being the namespace) is defined in the header `cmssw/DataFormats/TrackReco/interface/HitPattern.h`. Upon checking in the `cmssw` repo in github, it was found that the member `numberOfHits()` exists in `CMSSW_9_3_X` branch, but it is replaced by `numberOfAllHits()` since `CMSSW_9_4_X` branch. However, the strange thing is that MonoHiggsToGG is compatible with `CMSSW_10_6_8`.

The definition of both the methods `numberOfHits()` and `numberOfAllHits()` is exactly the same. So, just to check, changing the name of the method in the MonoHiggsToGG version of the header file and then compiling. The `numberOfHits()` error was gone but this did not compile due to multiple definitions of methods in `LeptonSelection.h` and `LeptonSelection_MHgg.h`. Tried "switching off" `LeptonSelection.h` and only keeping `LeptonSelection_MHgg.h`. But, four plugins use the `flashgg/Taggers/interface/LeptonSelection.h` which are showing error. The plugins:
```
flashgg/Taggers/plugins/VHLeptonicLoose.cc
flashgg/Taggers/plugins/VHLooseTagProducer.cc
flashgg/Taggers/plugins/DoubleHTagProducer.cc
flashgg/Taggers/plugins/VHLeptonicLoose.cc
```
Introduced new `namespace` as `MHgg` in the MonoHiggsToGG version of the header file. This removed the compilation error but the `FileInPath` error persists. After putting flags in the `DiPhoAnalyzer_Legacy18.cc`, it was seen that not even the method `beginJob()` is being executed. The error is thrown even before that. 


### Update 23/03/2023

If the analyzer is unable to access the input root file then the following error is being thrown which is different from what I am getting in the above case. This is checked by executing `cmsRun diPhoAna_2018.py` on the working analyzer (CMSSW_10_6_8) and deliberately giving a wrong path to the input root file. 
```
Now trying to get input file
cms.untracked.vstring('/ste/user/spigazzi/flashgg/Era2018_RR-17Sep2018_v2/legacyRun2FullV2/TGJets_TuneCP5_13TeV_amcatnlo_madspin_pythia8/Era2018_RR-17Sep2018_v2-legacyRun2FullV2-v0-RunIIAutumn18MiniAOD-102X_upgrade2018_realistic_v15-v2/190717_094233/0000/myMicroAODOutputFile_1.root')
flashggDiPhotonsVtx0
----- Begin Fatal Exception 23-Mar-2023 08:12:13 CET-----------------------
An exception of category 'LogicalFileNameNotFound' occurred while
   [0] Constructing the EventProcessor
   [1] Constructing input source of type PoolSource
Exception Message:
RootFileSequenceBase::initTheFile()
Logical file name '/ste/user/spigazzi/flashgg/Era2018_RR-17Sep2018_v2/legacyRun2FullV2/TGJets_TuneCP5_13TeV_amcatnlo_madspin_pythia8/Era2018_RR-17Sep2018_v2-legacyRun2FullV2-v0-RunIIAutumn18MiniAOD-102X_upgrade2018_realistic_v15-v2/190717_094233/0000/myMicroAODOutputFile_1.root' was not found in the file catalog.
If you wanted a local file, you forgot the 'file:' prefix
before the file name in your configuration file.
----- End Fatal Exception -------------------------------------------------
```

Again, by deliberately putting erroneous path to the configuration files (variables `photonFileName` and `jetFileName`), the following errors are thrown where, the erroneous path is explicitly included in the error statements. 
```
flashggDiPhotonsVtx0
----- Begin Fatal Exception 23-Mar-2023 08:18:54 CET-----------------------
An exception of category 'ConfigFileReadError' occurred while
   [0] Processing the python configuration file named diPhoAna_2018.py
Exception Message:
 unknown python problem occurred.
RuntimeError: An exception of category 'FileInPathError' occurred.
Exception Message:
edm::FileInPath unable to find file flashgg/Tggers/data/L1prefiring_photonpt_2017BtoF.root anywhere in the search path.
The search path is defined by: CMSSW_SEARCH_PATH
${CMSSW_SEARCH_PATH} is: /afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/poison:/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/src:/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/external/slc7_amd64_gcc700/data:/cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/src:/cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/external/slc7_amd64_gcc700/data
Current directory is: /afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/src/MonoHiggsToGG/analysis/test/MC_sig/ggTomonoH_aa_sinp0p35_tanb1p0_MXd10_MH3_300_MH4_200


At:
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Types.py(639): insertInto
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Mixins.py(356): insertContentsInto
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Mixins.py(482): insertInto
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Modules.py(162): insertInto
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Config.py(900): _insertManyInto
  /cvmfs/cms.cern.ch/slc7_amd64_gcc700/cms/cmssw/CMSSW_10_6_8/python/FWCore/ParameterSet/Config.py(1102): fillProcessDesc
  <string>(2): <module>

----- End Fatal Exception -------------------------------------------------
```

So, most likely the paths to these config files are not the source of the problem. 

The same error as of CMSSW_10_6_29 could be reproduced in CMSSW_10_6_9 by leaving `photonFileName` completely empty. So, why is this error being encountered in the former even though the string is filled needs to be checked now. It might be there is some other module that is using `cms.FileInPath` with an empty string. Need to find that. It is most likely in flashgg.

Did a `grep -r "FileInPath" .` in the flashgg directory (of CMSSW_10_6_29). Among numerous results found the following two which are being used in the `diPhoAna_2018.py`
```
./MicroAOD/python/flashggDiPhotons_cfi.py:                                  vertexIdMVAweightfile   = cms.FileInPath(""),
./MicroAOD/python/flashggDiPhotons_cfi.py:                                  vertexProbMVAweightfile = cms.FileInPath(""),
```
Finding the analogous lines in the previous flashgg (of CMSSW_10_6_8) found the following:
```
./MicroAOD/python/flashggDiPhotons_cfi.py:                                  vertexIdMVAweightfile   = cms.FileInPath("flashgg/MicroAOD/data/TMVAClassification_BDTVtxId_SL_2016.xml"),
./MicroAOD/python/flashggDiPhotons_cfi.py:                                  vertexProbMVAweightfile = cms.FileInPath("flashgg/MicroAOD/data/TMVAClassification_BDTVtxProb_SL_2016.xml"),
```
Replaced the empty strings in flashgg directory (of CMSSW_10_6_29), IT IS WORKING NOW!

After running the analyzer on the microAOD, the previous microAOD and the one produced by Ashim da, both gave the following error:
```
Begin processing the 100th record. Run 1, Event 107, LumiSection 1 on stream 0 at 23-Mar-2023 11:59:12.224 CET
----- Begin SkipEvent Exception 23-Mar-2023 11:59:12 CET-----------------------
An exception of category 'ProductNotFound' occurred while
   [0] Processing  Event run: 1 lumi: 1 event: 107 stream: 0
   [1] Running path 'p'
   [2] Calling method for module FlashggDiPhotonProducer/'flashggDiPhotonsVtx0'
Exception Message:
Principal::getByToken: Found zero products matching all criteria
Looking for a container with elements of type: reco::Conversion
Looking for module label: reducedEgamma
Looking for productInstanceName: reducedSingleLegConversions

----- End SkipEvent Exception -------------------------------------------------
```

On suggestion from Debabrata da, changing `useVtx0` to `False`. 
Reason: ""flashggDiPhotonVtx0" is corresponding to the choice among the two possible vertex scenarios, the CMS reconstructed primary vertex and H->gg most probable true vertex. To quickly check if it's not getting the BDT vertex and can work with CMS primary vertex
In that case we can look into. This fraction of the diPhoAna, and see if all the things are correctly located as mentioned" - Debabrata da

This did away the previous error. It confirms what Debabrata da had doubted. 



### Discussion with Satyaki sir and MHgg skype group

There are two ways by which the primary vertex is calculated that is relevant for the analysis. The first is the way it is calculated in CMS. Whenever there is a hard collision between two protons (it is the quarks that actually "collide"), there are also other particles (quarks, anti-quarks, gluons etc.) near the hard collision. These other particles too "collide" albeit softly. After the collision these soft collisions emit many gluons and quarks as debris that soon hadronize and form jets. These jets are composed of charged hadrons (mostly charged pions, since they are the lightest). Track fitting is done on them and then checked where all these tracks are converging. Each vertex so created is a candidate for being the primary vertex. The final selection is done based on the criteria of having max &Sigma;p<sub>T</sub><sup>2</sup>). This is the CMS-calculated primary vertex.  



# Making Limit plots

The microAOD trees need to skimmed appropriately. In the skimming step new branches will be added to the trees corresponding to quantities that are required (and hence need to be calculated). In the recent discussions, it was decided in the cut-based approach only events that pass the cuts will be stored in the skimmed trees, while in case of the DNN (parametric or non-parametric), all the events will be stored along with the DNN score. The `skim_paraDNN.py` (`skim_DNN.py`) skims the trees based on the parametric DNN (non-parametric DNN) and can also produce trees implementing the cut-based method. 

Note that while loading the DNN model json file in the above python script the following error is encountered (if not already corrected).
```
Traceback (most recent call last):
  File "skim_paraDNN.py", line 593, in <module>
    dnn_model = model_from_json( dnn_model_json )
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/saving/model_config.py", line 96, in model_from_json
    return deserialize(config, custom_objects=custom_objects)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/layers/serialization.py", line 106, in deserialize
    printable_module_name='layer')
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/utils/generic_utils.py", line 303, in deserialize_keras_object
    list(custom_objects.items())))
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/engine/sequential.py", line 377, in from_config
    custom_objects=custom_objects)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/layers/serialization.py", line 106, in deserialize
    printable_module_name='layer')
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/utils/generic_utils.py", line 305, in deserialize_keras_object
    return cls.from_config(cls_config)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/engine/base_layer.py", line 519, in from_config
    return cls(**config)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/layers/core.py", line 1086, in __init__
    self.kernel_regularizer = regularizers.get(kernel_regularizer)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/regularizers.py", line 302, in get
    return deserialize(identifier)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/regularizers.py", line 294, in deserialize
    printable_module_name='regularizer')
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/utils/generic_utils.py", line 292, in deserialize_keras_object
    config, module_objects, custom_objects, printable_module_name)
  File "/home/shubhamdutta/.local/lib/python2.7/site-packages/tensorflow_core/python/keras/utils/generic_utils.py", line 250, in class_and_config_for_serialized_keras_object
    raise ValueError('Unknown ' + printable_module_name + ': ' + class_name)
ValueError: Unknown regularizer: L2
```
The solution is to change `kernel_regularizer": {"class_name": "L2"` to `"kernel_regularizer": {"class_name": "L1L2"` in the json file. The error is due to the way the model is saved in the json file. For more details refer to the [link](https://stackoverflow.com/questions/64063914/unknown-regularizer-l2-in-tensorflowjs). 


The events will then be furthur divided into bins of DNN score (or MET bins) using the script `fitterFormatting_DNNcat_array.cc` (or `fitterFormatting_METcat_array.cc`) (these scripts are run by using the bash script `formatNtupleForFitting_DNNcat_array.sh` (`formatNtupleForFitting_METcat_array.sh`) by passing appropriate arguments). 




## Unable to recreate previous limit plot (of 2021) with current setup

Using the current setup, attempts were made to recreate the plots that were made in 2021. However, till now it has not been possible. Initially (initial in 2023), when the plots were made using the DNN skimmed trees both from the parametric and non-parametric models, it did not give the best plot (of DNN) produced in 2021. To check this further, the DNN skimmed trees of 2021 were used to make the plots and they too didn't give the best result of 2020. Next, the trees produced in 2021 (from the path `/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/src/MonoHiggsToGG/analysis/fits/2018_2HDMa_EOY/LowMET`), after the MET categorization using the `fitterformatting_METcat_array.cc` script, were used to make the limit plots and it DID GIVE THE BEST PLOT OF 2021. So, after this the differences were checked in the `fitterformatting_METcat_array.cc` script to find out if any additional cuts were used after skimming. It was found that there was an additional cut on ptgg corresponding to highMET and lowMET. Events with MET 50 < met < 150 were chosen with ptgg > 40 while events with met > 150 were chosen with ptgg > 90. The reason for this could be understood from the ptgg vs. MET 2D histograms. However, even using this condition in the current `fitterformatting_METcat_array.cc` script, the best of 2020 could not be reproduced. On comparing the trees of 2021 and 2023 after the "fitterformatting" step, substantial difference was observed in the ptgg distribution of the events which till now (as of 30/03/2023) could not be understood. This is being investigated further. 

### Some directory nomenclatures:

SKIMMED TREES DIRECTORIES:

`DNN_skim_METbins` -> Skimmed trees with non-parametric DNN for making MET bins (MET-binning is not done yet; this is before "fitterformatting_METcat" step). It has events passing the cut dnnScore > 0.95

`DNN_skim_METbins_withPreviousSkimmedTrees` -> Skimmed trees of 2021. Remaining details same as above. 

`DNN_skim_DNNbins` -> Skimmed trees with non-parametric DNN for making DNN bins (DNN-binng is not done yet; this is before "fitterformatting_DNNcat" step). It has all events, since the cut is dnnScore > 0.0

`paraDNN_skim_METbins` -> Skimmed trees with parametric DNN for making MET bins (MET-binning is not done yet; this is before "fitterformatting_METcat" step). It has events passing the cut dnnScore > 0.95. Since it is parametric, it has one set of trees corresponding to each mA mass (200, 300, 400, 500, 600). 

`paraDNN_skim_DNNbins` -> Skimmed trees with parametric DNN for making DNN bins (DNN-binning is not done yet; this is before "fitterformatting_DNNcat" step). It has all events, since the cut is dnnScore > 0.0. Since it is parametric, it has one set of trees corresponding to each mA mass (200, 300, 400, 500, 600).


AFTER FITTER-FORMATTING STEP DIRECTORIES:

`ntuples4fit_DNN` -> For trees produced from non-parametric DNN using current setup i.e. current `fitterformatting_METcat_array.cc` script. This has the following sub-directories:
1. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_noPtggCut` -> no ptgg cut 
2. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90` -> ptgg > 40 cut for MET 50-150; ptgg > 90 for MET >150
3. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_150+_ptgg90` -> ptgg > 90 for all MET categories


`ntuples4fit_paraDNN` -> For trees produced from non-parametric DNN using current setup i.e. current `fitterformatting_METcat_array.cc` script. This has the following sub-directories:
1. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_noPtggCut` -> no ptgg cut 
2. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90` -> ptgg > 40 cut for MET 50-150; ptgg > 90 for MET >150
3. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_150+_ptgg90` -> ptgg > 90 for all MET categories


`ntuples4fit_DNN_withPreviousSkimmedTrees` -> For trees produced from DNN skimmed tress of 2020, but using the current `fitterformatting_METcat_array.cc` script. This has the following sub-directory:
1. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_noPtggCut` -> no ptgg cut


`ntuples4fit_DNN_withPreviousNtuples4FitTrees` -> For trees produced in 2021 after the "fitterformatting" step. This has the following sub-directory:
1. `ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_DNN_ptgg` -> Same name as used in 2020. 
This directory has additional two files `fitterFormatting_METcat.cc` and `fitterFormatting_METcat_array.cc` which are taken from the same directory (`/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/CMSSW_10_6_8/src/MonoHiggsToGG/analysis/fits/2018_2HDMa_EOY/LowMET`) from where the above trees were copied/found. 

03/04/2023
Two changes were made. First, the normalization of the inputs (to the network) was changed from the given array to column max and second the 2021 weight file (from `/home/shubhamdutta/ANAWORK/MonoHgg/2018Analysis/2018_2HDMa_EOY/AnalysisSelection_allvar_womgg_ANN_model.h5`) is being used. With these two changes, the efficiency nos. of 16/06/2021 could be reproduced viz. 
```
200 ---> 52.89%
300 ---> 46.37%
400 ---> 76.98%
500 ---> 80.37%
600 ---> 73.1%
```

However, this still doesn't reproduce the 'yellow line' of the comparison plot. 


10/04/2023 ISSUE SOLVED
Using the skimmed trees of 2021 with the 2023 MET categorization script, the limit plot did not coinicide with the yellow-line. But using the current 2023 version of non-parametric model but using the 2021 version of weight files and the MET categorization script, the limit plot exactly coincides with the yellow line! This confirms that the issue is with the MET categorization script and neither with the DNN skim script nor the datacard generator scripts. After running `diff` between the 2021 and 2023 MET cat scripts, no important difference could be found (there were some trivial differences only). However, on checking the bash script running the MET categorization script, it was found that in the 2023 verion `whichDphi=3` which was enabling the following and introducing an extra cut on dphi. 
```
   if (whichDphi==1 && dphiggmet < 2.1)                        continue;
   if (whichDphi==2 && mindphijmet < 0.5)                      continue;
   if (whichDphi==3 && (dphiggmet < 2.1 || mindphijmet < 0.5)) continue;
```
After setting `whichDphi=0` (as was done in the 2021 version), the issue was solved and now we get back the same trees as was obtained in 2021. 


## Chain to make limit plots under current setup of 10/04/2023 with MET categorization

1. Make a directory in the location `/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/Fit_DiPhotonTools/CMSSW_9_4_9/src/diphotons/Analysis/macros/2018_2HDMa_EOY_reloaded/`, for example `MET_DNN_normWithMax` was made for non-parametric DNN model with MET categorization and the input nodes of the DNN normalized by the column max. 

2. Copy the following files:
   
   i) For non-parametric DNN model copy from `MET_DNN` directory
      ```
      auto_plotter.py
      combine_maker_MonoHgg.py
      combine_maker_MonoHgg.sh
      templates_maker_MonoHgg.py
      mycombineall_MonoHgg2HDMa.sh
      mylimit_plots_MonoHgg_2HDMa.py
      mylimit_plots_MonoHgg.sh
      run_combineMakerMonoHgg.sh
      run_myCombineAlMonoHiggs2HDMa.sh
      run_limitPlotsMonoHgg.sh
      <ALL THE JSON FILES>
      ```
   ii) For parametric DNN model copy from `MET_paraDNN` directory 
      ```
      auto_plotter.py
      combine_maker_MonoHgg_test.py
      combine_maker_MonoHgg_test.sh
      templates_maker_MonoHgg.py
      mycombineall_MonoHgg2HDMa.sh
      mylimit_plots_MonoHgg_2HDMa.py
      mylimit_plots_MonoHgg.sh
      run_combineMakerMonoHgg.sh
      run_myCombineAlMonoHiggs2HDMa.sh
      run_limitPlotsMonoHgg.sh
      <ALL THE JSON FILES>
      ```
3. Open `combine_maker_MonoHgg.sh` (`combine_maker_MonoHgg_test.sh`) for non-parametric DNN (parametric DNN) and change the path to the current directory, for example `www=/afs/cern.ch/work/s/shdutta/public/Analysis/MHgg/2018Analysis/Fit_DiPhotonTools/CMSSW_9_4_9/src/diphotons/Analysis/macros/2018_2HDMa_EOY_reloaded/MET_DNN_normWithMax` was done for `MET_DNN_normWithMax`.

4. Open `mylimit_plots_MonoHgg.sh` and change the path to the current directory for the `-O` argument.

5. Copy the directory containing the trees produced after the MET categorization (after running `fitterFormatting_METcat_array` script) to the current location.

6. Make copies of this directory corresponding to each MET category as shown in the following example:
   ```
   cp -r ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90_MET_DNN2023_normWithMax_2021fitterFormatting_METcat_array ntuples4fit_pho_newSig_test_metBins_50_70
   cp -r ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90_MET_DNN2023_normWithMax_2021fitterFormatting_METcat_array ntuples4fit_pho_newSig_test_metBins_70_100
   cp -r ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90_MET_DNN2023_normWithMax_2021fitterFormatting_METcat_array ntuples4fit_pho_newSig_test_metBins_100_130
   cp -r ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90_MET_DNN2023_normWithMax_2021fitterFormatting_METcat_array ntuples4fit_pho_newSig_test_metBins_130_150
   cp -r ntuples4fit_pho_newSig_test_metBins_50_70_100_130_150_ptgg40_150+_ptgg90_MET_DNN2023_normWithMax_2021fitterFormatting_METcat_array ntuples4fit_pho_newSig_test_metBins_150
   ```
 7. Execute the following to make the plots:
    ```
    cmsenv
    ./run_combineMakerMonoHgg.sh
    ./run_myCombineAlMonoHiggs2HDMa.sh
    ./run_limitPlotsMonoHgg.sh
    ```
