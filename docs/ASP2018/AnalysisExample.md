# ATLAS Analysis Example

## Introduction

Root may be run in batch mode on the grid to analyze large data samples. This example creates simulated data in root format using trees and performs analysis on the simulated data by means of processing on the grid. This example is based on a demo developed by OU programmer Chris Walker.

## Prerequisite 

* Login on submission node

```
$ ssh -XY YOUR_USER_ID@user-training.osgconnect.net
```

* Make a directory for this exercise

```
$ mkdir -p analysis_example
$ cd analysis_example
```

Again the `$` sign at the beginning of the commands to execute is the *command prompt*, so it should *not* be entered as part of the command.

## Simple Analysis Example

### Step 1: Create simulated data using the grid

Now in your test directory on the submission host we will create the three files: *run-root.cmd*, *run-root.sh*, and *run-root.C* with the contents given below. This may require running an editor such as `emacs` on your local desktop and then copying the created files to the submission host. Or the `nano` editor can be run directly on the submission host. A typical copy command would be as follows. 

```
$ scp run-root.* YOUR_USER_ID@user-training.osgconnect.net:analysis_example/
```

It is probably easier to create all scripts with `nano` on the submission node, though, and then you won't have to copy (`scp`) anything at all. So everything below assumes you are logged on to a terminal session on the submission node.

First, we will utilize a simple command script to submit the grid jobs. It is *run-root.cmd*:

```
universe=vanilla
executable=run-root.sh
transfer_input_files = run-root.C
transfer_executable=True
when_to_transfer_output = ON_EXIT
log=run-root.log
transfer_output_files = root.out,t00.root,t01.root
output=run-root.out.$(Cluster).$(Process)
error=run-root.err.$(Cluster).$(Process)
notification=Never
queue 
```

Note that the executable script is:  *run-root.sh* which is as follows:

```
#!/bin/bash 
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
module load root
module load libXpm
root -b < run-root.C > root.out
```

This script runs Root in batch mode and executes input macro *run-root.C* and produces output that is routed to file *root.out*.
It has to be made executable, by use of the `chmod` Linux command (protections can be checked with the command `ls -l`):

```
$ chmod +x run-root.sh
```

The macro  *run-root.C* consists of the following code:

```
{ 
 
 // create files containing simulated data
 
 TRandom g; 
 char c[256]; 
 for ( int j = 0 ; j < 2 ; j++ ){ 
    sprintf(c,"t%2.2d.root\000",j); 
    TFile f(c,"RECREATE","MyFile", 0/*no compression*/); 
    TTree *t = new TTree("t0","t0"); 
    Int_t Run; 
    TBranch * b_Run = t->Branch("Run",&Run); 
    Int_t Event; 
    TBranch * b_Event = t->Branch("Event",&Event); 
    Float_t Energy; 
    TBranch * b_Energy = t->Branch("Energy",&Energy); 
    Run = j; 
 
        for( Event = 0 ; Event < 100 ; Event++ ){ 
          Energy = g.Gaus(500.0 , 200.0);   
          t->Fill(); 
        }  
    f.Write(); 
    f.Close(); 
 } 
} 
.q 
```

The grid job can be submitted using:

```
$ condor_submit run-root.cmd
```

It can be checked with: 

```
$ condor_q YOUR_USER_ID -nobatch
```

After it runs, you will find a log file that describes the job: *run-root.log*, and output file: *root.out*, and the files containing the simulated data: *t00.root*, *t01.root* in your test directory. 
You need to copy these files into your public directory, so that you can download it to your local desktop:

```
$ cp t0*.root ~/public/
```

Now open a different terminal window on your local desktop, and download the root files with:

```
$ wget http://stash.osgconnect.net/~YOUR_USER_ID/t00.root  http://stash.osgconnect.net/~YOUR_USER_ID/t01.root
```

You can then inspect the contents of *t00.root* and *t01.root* by running Root in your current directory in the local terminal window in which you just ran the `wget` command:

```
$ root t00.root
```

And then the Root command:  *TBrowser b*

With the *TBrowser* you can plot the simulated data in branch *Energy* as well as the other branches. Double click on the name of the root files, and then on the variables you would like to plot.

Each data file contains a TTree named *t0*. You can plot the contents of all (in this example both) data file TTree's by using the TChain method as follows:

In Root execute the following commands:

```
TChain tc("t0");
tc.Add("t*.root");
tc.Draw("Energy");
```

When you are done with this, you can quit *root* again with the command `.q <Return>`.

### Step 2: Analyze Real Data

Now we want to have a look at a real live ATLAS root file. For this, go back to the remote terminal window on osgconnect. You will need a new condor submit script called *run-z.cmd*:

