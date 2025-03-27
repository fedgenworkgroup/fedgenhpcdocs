**Connect to the Cluster**
--------------------------

Once you have received an email confirming the creation of your
**Fedgen-UserID**, you have access to the FEDGEN HPC Cluster.

To begin using the cluster, you need to be aware of the following;

1.  You need to **log on to the cluster** using an SSH client (Windows
    Powershell, Windows CMD, Putty, MacOS Terminal, Linux Terminal,
    MobaXterm etc) to the login node ``allot.hpc.fedgen.net`` which will
    give you command-line access. 

2.  You can connect to the cluster only through the login node over the
    internet or on Covenant University Campus Network.

3.  When you login, you will be on your **$HOME** Directory on the login
    node. Do not run any long-lasting programs here. The login node
    shall only be used for job preparation and simple file operations.

4.  Before you can do some work, you'll have to **transfer the
    files** that you need from your computer to the cluster. At the end
    of a job, you might have to transfer your output files back.

5.  To move files from your computer to the cluster or vice versa, you
    may use any tool that works with ssh. On Linux and OSX, these are
    scp, rsync, or similar programs. On Windows, you may use
    WinSCP. 

    see `Transferring files to/from FEDGEN HPC Cluster <../others/Transferring%20files%20to%20and%20from%20the%20clusters.rst>`_

6.  Optionally, if you wish to use programs with a **graphical user
    interface (GUI)**, you will need an X-server on your client system and log
    in to the login nodes with X-forwarding enabled.
.. Note::
    This is still in the works

7.  Usually several versions of **software packages and libraries** are
    installed, so you need to select the ones you need. To manage
    different versions efficiently, the FEDGEN HPC Cluster use
    so-called **modules**, so you will need to select and load the
    modules that you require for your work. e.g. ``Module load Matlab``

    More information on modules `here <https://hpcdocs.fedgen.net/en/latest/job_scheduling/Applications.html#the-module-command>`_

8.  To eventually run the program, you have to write a job script. In
    this script, you can define how long the job (i.e. the program) will
    run and how much memory and compute cores it needs. For the actual
    computation, you need to learn at least the basics of Linux shell
    scripting. You can learn some basics here: `Linux command line <https://hpcdocs.fedgen.net/en/latest/others/Linux%20command%20line.html>`_.

9.  After writing the job script, you execute it
    with ``sbatch jobscript.sh``. This will put the script in the Job queue,
    where it will wait until an appropriate compute node with the
    requested resource is available. You can see the status of your job
    with ``squeue -u username``. 
    
    `See Batch system and Job script examples <https://hpcdocs.fedgen.net/en/latest/job_scheduling/Job%20script%20examples.html>`_

10. Understand that the Cluster is a Shared Resource therefore utmost
    responsibility is expected of each user. You are also responsible
    for the efficient management of Quota Resources assigned to you. You
    can see more in the `Policy <http://policy>`_.

See the following section for details on using ssh to access the
cluster.

    - `Working With An SSH Client <https://fedgenhpc.readthedocs.io/en/latest/access/Working%20With%20An%20SSH%20Client.html>`__
    - `Introduction to MobaXterm <https://fedgenhpc.readthedocs.io/en/latest/access/Introduction%20to%20MobaXterm.html>`__
