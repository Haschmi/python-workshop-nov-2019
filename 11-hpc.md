---
layout: page
title: Parallel Computing
subtitle: Using HPC Systems
minutes: 20
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

To turn this into a parallel script, we need to insert a line specifying a "parallel environment":

~~~ {.bash}
#$ -pe shm.pe 4
~~~

right before the bottom line that calls the program. "shm.pe" is a shared-memory parallel environment on our system. A parallel environment for the scheduler means a group of nodes that a job can be scheduled to, together with a configuration on how they can be scheduled. In the the case of shm.pe, the configuration forces all the processes that we run to be on the same node. This is required for shared-memory programs, because memory cannot be shared across nodes. The program we want to run (MPImandel.py) is an MPI program, and could run on a cluster parallel environment (i.e. one that allows the processes to spread across nodes) as well, but we're sticking with "shm.pe" to make our life easier.

Note that introducing this parallel environment allows us to specify how many processes we want to "reserve" for our job. The scheduler willl go an search for a node on our cluster that has enough free CPUs to accomodate 4 processes on the same node. When it finds that, it will mark those 4 CPUs as "busy" and send the job there. It is important to remember that this does not tell our program to run with 4 processes. It just tells the scheduler to reserve 4.

To specify that we want to use 4 processes, we use the last line of our script.

~~~ {.bash}
mpirun -np $NSLOTS ./MPImandel.py
~~~

As before, when we used mpirun to kick off MPImandel with a number of processes, we use the -np option. But what's the "$NSLOTS" business ?
NSLOTS is a Grid Engine internal variable that gets set through the -pe (parallel environment) option. So $NSLOTS (with the dollar sign) means "the value specified in the -pe option". If we use this, we have made sure that we are running the program with the same number of processes that we have requested and that are reserved for us. It is important to do it this way, so we specify the number of processes only once in the -pe line. If we are specifying it twice, once in the -pe line that reserves the processes, and once in the mpirun line that runs them, we may introduce a difference, and that's a problem:

* Specifying more processes in the -pe line than in the command line is unnecessarily reserving CPU's that cannot be used by others, increasing waiting times for everyone.
* Specifying less in the -pe line than in the comand line causes oversubscription: you ask for 4 and use 20, but the scheduler does not know about it and keeps pushing more jobs to a node that's already full. That slows down execution on the affected nodes and can be very bad. Particularly if you don't use a parallel environment altogether, the scheduler thinks you are running a serial job, makes space for one processor and runs however many you are telling it to.

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
#$ -S /bin/bash
#$ -q abaqus.q
#$ -l qname=abaqus.q
#$ -V
#$ -cwd
#$ -M my.email@here.com
#$ -m be
#$ -o STD.out
#$ -e STD.err
#$ -pr shm.pe 4
mpirun -np $NSLOTS ./MPImandel.py 1000 1000 -1.5 0.5 -1.0 1.0 1000
~~~

Before we submit it we should remove (or comment out) the last two lines of out MPImandel.py program:

~~~ {.python}
#    image=plt.imshow(mandel)
#    plt.show(image)
~~~

The reason for this is that we are submitting our job to a cluster and we have no graphicl interface anymore. This will cause problems if we are attempting to plot an image. Where would it go ? To avoid this, we just skip the last little bit, i.e. displaying the image. We could instead dump the results out as a csv file.

Now we're ready to submit:

~~~ {.bash}
$ qsub parallel.sh
~~~
~~~ {.output}
Your job 1056851 ("parallel.sh") has been submitted
~~~
~~~ {.bash}
$ qstat
~~~
~~~ {.output}
job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
-----------------------------------------------------------------------------------------------------------------
1056851 0.00000 parallel.s hasch        qw    02/09/2016 12:03:22                                    4
~~~
~~~ {.bash}
$ qstat
~~~
~~~ {.output}
job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
-----------------------------------------------------------------------------------------------------------------
1056851 0.50600 parallel.s hasch        r     02/09/2016 12:03:31 abaqus.q@sw0008.hpcvl.org          4
~~~
~~~ {.bash}
$ more STD.out
~~~
~~~ {.output}
Done in  24.06234097480774 seconds.
~~~
~~~ {.bash}
$ more STD.err
~~~
~~~ {.output}
-bash: module: line 1: syntax error: unexpected end of file
-bash: error importing function definition for `BASH_FUNC_module'
~~~

The first time we use qstat to check our job we see a state "qw" which means it's in the queue waiting. Next time we check it has moved on to "r" and we can see from the "queue" column how it's moved on to the node sw0008. If we don't get any responce from the qstat command anymore the job's done, and we can check the output and error files STD.out and STD.err, respectively. The first informs us that it took about 24 seconds to do this with 4 processes. The second gives us some strange error messagess. It turns out that those are an issue with the shell setup on that cluster that does not have an impact on job execution, so we ignore it.

Let's edit the -pe line one more time and run this with only 2 processes:

~~~ {.bash}
#$ -pe shm.pe 2
~~~

After running the whole thing again we get:

~~~ {.bash}
$ more STD.out
~~~
~~~ {.output}
Done in  24.06234097480774 seconds.
Done in  51.13189196586609 seconds.
~~~

It appears if we don't remove STD.out the output will just get attached, so the old 24 seconds are still there. This one took more than twice as long, so scaling with this few processes is still excellent.

> ## Same thing every few seconds {.callout}
> Instead of typing "qstat" every few seconds to check if you job's still runnning you can do "watch qstat" on some systems then it will repaeat that 
> command automatically until you stop it with "Control-C". 

So that's it. Next step is to learn some Fortran to get stuff done really fast ...