```
universe=vanilla
executable=run-z.sh
transfer_input_files = readEvents.C,/home/pskubic/public/muons.root
transfer_executable=True
when_to_transfer_output = ON_EXIT
log=run-z.log
transfer_output_files = root-z.out,histograms-z.root
output=run-z.out.$(Cluster).$(Process)
error=run-z.err.$(Cluster).$(Process)
notification=Never
queue 
```
The new executable script you need for this job is:  *run-z.sh* which is as follows:
```
#!/bin/bash 
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
module load root
module load libXpm
root -b -q readEvents.C+ > root-z.out
```
This script runs Root in batch mode and executes input macro *readEvents.C* and produces output that is routed to file *root-z.out*.
It has to be made executable, by use of the `chmod` Linux command (protections can be checked with the command `ls -l`):

```
$ chmod +x run-z.sh
```

The macro  *readEvents.C* consists of the following code:

```
#include "TFile.h"
#include "TTree.h"
#include "TCanvas.h"
#include "TH1F.h"
#include "iostream"
//#include "TLorentzVector.h"
using namespace std;

void readEvents(){

	// load the ROOT ntuple file
	TFile * f = new TFile("muons.root");
	TTree *tree = (TTree *) f->Get("POOLCollectionTree");
	int nEntries = tree->GetEntries();
	cout << "There are " << nEntries << " entries in your ntuple" << endl;
	
	// create local variables for the tree's branches
	UInt_t NLooseMuons;
	Float_t LooseMuonsEta1;
	Float_t LooseMuonsPhi1;
	Float_t LooseMuonsPt1;

	Float_t LooseMuonsEta2;
	Float_t LooseMuonsPhi2;
	Float_t LooseMuonsPt2;
	
	// set the tree's braches to the local variables
	tree->SetBranchAddress("NLooseMuon", &NLooseMuons);
	tree->SetBranchAddress("LooseMuonEta1", &LooseMuonsEta1);
	tree->SetBranchAddress("LooseMuonPhi1", &LooseMuonsPhi1);
	tree->SetBranchAddress("LooseMuonPt1", &LooseMuonsPt1);
	
	tree->SetBranchAddress("LooseMuonEta2", &LooseMuonsEta2);
	tree->SetBranchAddress("LooseMuonPhi2", &LooseMuonsPhi2);
	tree->SetBranchAddress("LooseMuonPt2", &LooseMuonsPt2);
	
	// declare some histograms
  TH1F *muPt1 = new TH1F("muPt1", ";p_{T} [GeV/c];Events", 50, 0, 200);
  TH1F *muPx1 = new TH1F("muPx1", ";p_{x} [GeV/c];Events", 50, 0, 200); //added px
  TH1F *muPy1 = new TH1F("muPy1", ";p_{y} [GeV/c];Events", 50, 0, 200); //added py
  TH1F *muPz1 = new TH1F("muPz1", ";p_{z} [GeV/c];Events", 50, 0, 200); //added pz
  TH1F *muEta1 = new TH1F("muEta1", ";#eta;Events", 50, -3, 3);
  TH1F *muPhi1 = new TH1F("muPhi1", ";#phi;Events", 50, -4, 4);
  TH1F *muE1 = new TH1F("muE1", ";Energy;Events", 50, 0, 200);
  
  TH1F *muPt2 = new TH1F("muPt2", ";p_{T} [GeV/c];Events", 50, 0, 200);
  TH1F *muPx2 = new TH1F("muPx2", ";p_{x} [GeV/c];Events", 50, 0, 200); //added px
  TH1F *muPy2 = new TH1F("muPy2", ";p_{y} [GeV/c];Events", 50, 0, 200); //added py
  TH1F *muPz2 = new TH1F("muPz2", ";p_{z} [GeV/c];Events", 50, 0, 200); //added pz
  TH1F *muEta2 = new TH1F("muEta2", ";#eta;Events", 50, -3, 3);
  TH1F *muPhi2 = new TH1F("muPhi2", ";#phi;Events", 50, -4, 4);
  TH1F *muE2 = new TH1F("muE2", ";Energy;Events", 50, 0, 200);
  
  TH1F *zPt = new TH1F("zPt", ";p_{T} [GeV/c];Events", 50, 0, 200);
  TH1F *zPx = new TH1F("zPx", ";p_{x} [GeV/c];Events", 50, 0, 200); //added px
  TH1F *zPy = new TH1F("zPy", ";p_{y} [GeV/c];Events", 50, 0, 200); //added py
  TH1F *zPz = new TH1F("zPz", ";p_{z} [GeV/c];Events", 50, 0, 200); //added pz
  //TH1F *zEta = new TH1F("zEta", ";#eta;Events", 50, -3, 3);
  //TH1F *zPhi = new TH1F("zPhi", ";#phi;Events", 50, -4, 4);
  TH1F *zE = new TH1F("zE", ";Energy;Events", 50, 0, 200);	
  TH1F *zMass = new TH1F("zMass", ";Mass;Events", 50, 0, 200);	

  
	// loop over each entry (event) in the tree
  	for( int entry=0; entry < nEntries; entry++ ){
      if( entry%10000 * 0 ) cout << "Entry:" << entry << endl;
    
      // check that the event is read properly
      int entryCheck = tree->GetEntry( entry );
      if( entryCheck <= 0 ){  continue; }
      
      // only look at events containing at least 2 leptons
      if(NLooseMuons < 2) continue;
      
      // require the leptons to have some transverse momentum
      if(abs(LooseMuonsPt1) *0.001 < 20 || abs(LooseMuonsPt2) *0.001 < 20 ) continue;
      
      // make a LorentzVector from the muon
      //TLorentzVector Muons1;
     // Muons1.SetPtEtaPhiM(fabs(LooseMuonsPt1), LooseMuonsEta1, LooseMuonsPhi1, 0);
  
      // print out the details of an electron every so often
      if( entry%10000 * 0 ){ 
        cout << "Muons pt1: " << LooseMuonsPt1 << " eta: " << LooseMuonsEta1 << " phi " << LooseMuonsPhi1 << endl;
        cout << "Muons pt2: " << LooseMuonsPt2 << " eta: " << LooseMuonsEta2 << " phi " << LooseMuonsPhi2 << endl;
      }

      //calculation of muon energy
        Double_t muonMass = 0.0;  // assume the mass of the muon is negligible
        Double_t muonPx1 = abs(LooseMuonsPt1)*cos(LooseMuonsPhi1);
        Double_t muonPy1 = abs(LooseMuonsPt1)*sin(LooseMuonsPhi1);
        Double_t muonPz1 = abs(LooseMuonsPt1)*sinh(LooseMuonsEta1);
	Double_t muonEnergy1 = sqrt (muonPx1*muonPx1 + muonPy1*muonPy1 + muonPz1*muonPz1 + muonMass*muonMass);

	Double_t muonPx2 = abs(LooseMuonsPt2)*cos(LooseMuonsPhi2);
        Double_t muonPy2 = abs(LooseMuonsPt2)*sin(LooseMuonsPhi2);
        Double_t muonPz2 = abs(LooseMuonsPt2)*sinh(LooseMuonsEta2);
	Double_t muonEnergy2 = sqrt (muonPx2*muonPx2 + muonPy2*muonPy2 + muonPz2*muonPz2 + muonMass*muonMass);

	Double_t zCompX = muonPx1 + muonPx2;
        Double_t zCompY = muonPy1 + muonPy2;
        Double_t zLongi = muonPz1 + muonPz2;
        Double_t zPerp = sqrt (zCompX*zCompX + zCompY*zCompY);  
	Double_t zEnergy = muonEnergy1 + muonEnergy2;
	Double_t zM = sqrt (zEnergy*zEnergy -zCompX*zCompX -zCompY*zCompY -zLongi*zLongi);
	

      // fill our histograms
        muPt1->Fill((LooseMuonsPt1)*0.001); // in GeV
        muEta1->Fill(LooseMuonsEta1);
        muPhi1->Fill(LooseMuonsPhi1);
	muPx1->Fill( muonPx1*0.001); // in GeV
	muPy1->Fill( muonPy1*0.001); // in GeV
	muPz1->Fill( muonPz1*0.001); // in GeV
        muE1->Fill(muonEnergy1*0.001); // in GeV

	muPt2->Fill((LooseMuonsPt2)*0.001); // in GeV
        muEta2->Fill(LooseMuonsEta2);
        muPhi2->Fill(LooseMuonsPhi2);
	muPx2->Fill( muonPx2*0.001); // in GeV
	muPy2->Fill( muonPy2*0.001); // in GeV
	muPz2->Fill( muonPz2*0.001); // in GeV
        muE2->Fill(muonEnergy2*0.001); // in GeV
      
 	zPt->Fill( zPerp*0.001); // in GeV
 	zPx->Fill( zCompX*0.001); // in GeV
	zPy->Fill( zCompY*0.001); // in GeV
	zPz->Fill( zLongi*0.001); // in GeV
        zE->Fill( zEnergy*0.001); // in GeV
        zMass->Fill(zM*0.001); // in GeV
     
	}

  // draw the eta distribution
  zMass->Draw();
  
  // make a ROOT output file to store your histograms
  TFile *outFile = new TFile("histograms-z.root", "recreate");
  muPt1->Write();
  muEta1->Write();
  muPhi1->Write();
  muE1->Write();
  muPx1->Write();
  muPy1->Write();
  muPz1->Write();

  muPt2->Write();
  muEta2->Write();
  muPhi2->Write();
  muE2->Write();
  muPx2->Write();
  muPy2->Write();
  muPz2->Write();

  zPt->Write();
  zE->Write();
  zPx->Write();
  zPy->Write();
  zPz->Write();
  zMass->Write();
  
  outFile->Close();
}
```

