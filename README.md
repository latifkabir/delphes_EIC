# delphes_EIC

Install Delphes3 following:
https://github.com/delphes/delphes

The detector card contains an EIC detector based on the EIC detector handbook v1.2
http://www.eicug.org/web/sites/default/files/EIC_HANDBOOK_v1.2.pdf

So far it incorporates tracking, EMCAL and HCAL but lacks implementation of PID (it can be done though, following the LHCb card example)

Magnetic field: 1.5 T, Solenoid length: 2.0 m, Tracker radius: 80 cm. 

You can run Pythia8 within Delphes. The command file shown here is suitable for DIS at EIC. 

Run generation command:
`./DelphesPythia8 cards/delphes_card_EIC.tcl examples/Pythia8/DIS.cmnd out.root`

You can see examples of analysis code in the Delphes page above

Run visualization command:
 `root -l examples/EventDisplay.C'("cards/delphes_card_EIC.tcl","out.root")'`
 
The two examples shown here are for neutral-current and charged-current event 
for beam energies of 10 GeV electron on 100 GeV proton (63 GeV center-of-mass energy). 


## Setting up the code


1. Install LHAPDF
   * https://lhapdf.hepforge.org/
   * Using LHAPDF 6.2.3
   * Download the tarball and unpack it
   * BUGFIX in 6.2.3: there is a python coding error in bin/lhapdf. Edit this file and find and replace "add_add_mutually_exclusive_group" with "add_mutually_exclusive_group"
   * Configure it for local installation in your work area, e.g. ```./configure --prefix=/users/ssekula/scratch/EIC/```
   * Build it, ```make -j```,
   * Install it, ```make install```,
   * Make sure the environment is set properly to find the binaries, libraries, and python code (c.f. https://lhapdf.hepforge.org/install.html#lxplus for examples)
1. Install PYTHIA8,
   * http://home.thep.lu.se/~torbjorn/Pythia.html,
   * Download the tarball and unpack it. ,
   * Configure it for local installation in your work area, e.g. ```./configure --prefix=/users/ssekula/scratch/EIC/ --with-lhapdf6=/scratch/users/ssekula/EIC/```,
   * Build it, ```make -j```,
   * Install it, ```make install```,
   * Make sure the work area binary directory is in your PATH: ```PATH=/users/ssekula/scratch/EIC/bin:${PATH}```,
1. Install Delphes,
   * https://github.com/delphes/delphes,
   * Clone the project and make sure you are on the master branch,
   * Make sure ROOT is available in your path, e.g. ```lsetup \"root 6.18.04-x86_64-centos7-gcc8-opt\"```,
   * Compile with PYTHIA8: ```HAS_PYTHIA8=true PYTHIA8=/users/ssekula/scratch/EIC ./configure --prefix=/users/ssekula/scratch/EIC/```,
   * Build: ```make -j```,
   * Install: ```make install```,
1. Get the Delphes/EIC code for simulation and analysis of a detector baseline/configuration.,
   * https://github.com/miguelignacio/delphes_EIC,
   * Clone the repository locally,
   * Follow the instructions to run the example and generate a ROOT file.


## Running Monte Carlo Production

The following command line will run these options:

* e-p beam energies of 10 and 275, respectively
* LHAPDF6:CT18NNLO PDF set
* Output will be written to CC_DIS_e10_p275_CT18NNLO/0/ (where 0 is the task ID, specified using SLURM_ARRAY_TASK_ID)
* 100,000 events generated in a single run of the command

```
SLURM_ARRAY_TASK_ID=0 ./run_study.py --template delphes_card_EIC.tcl --commands CC_DIS_template.cmnd -p '{"PARAM_NEVENTS": 100000, "PARAM_HADBEAM_ENERGY": 275, "PARAM_EBEAM_ENERGY": 10, "PARAM_PDFSET": "LHAPDF6:CT18NNLO"}' -n CC_DIS_e10_p275_CT18NNLO
```

## EIC Collider Variations

Beam energy recommended benchmarking points are (the order is hadron on lepton):

* 275 on 18 GeV
* 100 on 10 GeV
* 100 on 5 GeV
* 41 on 5 GeV


## Running the "SimpleAnalysis" framework

SimpleAnalysis is a basic C++ framework that can operate on the ROOT files produced by Delphes. To compile:

```
cd SimpleAnalysis/
export DELPHES_PATH=<PATH TO DELPHES INSTALLATION>
make
```

This builds ```SimpleAnalysis.exe```. You can then run this on files produced above using the ```run_study.py``` script:

```
./SimpleAnalysis.exe --input_dir '../CC_DIS_e10_p275_CT18NNLO/0/out.root' --output_file test.root --module_sequence KaonPIDModule,ElectronPIDModule,MuonPIDModule,TaggingModule --nevents=100000
```

This runs on a single Delphes ROOT file and produces a new output file, test.root, containing the results of running four modules in sequence:

* KaonPIDModule: uses a very basic PID model (efficiency and K/pi separation) to "identify" kaons from tracks
* ElectronPIDModule: same as kaon PID, but for electrons
* MuonPIDModule: same as kaon PID, but for muons
* TaggingModule: Uses the lists of kaon, muon, and electron candidates provided by the above modules, as well as an implementation of signed-high-impact-parameter track finding, to tag jets.

The output file contains one entry per jet studied with a few basic jet variables. These can be processed using the scripts in ```SimpleAnalysis/scripts``` to make some plots.


