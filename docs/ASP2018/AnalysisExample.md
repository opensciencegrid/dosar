# ATLAS Analysis Example

## Introduction

Root may be run in batch mode on the grid to analyze large data samples. This example creates simulated data in root format using trees and performs analysis on the simulated data by means of processing on the grid. This example is based on a demo developed by OU programmer Chris Walker.

## Exercises 

### Prerequisite 

* Login on submission node

```
$ ssh -XY osguser-YOUR-NUMBER@training.osgconnect.net
```

* Make a directory for this exercise

```
$ mkdir -p analysis_example
$ cd analysis_example
```

### Simple Analysis Example

#### Step 1: Create simulated data using the grid

Now in your test directory on the submission host we will create the three files: *run-root.cmd*, *run-root.sh*, and *run-root.C* with the contents
given below. This may require running an editor such as *emacs* on your local desktop and then copying the created files to the submission host. Or the *nano* editor can be run directly on the submission host. A
typical copy command would be as follows. 

```
scp run-root.* osguser-YOUR-NUMBER@training.osgconnect.net:analysis_example/
```

It is probably easier to create all scripts with *nano* on the submission node, though, and then you won't have to copy (*scp*) anything at all. So everything below assumes you are logged on to a terminal session on the submission node.

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
It has to be made executable, by use of the *chmod* Linux command (protections can be checked with the command *ls -l*):

```
chmod +x run-root.sh
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
condor_submit run-root.cmd
```

It can be checked with: 

```
condor_q osguser-YOUR-NUMBER
```

After it runs, you will find a log file that describes the job: *run-root.log*, and output file: *root.out*, and the files containing the simulated data: *t00.root*, *t01.root* in your test directory. 
You need to copy these files into your public directory, so that you can download it to your local desktop:

```
cp t0*.root ~/public/
```

Now open a different terminal window on your local desktop, and download the root files with:

```
wget http://stash.osgconnect.net/~osguser-YOUR-NUMBER/t00.root  http://stash.osgconnect.net/~osguser-YOUR-NUMBER/t01.root
```

You can then inspect the contents of *t00.root* and *t01.root* by running Root in your current directory in the local terminal window in which you just ran the *wget* command:

```
root t00.root
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

When you are done with this, you can quit *root* again with the command *.q <Return>*.