The grid job can be submitted using:

```
$ condor_submit run-z.cmd
```

It can again be checked with: 

```
$ condor_q YOUR_USER_ID -nobatch
```

After it runs, you will find a log file that describes the job: *run-z.log*, and output file: *root-z.out*, and the files containing the simulated data: *histograms-z.root* in your test directory. 

You again need to copy that file into your public directory, so that you can download it to your local desktop:

```
$ cp histograms-z.root ~/public/
```

Go back to the local terminal window on your local desktop, and download the root files with:

```
$ wget http://stash.osgconnect.net/~YOUR_USER_ID/histograms-z.root
```

You can inspect the contents of *histograms-z.root* by running Root (i.e., `root histograms-z.root`) in your current directory in your local terminal window:

```
$ root histograms-z.root
```

And then using the Root command:  *TBrowser b*

With the *TBrowser* you can plot the variables in the root file. Double click on histograms-z.root, and then on the variables to plot them.

### Step 3: Make TSelector

Now let's go back to the files created in step 1, in the remote terminal window. Start *root* in your test directory with the following commands:

```
$ module load root
$ root -b
```

And then execute the following commands:
```
TFile f("t00.root");
t0->MakeSelector("s0","=legacy");
f.Close();
.q
```

This will create files *s0.C* and *s0.h* in your test directory that contain code corresponding to the definition of the TTree "t0". This code can be used to process files containing data is these TTree's.

