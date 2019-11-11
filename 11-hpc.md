---
layout: page
title: Parallel Computing
subtitle: Using HPC Systems
minutes: 30
---
> ## Learning Objectives {.objectives}
>
> *   Setting up a runtime environment through lmod.
> *   Use the SLURM scheduler to submit, monitor, and kill a job.
> *   Setup and run a parallel job on an HPC System.

On the first day of this workshop, you may already have had a look at the package management system "lmod". This is a setup system that allows you to specify software packages that you want to use and then automatically sets environment variables etc. Let's have another look.

~~~ {.bash}
$ module avail 
~~~

~~~ {.output}
$ module avail

--------------------------- /global/software/lmod/modules ---------------------------
   abaqus/2017                         (phys)
   adf/2017_108                        (chem)
   afni/17.3.05                        (bio)
   agouti/v0.3.3
   allpaths-lg/52488                   (bio)
   anaconda/2.7.13
   anaconda/3.5.3                      (D)
   ansys/ansys181                      (phys)
   ansys/ansys193                      (phys)
   ansys/ext181                        (phys,D)
   basespace-cli/0.9.9.613
   bayescan/g++540                     (bio)
   caffe/1.0                           (ai)
   chapel/1.15.0                       (t)
   [...]
~~~

The list you get as an answer is very long and includes a lot of stuff we are not interested in. What we want is stuff to do with Python :

~~~ {.bash}
$ module avail python
~~~

~~~ {.output}
--------------------------- MPI-dependent avx2 modules -----------------------------
   python27-mpi4py/2.0.0 (t)    python35-mpi4py/2.0.0 (t)

-------------------------- Compiler-dependent avx2 modules --------------------------
   python27-scipy-stack/2017a (math)    python35-scipy-stack/2017a (math)

----------------------------------- Core Modules ------------------------------------
   python/2.7.14 (t,2:2.7)    python/3.6.3 (t,3:3.6)    python/3.7.4 (t)
   python/3.5.4  (t,D:3.5)    python/3.7.0 (t)

  Where:
   math:     Mathematical libraries / Bibliothèques mathématiques
   t:        Tools for development / Outils de développement
   Aliases:  Aliases exist: foo/1.2.3 (1.2) means that "module load foo/1.2" will load foo/1.2.3
   D:        Default Module

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching any
of the "keys".
~~~

What we need is the python distribution that has the mpi4py (MPI for Python) built into it, i.e. python35-mpi4py/2.0.0. We can get access to this by the load command:

~~~ {.bash}
$ module load python35-mpi4py/2.0.0
~~~

~~~ {.Output}
--------------------------------------------------------------------------------
There are messages associated with the following module(s):
--------------------------------------------------------------------------------
~~~

followed by some warnings about deprecated packages. We can safely ignore those as the package we are using is sufficient for our purposes.

We can check what we are using after loading this python version:

~~~ {.bash}
$ which python
~~~

~~~ {.output}
/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/python-3.5.2/bin/python
~~~

3.5 is the version, which we can check:

~~~ {.bash}
$ python --version
~~~

~~~ {.output}
Python 3.5.2
~~~

Let's check for MPI

~~~{.bash}
$ which mpirun
~~~

~~~ {.output}
/cvmfs/soft.computecanada.ca/easybuild/software/2017/avx2/Compiler/intel2016.4/openmpi/2.1.1/bin/mpirun
~~~

We can save ourselves this kind of typing by including the "module load" command in the setup file ".bash_profile".

~~~ {.bash}
more .bash_profile
~~~

~~~ {.output}
if [ -r ~/.bashrc ]
then
  . ~/.bashrc
fi

module load python35-mpi4py/2.0.0
~~~

There may be other settings in there, for instance to give you a nice prompt, adjust stack sizes (nevermind), and check for another setup file called ".bashrc".

Now let's see if we can get our parallel version of the Mandelbrot program running on the production nodes of this HPC system. This means we are submitting a parallel job through the scheduler. From the yesterday, you may remember that the ticket system's name is "SLURM". We will have to alter our submission script from yesterday somewhat to accomodate the parallel nature of our program.

Let's look at the serial script

~~~ {.bash}
more serial.sh
~~~
~~~ {.output}
#SBATCH --job-name=MPI_test 
#SBATCH --mail-type=ALL 
#SBATCH --mail-user=joe.user@email.ca 
#SBATCH --output=STD.out 
#SBATCH --error=STD.err 
#SBATCH --time=30:00 
#SBATCH --mem=1G 

./program
~~~

This is a "bare bones" including a job name, email notification at the beginning and end and specifying standard output and standard error. Let's try to run a serial job by modifying this and use it to submit the (serial) Mandelbrot program.

> ~~~ {.challenge}
> Locate the serial.sh submission script, make a copy of it.
> Alter the copy (mandel-serial.sh, for instance) to add options such as
> account, partition, and reservation.
> Finally, alter the command line to run the serial version
> of the Mandelbrot program.
> Submit the script to the cluster and check out the resulting outputs.
> ~~~

To turn this into a parallel script, we need to insert a few lines specifying the "parallel environment":

~~~ {.bash}
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --cpus-per-task=1
~~~

The --nodes option tells the system how many physical compute nodes to use. For jobs that run more processes than a single node has CPU's (or "cores"), this will have to be chosen accordingly. For our purposes, one is enough.

The number of processes we want to start in our parallel run is specified through the --ntasks option (task being another word for process). Finally we need to tell the scheduler how many CPU's to assign to each process. This is in case we are doing some additional "multi-threading", whcih we don't. So one is good here as well.

