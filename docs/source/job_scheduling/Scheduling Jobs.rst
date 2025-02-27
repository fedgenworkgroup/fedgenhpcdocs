**Scheduling Jobs**
-------------------------

**Anatomy of a job**
======================

Jobs are made of one or multiple **sequential steps**, each consisting
in one or multiple **parallel tasks** that will be dispatched to
possibly distinct nodes.

Each task will be allocated CPUs, memory, and possible other generic
resources in an exclusive manner by Slurm. Two jobs cannot share the
same resource unless explicitly forced by the admins, but that is
generally not the case. Therefore, jobs can only start when all needed
resources are free and not needed by another higher priority job. Jobs
are indeed assigned a **priority** when they are submitted, which can
depend upon multiple factors. 

Each job has two parts:

- Resource requests: describe the amount of computing resource (CPUs,
  memory, expected run time, etc.) that the job will need to
  successfully run.

- Job steps: describe individual tasks that must be executed into a job.
  You can execute a job step with the SLURM command srun. A job can has
  one or more steps, each consisting in one or more tasks each using one
  or more CPU, GPU, etc.


**Job Types**
================

There are a two popular types of jobs you could submit to the HPC cluster:

- `batch <#scheduling-a-batch-job>`__ which are unattended shell scripts
- `interactive <#interactive-jobs>`__ where you run and test out your workflows live interactively on the command line,


**Starting a SLURM Job**
===============================

There are two ways of starting jobs with SLURM; either interactively
with ``srun`` or as a script with ``sbatch`` commands.



**Defining and submitting A Batch job**
==========================================
For the scheduling process to work properly, you will need to describe your job before you submit it:

- what are the steps (i.e. which program must be run and how) ;
- how many tasks there will be ;
- what resource each task needs (CPU, memory, etc.), and
- for how long the job is supposed to run.

All of these, along with potentially aditionnal job parameter submission options, can be described in a **submission script**.

A submission script is a `shell
script <https://en.wikipedia.org/wiki/Shell_script>`__, e.g.
a `Bash <https://en.wikipedia.org/wiki/Bash_(Unix_shell)>`__ script,
whose comments, if they are prefixed with #SBATCH, are understood by
Slurm as directives describing resource requests and other submissions
options. You can get the complete list of parameters from the sbatch
manpage man sbatch.

