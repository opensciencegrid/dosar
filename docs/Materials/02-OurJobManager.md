# Our Condor Installation

## Objective

This exercise should help you understand the basics of how Condor is installed, what Condor processes (a.k.a. daemons) are running, and what they do.

## Login to the Condor submit computer
Before you start, make sure you are logged into `training.osgconnect.net`

```
$ hostname
training.osgconnect.net
```

You should have been given your name and password when you arrived this afternoon. If you don't know them, talk to Rob.

## Looking at our Condor installation

How do you know what version of Condor you are using? Try <code>condor_version</code>: 

```
$ condor_version
$CondorVersion: 8.4.11 Feb 24 2017 $
$CondorPlatform: X86_64-CentOS_6.8 $
```

Note that the "CondorPlatform" reports the type of computer we built it on, _not_ the computer we're running on. It was built on CentOS_6.8, but you might notice that we're running on Scientific Linux 6.8, which is a free clone of Red Hat Enterprise Linux.

### Extra Tip: The OS version

Do you know how to find the OS version? You can usually look in /etc/issue to find out:

```
$ cat /etc/issue
Scientific Linux release 6.9 (Carbon)
Kernel \r on an \m
```

Or you can run:

```
$ lsb_release -a
LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID:	Scientific
Description:	Scientific Linux release 6.8 (Carbon)
Release:	6.8
Codename:	Carbon
```

Where is Condor installed? 

```
# Show the location of the condor_q binary
$ which condor_q
/usr/bin/condor_q

# Show which RPM installed Condor
$ rpm -q condor
condor-8.2.10-1.1.osg32.el6.x86_64

# Show all the files installed by that RPM
$ rpm -ql condor | head -10
/etc/condor
/etc/condor/condor_config
/etc/condor/condor_ssh_to_job_sshd_config_template
/etc/condor/config.d
/etc/condor/config.d/00-restart_peaceful.config
/etc/condor/config.d/10-batch_gahp_blahp.config
/etc/condor/ganglia.d/00_default_metrics
/etc/rc.d/init.d/condor
/etc/sysconfig/condor
/usr/bin/condor_check_userlogs
```

Condor has some configuration files that it needs to find. They are in the standard location, `/etc/condor`

```
$ ls /etc/condor<
99-gratia.conf	condor_config.local			config.d      ganglia.d			 passwdfile
condor_config	condor_ssh_to_job_sshd_config_template	config.d.tgz  other_condor_config_files  passwdfile.daemon
```

Condor has some directories that it keeps records of jobs in. Remember that each submission computer keeps track of all jobs submitted to it. That's in the local directory: 

```
$ condor_config_val -v LOCAL_DIR
LOCAL_DIR = /var
 # at: /etc/condor/condor_config, line 26
 # raw: LOCAL_DIR = /var

$ ls -CF /var/lib/condor
execute/  spool/
```

The spool directory is where Condor keeps the jobs you submit, while the execute directory is where Condor keeps running jobs. Since this is a submission-only computer, it should be empty.

Check if Condor is running.  Your output will differ slightly, but you should see `condor_master` with the other Condor daemons listed under it:

```
$ ps auwx --forest | grep condor_ | grep -v grep
jtqv84    5997 50.0  0.0 238996 17012 ?        Ss   15:05   0:11          \_ /usr/bin/python /home/jtqv84/bundle_probe/condor_slot2site
jtqv84    6092 35.5  0.8 494484 405256 ?       R    15:05   0:08              \_ condor_status -pool osg-flock.grid.iu.edu -l
root     18394  0.0  0.0   9292  2104 ?        Ss   15:00   0:00  |   \_ /bin/sh -c /usr/share/gratia/common/cron_check  /etc/gratia/condor/ProbeConfig && /usr/share/gratia/condor/condor_meter -s 9
00
root     18557  0.0  0.0 104800 14284 ?        S    15:00   0:00  |       \_ /usr/bin/python /usr/share/gratia/condor/condor_meter -s 900
condor   11170  0.0  0.0  98676  4888 ?        Ss   Jun08   0:19 condor_master -pidfile /var/run/condor/condor_master.pid
root     11176  1.4  0.1  77964 58312 ?        S    Jun08 755:03  \_ condor_procd -A /var/run/condor/procd_pipe -L /var/log/condor/ProcLog -R 1000000 -S 60 -C 497
condor   11186  0.0  0.0 106884  6844 ?        Ss   Jun08  43:26  \_ condor_collector -f
condor   11223  8.4  1.4 1345192 701328 ?      Ss   Jun08 4263:55  \_ condor_schedd -f
...
```

For this version of Condor there are four processes running: the condor_master, the condor_schedd, the condor_procd, and condor_schedd. In general, you might see many different Condor processes. Here's a list of the processes:

   * *condor_master*: This program runs constantly and ensures that all other parts of Condor are running. If they hang or crash, it restarts them.
   * *condor_schedd*: If this program is running, it allows jobs to be submitted from this computer--that is, your computer is a "submit machine". This will advertise jobs to the central manager so that it knows about them. It will contact a condor_startd on other execute machines for each job that needs to be started.
   * *condor_procd:* This process helps Condor track process (from jobs) that it creates
   * *condor_collector:* This program is part of the Condor central manager. It collects information about all computers in the pool as well as which users want to run jobs. It is what normally responds to the condor_status command. At the school, it is running on a different computer, and you can figure out which one: 

