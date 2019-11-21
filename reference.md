---
layout: page
title: Programming with Python
subtitle: Reference
---
## [Parallel Computing](01-parallel.md)

*   Moore's Law: Computers capability increases exponentially.
*   Parallel Computing: Split up tasks into independant subtasks and perform them on separate hardware simultaneously.
*   Running "multithreaded" applications: set environment variable, then run program like a serial one.
*   Speedup: Ratio of time in serial and time in parallel run. This is a function of the number of processors.
*   Scaling: Ideal scaling if the speedup is equal to the number of processes, i.e. twice the processes, half the time.
*   Amdahls Law: Limitations to scaling when not everything runs in parallel.
*   Weak Scaling: Twice the processes, twice the work in the same time.
*   Strong Scaling: Twice the processes, same work in half the time.
*   Load balancing: Make sure all processes do the same amount of work.

## [Shared- and Distributed-Memory Machines](02-smdm.html)

*   Shared-memory: all processors attached to all memory.
*   Ranges from SMP (Symmetric Multi-Processor) to NUMA (Non-Uniform Memory Access), depending on how easily memory is reached.
*   Box full of CPU's.
*   Distributed Memory: clusters, each CPU has got it's own memory, all of them are connected.
*   Interconnect determines how good the cluster is (ethernet, Infiniband (optical))
*   Room full of computers.
*   SM machines are fast, easy to use, versatile, but expensive and limited.
*   DM clusters are cheap, expandable, and sometimes huge, but need to communicate and are hard to program for.
*   Modern clusters are usually a combination.

## [Multicore machines and Multithreading](03-multicore.md)

*   There's been a multicore revolution and now everything's a shared-memory machine (cellphone, laptop, washing machine).
*   Multithreading: start with single process, create new ones (lightweight=threads) when needed (dynamically).
*   Automatic parallelization: Leave it to the compiler, only an option required. Works only with very simple loops.
*   Compiler Directives: OpenMP. Local compiler options are written into code to guide compiler. Simple at expense of flexibility, used for parallelism.
*   Set environment variable OMP_NUM_THREADS=N and run program like a serial one.
*   Thread Libraries: for instance Posix. Full-fledged API, hard to use but powerful and flexible. 


## [Clusters and Message Passing](04-clusters.md)

*   If you really need it parallel, go on a cluster.
*   Multiprocessing: start full program N times, and have processes (heavy-weight, standalone) communicate (message passing, MPI).
*   No automatic parallelization, no compiler directive, everything's explicit.
*   Needs external "Runtime environment" to start program N times: mpirun -np N program_name
*   mpirun does a lot more if needed (what-goes-where, which communication system)

## [Message Passing Interface (MPI)](05-mpi.md)

*   Serial code: one process, one CPU. Parallel code: multiple processes, multiple CPUs.
*   Numbers are often not the same. Often user decides how many processes, system decides what goes where.
*   Rank: Unique number for the process (0,...N-1).
*   Size: Total number of processes.
*   Rank does not determine order in time (siumltaneous and asynchroneous).
*   Initialization/Finalization: Required for languages like C and Fortran (MPI_Init and MPI_Finalize). Not necessary with Python.
*   Communicator: group of processors and a communication system. Rank and Size are specific to communicators. Must be specified.
*   Multiple communicators can overlap, i.e. same process can belong to multiple communicators.
*   Default communicator: MPI_COMM_WORLD (contains all processes)
*   Point-To-Point Communication: One process talks with another one.
*   Requires MPI_Send and MPI_Recv, specification of source, target, message (type and dimension), and tag
*   Tag is used to distinuish between multiple messages between the same processes.
*   Blocking: get out of routine when everything's done; Nonblocking: Initiate operation, move on to other things.
*   Blocking is safe and the default, but may slow things down, non-blocking requires additional checks, but speeds things up.
*   Collective communication: involves all processes, all must call the routine. Simple and fast, albeit not as flexible as P2P.
*   MPI_Bcast: One process has the data, sends it to others.
*   Root process: That's the one who has the data, must be specified.
*   MPI_Reduce: All processes have pieces, combine them through operation, result is on one process.
*   Root process: The one who has the result at the end, specify. Operation, e.g. MPI_SUM for summation, specify.
*   MPI_Barrier: All processes must call, none may move on before all have called. Used to sync processes.

## MPI for Python (MPI4Py)](06-mpi4py.md)

*   Many MPI calls are simplified when used with suitable data (for instance, Numpy arrays).
*   Python can recognize type and shape, so no need to specify.
*   No initialization, just "from mpi4py import MPI", no finalization either (just let it run to end).
*   Commuicator specified by direct setting: comm=MPI_COMM_WORLD.
*   Rank: comm.Get_rank(). Size: comm.Get_size()
*   Blocking Send: comm.Send(data, dest=drank, tag=itag). Blocking Receive: comm.Recv(data, source=srank, tag=itag)
*   Broadcast (for numpy arrays): comm.Bcast(data, root=rrank)
*   Reduce (numpy arrays): comm.Reduce(pdata, tdata, op=MPI.op, root=rrank) 
*   Barrier: Comm.Barrier()

## [Hello Worlds](07-hello.md)

*   Header required so we can run python program with mpirun: #!/usr/bin/env python3
*   Running simple test program: mpirun -np 8 ./hello.py
*   Some system issue error message if too many processes are used (more than are on the machine or cluster).

## [Point-to-point Communication](08-p2p.md)

*   P2P often requires loops over all "other" processes (ranks).
*   This may cause serialization: careful, communication is expensive and must be minimized.
*   Use if statements to distinguish between sender and receiver: if (rank.eq.0) ... else ...
*   Rule of thumb: Rare communication of large data is better than frequent communication of little data (latency, overhead).
*   Use postconditions to check that the output from a function is safe to use.
*   P2P is flexible, good for complicated stuff. For simple cases, use collective communication (robust, simple to use).

## [Sum of Squareroots - Broadcast and Reduce](09-rootsum.md)

*   Broadcast often used after reading in inital data to set up all processes. Alternative: MPI_Scatter
*   Reduce often used to collect results (condensed). Alternative: MPI_Gather
*   Warning: Python is much slower than C or Fortran, so parallelization doesn't really help all that much.

## [Parallelizing the Mandelbrot Program](10-mandel.md)

*   Turn serial program parallel stepwise.
*   Import MPI package
*   Set a communicator
*   Determine rank and size
*   Use if/else to split stuff that needs to be done by one process (e.g. I/O) from stuff all of them do (computation of partial tasks).
*   Insert timing routines to check scaling: start=tm.time() ... end=tm.time() ... time_elapsed=end-start

## [Running parallel programs on HPC systems](11-hpc.md)

*   Setting up a runtime environment through lmod.
*   Use the SLURM scheduler to submit, monitor, and kill a job.
*   Setup and run a parallel job on an HPC System.

## [More MPI: The Master Slave Model](12-msm.md)

* Understanding workload imbalances
* Using point-to-point communication to distribute workload dynamically
* Understanding the basic principle of the MSM model
* Applying the MSM model to the Mandelbrot set

## [Memory Distribution: Multiplying Matrices](13-matmul.md)

* Understanding the importance of memory distribution
* Learning about different methods of distributing memory
* Learning the application of Scatter/Gather for collective MD
* Tracking timing and memory usage of programs

## Glossary

[...some other time...]