Now we will add a histogram to the TSelector code. Several code lines have to be added to the TSelector code files *s0.C* and *s0.h*.

To *s0.h* make the following additions:
after existing include statements add:

```
#include "TH1F.h"
```

After class s0 definition:
```
class s0 : public TSelector {
public :
```
add
```
TH1F *e;
```

To *s0.C* make the following additions:

After entry:
```
void s0::SlaveBegin(TTree * /*tree*/)
{
```
add
```
e = new TH1F("e", "e", 1000, -199.0, 1200.0);
```

After Process entry:
```
Bool_t s0::Process(Long64_t entry)
{
```
add
```
GetEntry(entry);
e->Fill(Energy);
```

After terminate entry:
```
void s0::Terminate()
{
```
add
```
TFile f("histograms.root","RECREATE");
f.WriteObject(e,"Energy");
f.Close();
```

Now create the new script files for Step 2:

create:
*run-root-2.cmd*
```
universe=vanilla
executable=run-root-2.sh 
transfer_input_files = s0.C,s0.h,run-root-2.C,t00.root,t01.root 
transfer_executable=True 
when_to_transfer_output = ON_EXIT 
log=run-root-2.log 
transfer_output_files = root-2.out,histograms.root 
output=run-root-2.out.$(Cluster).$(Process) 
error=run-root-2.err.$(Cluster).$(Process) 
notification=Never 
queue 
```

Create *run-root-2.sh*
```
#!/bin/bash 
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
module load root
module load libXpm
root -b < run-root-2.C > root-2.out 
```

It has to be made executable, by use of the `chmod` Linux command:

```
chmod +x run-root-2.sh
```


Create *run-root-2.C*
```
.L s0.C++ 
{ 
 //Load and run TSelector 
 
  s0 *s = new s0(); 
 
  TChain tc("t0"); 
  tc.Add("t*.root"); 
  tc.Process(s); 
 
} 
```

We can test the Root job on the osgconnect training machine by issuing command:

```
root -b < run-root-2.C
```

If this works, we can process the data files *t00.root* and *t01.root* on the
Grid with our new command script *run-root-2.cmd*.

This can be done with command:

```
condor_submit run-root-2.cmd
```

Once your job has finished, you again need to copy that file into your public directory, so that you can download it to your local desktop:

```
cp histograms.root ~/public/
```

Go back to the local terminal window on your local desktop, and download the root files with:

```
wget http://stash.osgconnect.net/~YOUR_USER_ID/histograms.root
```

You can look at the output histogram file: *histograms.root*
with *TBrowser b* as before, in your local terminal window.