```
$ condor_config_val COLLECTOR_HOST
192.170.227.195
```

Other daemons include:

   * *condor_negotiator:* This program is part of the Condor central manager. It decides what jobs should be run where. It is run on the same computer as the collector.
   * *condor_startd:* If this program is running, it allows jobs to be started up on this computer--that is, your computer is an "execute machine". This advertises your computer to the central manager so that it knows about this computer. It will start up the jobs that run.
   * *condor_shadow:* For each job that has been submitted from this computer, there is one condor_shadow running. It will watch over the job as it runs remotely. In some cases it will provide some assistance (see the standard universe later.) You may or may not see any condor_shadow processes running, depending on what is happening on the computer when you try it out. 
   * *condor_shared_port:* Used to assist Condor with networking by allowing multiple Condor processes to share a single network port. 


## condor_q

You can find out what jobs have been submitted on your computer with the condor_q command: 

```
$ condor_q
-- Submitter: login01.osgconnect.net : <192.170.227.195:49053> : login01.osgconnect.net
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
9021661.0   jtqv84          4/27 16:33   0+00:00:00 H  0   0.0  mytest.sh         
13459507.0   nlharr         11/13 14:34   0+00:00:00 I  0   0.0  subscript.sh      
19220600.0   yadunand        4/12 21:26   0+00:00:29 C  0   48.8 perl cscript497391
19336665.0   fsurf           5/7  07:45  66+17:20:16 R  0   0.0  pegasus-dagman -f 
19336666.0   fsurf           5/7  07:46  66+17:14:45 R  0   0.0  pegasus-dagman -f 

...   

2555 jobs; 34 completed, 4 removed, 1276 idle, 525 running, 716 held, 0 suspended
```

The output that you see will be different depending on what jobs are running. Notice what we can see from this:

   * *ID*: We can see each jobs cluster and process number. For the first job, the cluster is 60256 and the process is 0.
   * *OWNER*: We can see who owns the job.
   * *SUBMITTED*: We can see when the job was submitted
   * *RUN_TIME*: We can see how long the job has been running.
   * *ST*: We can see what the current state of the job is. I is idle, R is running.
   * *PRI*: We can see the priority of the job.
   * *SIZE*: We can see the memory consumption of the job.
   * *CMD*: We can see the program that is being executed. 

### Extra Tip

What else can you find out with condor_q? Try any one of:

   * `man condor_q`
   * `condor_q -help`
   * [condor_q from the online manual](http://www.cs.wisc.edu/condor/manual/v8.0/condor_q.html)

### Double bonus points

How do you use the `-constraint` or `-format` options to `condor_q`? When would you want them? When would you use the -l option? This might be an easier exercise to try once you submit some jobs.

## condor_status

You can find out what computers are in your Condor pool. (A pool is similar to a cluster, but it doesn't have the connotation that all computers are dedicated full-time to computation: some may be desktop computers owned by users.) To look, use condor_status:

```
$ condor_status
```

This will be blank on the training.osgconnect.net as it "flocks" jobs to another node for matching to the appropriate pool of resources. If there were a HTCondor pool directly attached you would see something like this:

```
Name               OpSys      Arch   State     Activity LoadAv Mem   ActvtyTime

slot1@frontal.cci. LINUX      X86_64 Claimed   Busy      0.000 1909  0+00:00:04
slot2@frontal.cci. LINUX      X86_64 Claimed   Busy      0.000 1909  0+00:00:05
slot3@frontal.cci. LINUX      X86_64 Claimed   Busy      0.000 1909  0+00:00:06
slot4@frontal.cci. LINUX      X86_64 Claimed   Busy      0.180 1909  0+00:00:07
slot1@node2.cci.uc LINUX      X86_64 Claimed   Busy      0.000  469  0+00:00:04
slot2@node2.cci.uc LINUX      X86_64 Claimed   Busy      0.000  469  0+00:00:05
slot3@node2.cci.uc LINUX      X86_64 Claimed   Busy      0.000  469  0+00:00:06
slot4@node2.cci.uc LINUX      X86_64 Claimed   Busy      0.000  469  0+00:00:07
                     Machines Owner Claimed Unclaimed Matched Preempting

        X86_64/LINUX        8     0       8         0       0          0

               Total        8     0       8         0       0          0
```

Let's look at exactly what you can see:

   * *Name*: The name of the computer. Sometimes this gets chopped off, like above.
   * *OpSys*: The operating system, though not at the granularity you may wish: It says "Linux" instead of which distribution and version of Linux.
   * *Arch*: The architecture, such as INTEL or PPC.
   * *State*: The state is often Claimed (when it is running a Condor job) or Unclaimed (when it is not running a Condor job). It can be in a few other states as well, such as Matched.
   * *Activity*: This is usually something like Busy or Idle. Sometimes you may see a computer that is Claimed, but no job has yet begun on the computer. Then it is Claimed/Idle. Hopefully this doesn't last very long.
   * *LoadAv*: The load average on the computer.
   * *Mem*: The computers memory in megabytes.
   * *ActvtyTime*: How long the computer has been doing what it's been doing. 

### Extra credit

What else can you find out with condor_status? Try any one of:

   * `man condor_status`
   * `condor_status -help`
   * [condor_status from the online manual](http://www.cs.wisc.edu/condor/manual/v8.0/condor_status.html)

Note in particular the options like `-master` and `-schedd`. When would these be useful? When would the `-l` option be useful? 

