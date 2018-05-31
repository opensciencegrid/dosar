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

