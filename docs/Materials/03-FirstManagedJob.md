# Submitting your first Condor job

### Objective of this exercise
The object of this exercise to have you run and understand your first Condor job, as well as run small sets of jobs in a parameter sweep. This is an important exercise because it is the basis for everything that follows. If there is _anything_ you don't understand in this exercise, _please ask_ before you continue on.

Because this is an important foundation, please seriously consider doing the &ldquo;On Your Own&rdquo; section.

## First you need a job

Before you can submit a job to Condor, you need a job. We will quickly write a small program in C. If you aren't an expert C programmer, fear not. We will hold your hand throughout this process.

First, create a file called simple.c using your favorite editor. Put it anywhere you like in your home directory. In that file, put the following text. Copy and paste is a good choice: 

<pre><code>
$ mkdir -p ~/condor-test 
$ cd ~/condor-test
</code></pre>

Use your preferred text editor to create this C program. (Shown below with nano.)

<pre><code>
$ nano simple.c
</code></pre>

Paste in the following C code. 

<pre><code>
#include &lt;stdio.h&gt;

int main(int argc, char **argv)
{
    int sleep_time;
    int input;
    int failure;

    if (argc != 3) {
        printf("Usage: simple &lt;sleep-time&gt; &lt;integer&gt;\n");
        failure = 1;
    } else {
        sleep_time = atoi(argv[1]);
        input      = atoi(argv[2]);

        printf("Thinking really hard for %d seconds...\n", sleep_time);
        sleep(sleep_time);
        printf("We calculated: %d\n", input * 2);
        failure = 0;
    }
    return failure;
}
</code></pre>

Now compile that program.: 

<pre><code>
$ gcc -o simple simple.c

$ ls -lh simple
-rwxrwxr-x 1 roy roy 595K Jun 20 11:12 simple
</code></pre>

Finally, run the program and tell it to sleep for four seconds and calculate 10 * 2: 

<pre><code>
$ ./simple 4 10
Thinking really hard for 4 seconds...
We calculated: 20
</code></pre>

Great! You have a job run locally on the machine you are logged into =training.osgconnect.net=, not very impressive. But you also have a job you can tell Condor to run! Although it clearly isn't an interesting job, it models some of the aspects of a real scientific program: it takes a while to run and it does a calculation. 

Think back to the lecture. I said that our first step was to have a job to run. Now we'll work on running it in Condor, and eventually running lots of copies of it.

## Submitting your job
Now that you have a job, you just have to tell Condor to run it. Put the following text into a file called =submit=: 

<pre><code>
Universe   = vanilla
Executable = simple
Arguments  = 4 10
+ProjectName = "DataTrieste"
Log        = simple.log
Output     = simple.out
Error      = simple.error
should_transfer_files   = YES
when_to_transfer_output = ON_EXIT
Queue
<code></pre>