Note that introducing this parallel environment allows us to specify how many processes we want to "reserve" for our job. The scheduler will go an search for a node on our cluster that has enough free CPUs to accomodate 8 processes on the same node. When it finds that, it will mark those 8 CPUs as "busy" and send the job there. It is important to remember that this does not tell our program to run with 8 processes. It just tells the scheduler to reserve 8.

To specify that we want to use 8 processes, we use the last line of our script.

~~~ {.bash}
mpirun -np $SLURM_NTASKS ./mpi-program
~~~

The puirpose of the $SLURM_NTASKS environment variable is to let the mpirun command know how many processes to start. This variablke is automatically set to the number we specified in the --ntasks line. So we only have to specify this once in that line and there is no risk conflicts:

* Specifying more processes in the --ntasks line than in the command line is unnecessarily reserving CPU's that cannot be used by others, increasing waiting times for everyone.
* Specifying less in the -pe line than in the comand line causes oversubscription: you ask for 4 and use 20, but the scheduler does not know about it and keeps pushing more jobs to a node that's already full. That slows down execution on the affected nodes and can make things very slow. Particularly if you don't use a parallel environment altogether, the scheduler thinks you are running a serial job, makes space for one processor and runs however many you are telling it to.

> ## Some jobs want it all {.callout}
> A common problem with the submission of jobs is that the user doesn't really know how many processes the software is using.
> Some software packages are designed with a desktop machine in mind, and will detect how many CPUs are present or available in the node
> that the software runs on. Typically, the user will ask for only one "slot", i.e. reserved CPU. The software then goes ahead and runs
> a dozen or more processes, leading to overloading. In such situations, you have to find a way to restrict the software to use only as many CPU's as
> you have asked for in the -pe line. THat may be tricky but it's almost always possible. If all else fails you have to ask for all the processes of the node
> because that's what you are using.

So now we have this script (don't forget to use a real email address):

~~~ {.bash}
more parallel.sh
~~~
~~~ {.output}
#!/bin/bash
#SBATCH --job-name=MPI_test
#SBATCH --mail-type=ALL
#SBATCH --mail-user=joe.user@email.ca
#SBATCH --output=STD.out
#SBATCH --error=STD.err
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --cpus-per-task=1
#SBATCH --time=30:00
#SBATCH --mem=1G

mpirun -np $SLURM_NTASKS ./MPImandel.py 1000 1000 -1.5 0.5 -1.0 1.0 1000
~~~

Before we submit this with the MPImandel.py program we should remove lines like:

~~~ {.python}
#    image=plt.imshow(mandel)
#    plt.show(image)
~~~

The reason for this is that we are submitting our job to a cluster and we have no graphical interface anymore. This will cause problems if we are attempting to plot an image. Where would it go ? To avoid this, we just skip the last little bit, i.e. displaying the image. We could instead dump the results out as a csv file.

Now we're ready to submit:

~~~ {.bash}
$ sbatch parallel.sh
~~~
~~~ {.output}
~~~

We can monitor what happens to our job after submission through the squeue command:

~~~ {.bash}
hasch@caclogin02$ squeue -u hasch
~~~
~~~ {.output}
JOBID          PARTITION  NAME            USER     ST TIME       CPUS MIN_MEMORY NODELIST(REASON)
4121621        reserved   serial_test     hasch    PD 0:00       1    1G         (None)
~~~
~~~ {.bash}
hasch@caclogin02$ squeue -u hasch
~~~
~~~ {.output}
JOBID          PARTITION  NAME            USER     ST TIME       CPUS MIN_MEMORY NODELIST(REASON)
4121621        reserved   serial_test     hasch    R  0:05       1    1G         cac076
~~~
~~~ {.bash}
hasch@caclogin02$ squeue -u hasch
~~~
~~~ {.output}
JOBID          PARTITION  NAME            USER     ST TIME       CPUS MIN_MEMORY NODELIST(REASON)
~~~

We may find when using squeue that the job is in different states, for instance "PD" or "R" for "pending" and "running", respectively. If it's the former, it will give us a reason why it's still in waiting. If it's the latter, it tells us the name of the node(s) the job is running on.

If we don't get any response from the squeue command anymore the job's done, and we can check the output and error files STD.out and STD.err, respectively.

~~~ {.bash}
$ more STD.out
~~~
~~~ {.output}
Done in  72.80901670455933 seconds.
[[2. 2. 2. ... 2. 2. 2.]
 [2. 2. 2. ... 2. 2. 2.]
 [2. 2. 3. ... 3. 3. 2.]
 ...
 [2. 3. 3. ... 3. 3. 3.]
 [2. 2. 3. ... 3. 3. 2.]
 [2. 2. 2. ... 2. 2. 2.]]
~~~

~~~ {.bash}
$ more STD.err
~~~
~~~ {.output}
-------------------------------------------------------------------------------
There are messages associated with the following module(s):
-------------------------------------------------------------------------------
~~~
followed by the afore-mentioned deprecation warnings.

>~~~ {.challenge}
> Run the parallel script several times with various setting of
> the --ntasks variable and note down the results.
> Check how well (or badly) the MPImandel program scales.
> See what happens if you ask for more processes than the nodes
> that were reserved for this course have. See if you can bypass this issue.
>~~~

> ## Same thing every few seconds {.callout}
> Instead of typing "squeue" every few seconds to check if you job's still runnning you can do "watch squeue" on some systems then it will repaeat that 
> command automatically until you stop it with "Control-C". This practice may be discouraged by the sysadmin, as it is keeping the scheduler too busy.

So that's it. Next step is to learn some Fortran to get stuff done really fast ...