.. Important::

    The #SBATCH directives must appear at the top of the submission file,
    before any other line except for the very first line which should be
    the `shebang <https://en.wikipedia.org/wiki/Shebang_(Unix)>`__ (e.g. ``#!/bin/bash`` ).
    
    The script itself is a job step. Other job steps are created with
    the ``srun`` command, that takes as argument the name and options of the
    program to run.

.. Note::

    If there is only one step, the srun command can be omitted, but it has
    consequences in how signals are handled and how accurate reporting is.
    It is often best to use it in all cases.


Below is a slurm script template to Submit a batch job from the login node by calling sbatch <script_name>.slurm.

.. code-block:: python


    $cat script.slurm

    #!/bin/bash
    
    #SBATCH --partition=debug      # partition name. Eg. Debug, bio, bigmem
    #SBATCH --job-name=demosample        # job name
    #SBATCH --nodes=2               # number of nodes allocated for this job
    #SBATCH --ntasks=2              # total number of tasks / mpi processes
    #SBATCH --cpus-per-task=8       # number OpenMP Threads per process
    #SBATCH --time=08:00:00         # total run time limit ([[D]D-]HH:MM:SS)
    ##SBATCH --gres=gpu:tesla:2      # number of GPUs
    ##The above line and this  will be ignored by Slurm
    # Get email notification when job begins, finishes or fails
    #SBATCH --mail-type=ALL         # type of notification: BEGIN, END, FAIL, ALL
    #SBATCH --mail-user=your@mail   # e-mail address
    SBATCH --chdir=<working directory>
    #SBATCH --export=all
    #SBATCH --output=<file> # where STDOUT goes
    #SBATCH --error=<file> # where STDERR goes
    
    
    # Modules to use (optional).
    #<e.g., module load singularity>
    
    # Run the application.
    #<my_programs>
    echo [`date '+%Y-%m-%d %H:%M:%S'`] Running $AE_ARCH
    srun  hostname
    sleep  60 



For instance, the following batch script, hypothetically named submit.slurm,


.. code-block:: python

    #!/bin/bash
    #
    #SBATCH --job-name=testrun
    #SBATCH --output=result.txt
    #SBATCH --partition=debug
    #
    #SBATCH --time=5:00
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=1
    #SBATCH --mem-per-cpu=200
    
    srun hostname
    srun sleep 30

describes a job made of 2+1 steps (the submission script itself plus two
explicit steps created by calling srun twice), each step consisting of
only one task that needs one CPU and 200MB of RAM. The first step will
run the ``hostname`` command, and the second one the useless ``sleep`` command.
The job is supposed to run for *5 minutes* on the debug partition, be
named testrun, and create an output file named *result.txt*.


.. Important::

    It is important to note that as the job will run unattended, it will not
    be attached to a terminal (screen) so everything that the program that
    is run would like to write to the terminal will be redirected by Slurm
    to a file, whose name is specified by the --output parameter. (Note that
    for the same reasons, the program cannot expect input from the user
    through the keyboard to run properly.)

Once the submission script is written properly, you need
to **submit** it to slurm through the ``sbatch`` command, which, upon
success, responds with the *jobid* attributed to the job. (The dollar
sign below is the `shell
prompt <https://en.wikipedia.org/wiki/Unix_shell#Bourne_shell>`__)

.. code-block:: python
  
    $ sbatch submit.slurm
    sbatch: Submitted batch job 12321

.. Warning::

  Make sure to submit the job with sbatch and not bash; also do not
  execute it directly. This would ignore all resource request and your job
  would run with minimal resources, or could possible run on the login node
  rather than on a compute node.

The job then enters the Job queue in the *PENDING* state which be verified with;

.. code-block:: python

    $ squeue --me

Once resources become available and the job has highest priority,
an **allocation** is created for it and switches to the RUNNING state. If
the job completes correctly, it goes to the *COMPLETED* state,
otherwise, it is set to the *FAILED* state.


**Slurm Arguments**
======================

These are the common and recommended arguments suggested at a minimum to
get a job in any form.

+-------+-------+-----------------------------------------------------+
| **A   | **Co  | **Notes**                                           |
| rgume | mmand |                                                     |
| nts** | Fl    |                                                     |
|       | ags** |                                                     |
+=======+=======+=====================================================+
| Ac    | -A or | What lab are you part of? If you run                |
| count |  --ac | the groups command you can see what groups (usually |
|       | count | labs) you're a member of, these are associated with |
|       |       | resource limits on the cluster.                     |
+-------+-------+-----------------------------------------------------+
| Part  | -p    | What resource partition are you interested in       |
| ition |  or - | using? This could be anything you see when you      |
|       | -part | run sinfo -s as each partition corresponds to a     |
|       | ition | class of nodes (e.g., high memory, GPU).            |
+-------+-------+-----------------------------------------------------+
| Nodes | -N    | How many nodes are these resources spread across?   |
|       | or -- | In the overwhelming number of cases this is 1 (for  |
|       | nodes | a single node) but more sophisticated multi-node    |
|       |       | jobs could be run if your code supports it.         |
+-------+-------+-----------------------------------------------------+
| Cores | -     | How many compute cores do you need? Not all codes   |
|       | c or  | can make use of multiple cores and if they do, the  |
|       | --cpu | performance of the code is not always linear with   |
|       | s-per | the resources requested. If in doubt consider       |
|       | -task | contacting the research computing team to assist in |
|       |       | this optimization.                                  |
+-------+-------+-----------------------------------------------------+
| M     | --mem | How much memory do you need for this job? This is   |
| emory |       | in the format size[units] were size is a number and |
|       |       | units are either M, G, or T for megabyte, gigabyte, |
|       |       | and terabyte respectively. Megabyte is the default  |
|       |       | unit if none is provided.                           |
+-------+-------+-----------------------------------------------------+
| Time  | -t    | What's the maximum runtime for this job? Common     |
|       |  or - | acceptable time formats                             |
|       | -time | include hours:minutes:seconds, days-hours,          |
|       |       | and minutes.                                        |
+-------+-------+-----------------------------------------------------+


**Slurm Environment Variables**
================================

When a job scheduled by Slurm begins, it needs to about how it was
scheduled, what its working directory is, who submitted the job, the
number of nodes and cores allocated to it, etc. This information is
passed to Slurm via environment variables. Additionally, these
environment variables are also used as default values by programs
like mpirun. To view a node's Slurm environment variables, use export \|
grep SLURM. A comprehensive list of the environment variables Slurm sets
for each job can be found at the end of the *sbatch man page*.



**Interactive jobs**
==========================

Slurm jobs are normally batch jobs in the sense that they are run
unattended. If you want to have a direct view on your job, for tests or
debugging, you have two options.

If you need simply to have an interactive Bash session on a compute
node, with the same environment set as the batch jobs, run the following
command:

.. code-block:: python

    srun --pty bash -l

Doing that, you are submitting a 1-CPU, default memory, default duration
job that will return a Bash prompt when it starts.

If you need more flexibility, you will need to use
the `salloc <https://slurm.schedmd.com/salloc.html>`__ command.
The salloc command accepts the same parameters as sbatch as far as
resource requirement are concerned. Once the allocation is granted, you
can use srun the same way you would in a submission script.


**Starting an interactive job**

You can run an interactive job like this:

.. code-block:: python

    $ srun --nodes=1 --ntasks-per-node=1 --time=01:00:00 --pty bash -i

Here we ask for a single core on one interactive node for one hour with
the default amount of memory. The command prompt will appear as soon as
the job starts.

This is how it looks once the interactive job starts:

.. code-block:: python

    srun: job 12345 queued **and** waiting **for** resources
    srun: job 12345 has been allocated resources

Exit the bash shell to end the job. If you exceed the time or memory
limits the job will also abort.

Interactive jobs have the same policies as normal batch jobs, there are
no extra restrictions. You should be aware that you might be sharing the
node with other users, so play nice.

Some users have experienced problems with the command, then it has
helped to specify the cpu account:

.. code-block:: python

    $ srun --account=<NAME_OF_MY_ACCOUNT> --nodes=1 --ntasks-per-node=1
    --time=01:00:00 --pty bash -i


**Interactive Jobs (Single Node)**

Resources for interactive jobs are attained either using ``salloc``. To
request a compute node from the **bio partition** (biol)
interactively consider the example below.

*# Below replace the word account with an account name you belong to*

*# Use sinfo to see your accounts and partitions*

.. code-block:: python
    salloc -A account -p bio -N 1 -c 4 --mem=12G --time=2:45:00

You are asking slurm for 4 compute cores with 12GB of memory for 2 hours and 45 minutes
spread across 1 node (single machine). The ``salloc`` command will
automatically create an interactive shell session on an allocated node.


**Interactive Jobs (Multi Node)**

Building upon the previous section, if -N or --nodes is >1 when
running ``salloc`` you are automatically placed into a shell of one of the
allocated nodes. This shell is NOT part of a Slurm task. To view the
names of the remainder of your allocated nodes use ``scontrol show
hostnames``. The ``srun`` command can be used to execute a command on all of
the allocated nodes as shown in the example session below.

.. code-block:: python

    [user@allot ~]$ salloc -N 2 -p full -A stf --time=5 --mem=5G
    salloc: Pending job allocation 2620960
    salloc: job 2620960 queued and waiting for resources
    salloc: job 2620960 has been allocated resources
    salloc: Granted job allocation 2620960
    salloc: Waiting for resource configuration
    salloc: Nodes giga[002-003] are ready for job
    
    [user@allot ~]$ srun hostname
    giga002
    giga003
    


.. note::

- If you are not allocated a session with the specified --mem value, try
  smaller memory values

For more details, read the *salloc man page*.


**Keeping interactive jobs alive**

Interactive jobs die when you disconnect from the login node either by
choice or by internet connection problems. To keep a job alive you can
use a terminal multiplexer like tmux or screen.

tmux allows you to run processes as usual in your standard bash shell

You start tmux on the login node before you get a interactive slurm
session with srun and then do all the work in it. In case of a
disconnect you simply reconnect to the login node and attach to the tmux
session again by typing:

.. code-block:: python

    tmux attach

or in case you have multiple sessions running:

.. code-block:: python

  tmux list-session
  tmux attach -t SESSION_NUMBER

As long as the tmux session is not closed or terminated (e.g. by a
server restart) your session should continue. 

To log out a tmux session without closing it you have to press CTRL-B
(that the Ctrl key and simultaneously “b”, which is the standard tmux
prefix) and then “d” (without the quotation marks). To close a session
just close the bash session with either CTRL-D or type exit. You can get
a list of all tmux commands by CTRL-B and the ? (question mark). See
also `this
page <https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/>`__ for
a short tutorial of tmux. Otherwise working inside of a tmux session is
almost the same as a normal bash session.





**Job related environment variables**

Here we list some environment variables that are defined when you run a
job script. These is not a complete list. Please consult the `SLURM documentation <https://slurm.schedmd.com/sbatch.html#SECTION_INPUT-ENVIRONMENT-VARIABLES>`_ for a complete list.


+--------------------------+--------------------------------------------------------------------------+
| Variable                 | Description                                                              |
+==========================+==========================================================================+
| $SLURM_JOB_ID            | The Job ID.                                                              |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_JOBID             | Deprecated. Same as $SLURM_JOB_ID                                        |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_SUBMIT_DIR        | The path of the job submission directory.                                |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_SUBMIT_HOST       | The hostname of the node used for job submission.                        |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_JOB_NODELIST      | Contains the definition (list) of the nodes that is assigned to the job. |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NODELIST          | Deprecated. Same as SLURM_JOB_NODELIST.                                  |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_CPUS_PER_TASK     | Number of CPUs per task.                                                 |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_CPUS_ON_NODE      | Number of CPUs on the allocated node.                                    |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_JOB_CPUS_PER_NODE | Count of processors available to the job on this node.                   |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_CPUS_PER_GPU      | Number of CPUs requested per allocated GPU.                              |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_MEM_PER_CPU       | Memory per CPU. Same as --mem-per-cpu .                                  |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_MEM_PER_GPU       | Memory per GPU.                                                          |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_MEM_PER_NODE      | Memory per node. Same as --mem .                                         |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_GPUS              | Number of GPUs requested.                                                |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NTASKS            | Same as -n, –ntasks. The number of tasks.                                |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NTASKS_PER_NODE   | Number of tasks requested per node.                                      |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NTASKS_PER_SOCKET | Number of tasks requested per socket.                                    |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NTASKS_PER_CORE   | Number of tasks requested per core.                                      |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NTASKS_PER_GPU    | Number of tasks requested per GPU.                                       |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NPROCS            | Same as -n, --ntasks. See $SLURM_NTASKS.                                 |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_NNODES            | Total number of nodes in the job’s resource allocation.                  |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_TASKS_PER_NODE    | Number of tasks to be initiated on each node.                            |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_ARRAY_JOB_ID      | Job array’s master job ID number.                                        |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_ARRAY_TASK_ID     | Job array ID (index) number.                                             |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_ARRAY_TASK_COUNT  | Total number of tasks in a job array.                                    |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_ARRAY_TASK_MAX    | Job array’s maximum ID (index) number.                                   |
+--------------------------+--------------------------------------------------------------------------+
| $SLURM_ARRAY_TASK_MIN    | Job array’s minimum ID (index) number.                                   |
+--------------------------+--------------------------------------------------------------------------+