Let's examine each of these lines:

   * *Universe:* The vanilla universe means a plain old job. Later on, we'll encounter some special universes.
   * *Executable:* The name of your program
   * *Arguments:* These are the arguments you want. They will be the same arguments we typed above.
   * *Log:* This is the name of a file where Condor will record information about your job's execution. While it's not required, it is a really good idea to have a log.
   * *Output:* Where Condor should put the standard output from your job.
   * *Error:* Where Condor should put the standard error from your job. Our job isn't likely to have any, but we'll put it there to be safe.
   * *should_transfer_files:* Tell Condor that it should transfer files, instead of relying on a shared filesystem. While your home directories (on the glite-tutor computers) are mounted on NFS, you do not have user accounts on the worker nodes, so your jobs cannot access files on NFS. In addition, NFS isn't available between the local UI computers and the remote worker nodes. Therefore we will have Condor transfer files to the remote computer.
   * *when_to_transfer_output:* A technical detail about when files should be transported back to the computer from which you submitted your job. Don't worry about the details for now. If you're really curious, you can read [[http://www.cs.wisc.edu/condor/manual/v7.6/2_5Submitting_Job.html#sec:file-transfer][all the details in the Condor manual]].

Next, tell Condor to run your job: 

<pre><code>
$ condor_submit submit
Submitting job(s).
1 job(s) submitted to cluster 16.
</code></pre>

Now, watch your job run: 

<pre><code>
$ condor_q %UCL_USER%

-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
  33.0   kagross         8/18 15:19   0+00:00:00 I  0   0.0  simple 0 33  

1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended

$ condor_q %UCL_USER%

-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
  33.0   kagross         8/18 15:19   0+00:00:00 %RED%R%ENDCOLOR%  0   0.0  simple 0 33  

1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended

$ condor_q %UCL_USER%

-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               

0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
</code></pre>

*Tip*: if you see lots of jobs running that aren't yours (because your classmates are busy), jump ahead to [[https://twiki.grid.iu.edu/bin/view/Operations/HTCReviewCondorTips]["A few tips and tricks"]] to learn how to look at just your jobs and no others.

When my job was done, it was no longer listed. Because I told Condor to log information about my job, I can see what happened: 

<pre><code>
$ cat simple.log
000 (032.000.000) 08/18 15:18:13 Job submitted from host: <10.0.0.252:9645>
...
001 (032.000.000) 08/18 15:18:32 Job executing on host: <172.16.200.1:9250>
...
006 (032.000.000) 08/18 15:18:32 Image size of job updated: 7
	0  -  MemoryUsage of job (MB)
	0  -  ResidentSetSize of job (KB)
...
005 (032.000.000) 08/18 15:18:33 Job terminated.
	(1) Normal termination (return value 0)
		Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
	56  -  Run Bytes Sent By Job
	7059  -  Run Bytes Received By Job
	56  -  Total Bytes Sent By Job
	7059  -  Total Bytes Received By Job
	Partitionable Resources :    Usage  Request Allocated
	   Cpus                 :                 1         1
	   Disk (KB)            :       15        7  17605109
	   Memory (MB)          :        0        1      1900
</code></pre>

That looks good: the job started up quickly, though you will often see slightly slower startups. Condor doesn't optimize for fast job startup, but for high throughput, The job ran for four seconds. 

<pre><code>
$ cat simple.out
Thinking really hard for 4 seconds...
We calculated: 20
</code></pre>

Excellent! We ran our sophisticated scientific job on a Condor pool! We've only run one job though. Can we run more?

## Doing a parameter sweep
If you only ever had to run a single job, you probably wouldn't need Condor. But we would like to have our program calculate a whole set of values for different inputs. How can we do that? Let's change our submit file to look like this: 

<pre><code>
Universe   = vanilla
Executable = simple
+ProjectName = "DataTrieste"
Arguments  = 4 10
Log        = simple.%RED%$(Process)%ENDCOLOR%.log
Output     = simple.%RED%$(Process)%ENDCOLOR%.out
Error      = simple.%RED%$(Process)%ENDCOLOR%.error
should_transfer_files   = YES
when_to_transfer_output = ON_EXIT
Queue

Arguments = 4 11
Queue

Arguments = 4 12
Queue
</code></pre>

There are two important differences to notice here. First, the Log, Output and Error lines have the $(Process) macro in them. This means that the output and error files will be named according to the process number of the job. You'll see what this looks like in a moment. Second, we told Condor to run the same job an extra two times by adding extra Arguments and Queue statements. We are doing a parameter sweep on the values 10, 11, and 12. Let's see what happens: 

<pre><code>
$ condor_submit submit
Submitting job(s)...
3 job(s) submitted to cluster 18.

$ condor_q
-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
            
  34.0   kagross         8/18 15:28   0+00:00:00 I  0   0.0  simple 0 34       
  34.1   kagross         8/18 15:28   0+00:00:00 I  0   0.0  simple 1 34       
  34.2   kagross         8/18 15:28   0+00:00:00 I  0   0.0  simple 2 34        

3 jobs; 0 completed, 0 removed, 4 idle, 0 running, 0 held, 0 suspended

$ condor_q %UCL_USER%
-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
            
  34.0   kagross         8/18 15:28   0+00:00:00 R  0   0.0  simple 0 34       
  34.1   kagross         8/18 15:28   0+00:00:00 R  0   0.0  simple 1 34       
  34.2   kagross         8/18 15:28   0+00:00:00 R  0   0.0  simple 2 34     

3 jobs; 0 completed, 0 removed, 0 idle, 4 running, 0 held, 0 suspended

$ condor_q %UCL_USER%
-- Submitter: frontal.cci.ucad.sn : <10.0.0.252:9645> : frontal.cci.ucad.sn
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               

0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended

$ ls simple*out
simple.0.out  simple.1.out  simple.2.out  simple.out

$ cat simple.0.out
Thinking really hard for 4 seconds...
We calculated: 20

$ cat simple.1.out
Thinking really hard for 4 seconds...
We calculated: 22

$ cat simple.2.out
Thinking really hard for 4 seconds...
We calculated: 24
</code></pre>

Notice that we had three jobs with the same cluster number, but different process numbers. They have the same cluster number because they were all submitted from the same submit file. When the jobs ran, they created three different output files, each with the desired output.

You are now ready to submit lots of jobs! Although this example was simple, Condor has many, many options so you can get a wide variety of behaviors. You can find many of these if you look at [the documentation for condor_submit](http://www.cs.wisc.edu/condor/manual/v8.4/condor_submit.html).

## On your own

Now that you've gotten your feet wet, try a few things on your own.

### Just one log file

There's no reason to have a separate log file for each job. Change your submit file so that it uses a single log file. Does it all still work?

### New outputs for each run

You might have noticed that the output files were over-written when you re-ran the jobs. (That is, =simple.1.out= was just re-written.) That was okay for a simple exercise, but it might be very bad if you had wanted to keep around the results. Maybe you changed a parameter or rebuilt your program, and you want to compare the outputs.

Just like you used =$(Process)=, you can also use =$(Cluster)=. This will be a number from your job ID. For example, it would be 60282 from the above example. Change your submit file to use =$(Cluster)= and =$(Process)=. If you do two job submissions, will you have separate output files?

### Lots of jobs

Instead of specifying the Arguments multiple times with multiple =queue= statements, try this:

<pre><code>
Arguments = $(Process) $(Cluster)
queue 10
</code></pre>

What does it mean? What happens? Does it work as you expect?

(An aside: you might wish to be able to do math, something like =$(Process)+1=. Unfortunately, you can't do that.)

## Challenges

If you have time and feel comfortable with the technical background, try these extra challenges. You'll need to peruse the Condor manual (particularly the [manual page for condor_submit](http://www.cs.wisc.edu/condor/manual/v8.4/condor_submit.html)) to find answers. Feel free to ask Rob--he'd love to give you hints!

   * Make another scientific program (probably just modify simple.c) that takes its input from a file. Now submit 3 copies of this program where each input file is in a separate directory. Use the initialdir option [described in the manual](http://www.cs.wisc.edu/condor/manual/v8.4/condor_submit.html). This will let you specify a directory for the input to the program. You can run specify the initialdir with $(Process). You can specify extra files to copy with =transfer_input_files=. Now you're really learning the basics of running something like a real scientific job!
   * Condor can send you email when a job finishes. How can you control this? 
   * You know that your job should never run for more than four hours. If it does, then the job should be killed because there is a problem. How can you tell Condor to do this for you? 
