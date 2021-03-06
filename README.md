# Tau Reconstruction exercise @ CMSPOS
In this exercise you'll see first-hand how the tau reconstruction is run on low level, Particle Flow inputs.  

To make things clearer to follow, you'll run the tau reconstruction on miniAOD samples (instead of the usual RECO/AOD). 

In this way the typical turnaround change parameters -> re-run the reconstruction -> investigate the outcome is much faster and will allow you to actually test a few different configurations and see their impact on the relevant quantities, such as efficiency and fake rejection. 

## Installation and setup
First of all you need to initialise a fresh CMSSW release and install the needed packages.

```
# setup the CMSSW release
cmsrel CMSSW_9_2_7
cd CMSSW_9_2_7/src
cmsenv

# checkout the necessary packages
git-cms-addpkg /DataFormats/PatCandidates/
git-cms-addpkg /DataFormats/TauReco/
git-cms-addpkg /PhysicsTools/PatAlgos/
git-cms-addpkg /RecoTauTag/RecoTau/

# add Jan’s repository
git remote add jan https://github.com/steggema/cmssw.git
git fetch jan

# create a new branch and make it point to Jan’s
git checkout CMSSW_9_2_X_TauRecoMiniAOD

# now compile
scram b -j16

# move to the RecoTau package
cd RecoTauTag/RecoTau/test/

# get the cfg file and run it, this will produce 'outputFULL.root' containing everything from the original files
# *plus* your new tau collections
wget https://gist.githubusercontent.com/steggema/f5b65e09ebee723f5f27b1bef53dfa03/raw/05e064b65d0fe4c3aeeb65b028ccbda857b27814/tau_miniaod.py
voms-proxy-init --voms cms
cmsRun tau_miniaod.py

# suggestion: check that the output files really contains what you want using edmDumpEventContent
```

## Produce flat root trees
Now that you have run the tau reconstruction on the original miniAOD samples and you have produced `outputFULL.root`, you need to produce handy and swift ROOT flat trees to be used afterwards for producing plots and studies.

The main piece of code is `read_taus.py`. We encourage you to go through it before running it, it is thoroughly commented and it is (should be, in case just ask!) self explanatory.

The basic idea is to run over the sample you produced, operate the geometrical matching between reco and generator-level taus and fill *two* flat ROOT trees:
* one with an entry for each generated hadronic tau
* one with an entry for each reconstructed jet (used as a seed to the tau reconstruction)

```
# clone the present package
cd $CMSSW_BASE/src/RecoTauTag/RecoTau/test/
cmsenv
git clone https://github.com/rmanzoni/TauRecoCMSPOS.git
cd TauRecoCMSPOS

# create a soft link to outputFULL.root (or alternatively move it to your convenience...)
ln -s ../outputFULL.root

# run the ntupliser! (ipython is not mandatory, but it simply happens to be more useful than the bare python in many contexts)
ipython -i read_taus.py 

# now you've finally obtained your two flat trees
tau_gen_tuple.root
tau_reco_tuple.root
```

inspect your two ntuples with ROOT's TBrowser: you'll notice that there are only a few basic quantities saved (where is the isolation?!).

Try to go through the code again, understand how to add more information to the trees and add the isolation discriminators.

## Plotting
Once you have produced the flat ntuples, you can now easily perform studies and produce plots out of them.  

For example you can inspect `plotting.py` and run it `ipython -i plotting.py`.  
It will produce two plots showing the distribution and the efficiency of reconstructed quantities with respect to the generated ones.

Use this file as an example and produce:
1. pulls split by decay mode
2. efficiency and fake rates for different isolation working points
  1. as a function of pt
  2. as a function of number of vertices
3. ROC curves
4. 2D table gen vs reco decay mode (a.k.a. decay mode migration matrix)

## Vary tau reconstruction parameters and observe their effect on the higher level observables
After the first spin, try now to play with the tau reconstruction!  

We suggest a number of parameters you can play with:
* remove 2prong and 3-prong + pi0 decay modes
* relax and/or tighten the intermediate resonance mass cuts
* vary the signal cone size
* increase the pt requirements for a strip to be identified as a pi0
* change track quality requirements (e.g. number of hits)

For each of them, which effect do you expect to see? Rerun the reconstruction, ntuplisation and plotting and test it!

In this example `customise_tau_reco_miniaod_cfg.py` you can see how to change the isolation cone size. Run it with `cmsRun customise_tau_reco_miniaod_cfg.py`

Full example to answer all questions can be found here (don't peek before you tried yourself)
https://gist.github.com/rmanzoni/26f8d4eb09bd8e103b421b5156fc1f17

## Tips & Tricks
* Let the job run even ifthe session is closed: https://en.wikipedia.org/wiki/Nohup
* change file name and run! https://gist.github.com/rmanzoni/3fb37a61e4b74b55b6e948ecd0fb36b4
* reco decay mode logic: `decayMode = 5*(n_ch-1) + n_pi0`
   * e.g.: 1-prong + 1pi0 ==> `5*(1-1) + 1 = 1`
* generated decay mode: https://github.com/cms-sw/cmssw/blob/master/PhysicsTools/Heppy/python/physicsutils/TauDecayModes.py#L44-L46
* python debugger: https://docs.python.org/2/library/pdb.html
   * insert this line `import pdb; pdb.set_trace()` at the exact point you want to pause the script execution and interactively inspect the code

## Save the results!

Please save your plots at this location
`/home/common/LongExercises/Tau/TauRECO/results`

In order to keep things tidy, please organise into different subdirectories, e.g.:
`/home/common/LongExercises/Tau/TauRECO/results/nominal_reco`

Perhaps it is not needed, but please set the unix permissions to 777, so taht anybody can do anything and all of us have access to the results
`chmod 777 YOUR_DIR`

As for the format, `pdf` is fine but root histograms saved into a root file would be better for when we'll produce the final plots for the presentation.

Here an example
```
outfile = ROOT.TFile.Open('myresults.root', 'recreate')
outfile.cd()
myhistogram.Write()
outfile.Close()
```
### Generate all plots and save them to a rootfile with one script

Use the script `/home/common/LongExercises/Tau/TauRECO/results/nominal_reco/plotting_to_root.py` to generate the relevant plots and dump them to a root file.

Lines you need to change before execution:
* The filenames of your tau_tuples (sim for the signal, reco for the signal)
* The output filename and directory corresponding to your task

Copy the rootfile then to the results directory corresponding to your task.
Please also make a small file there noting which iso discriminant you used and copy your custom config script so is is clear which parameters you changed.


