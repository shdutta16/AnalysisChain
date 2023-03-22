# AnalysisChain
Full analysis chain starting from microAOD production (from miniAOD) to limit plots

The miniAOD datasets are centrally produced. They have different parameters associated with them. Currently, I can remember the following terms that I don't fully understand. Campaign name(?), Eras(?), MetaConditions(?), ReReco(?), UL (ultra legacy)(?), EOY(end of year)(?), Global Tag. When ID value is upgraded only the cut value is changed in case of cut-based and weight file in case of MVA ID. MVAestimatorV2(?)

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
