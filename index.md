---
layout: page
title: Parallel Programming with Python
---

Modern Computers systems are often "parallel", i.e. they have multiple processing units that can be taught to work together on a single application. Parallel computers range from multicore cell phones to huge clusters with hundreds of thousands of processors. Programming for such machines can be a daunting task. But a few basic principles can get you surprisingly far.

In this course we would like to give you a basic introduction into how to go about writing a parallel program, and then run it on a small parallel system. We're using a system called "MPI" (Message Passing Interface) for this. There's a few reasons why we're using MPI to illustrate parallel programming:

1.  MPI works on any kind of parallel computer, irrespective of the details of the architecture;
2.  It is very explicit, i.e. everything to do with parallel processing has to be "spelled out";
3.  Python bindings are available, so we can build on what we learned before; and
4.  Although it is large and can get very complex, there's a lot of simlifications we can make, and we get some good parallelism out of a handful of function calls.

Note that we won't be able to make a parallel programmer out of you in a few hours. We're just using this opportunity to give you an idea of what's what. We encourage you to join the one or other of the "High-Performance Computing" courses that we offer, where we can go into more detail.

The approach we are taking here is to work towards taking our "Mandelbrot Set" program that we developed in the Programming in Python session, and "parallelize" it by letting multiple processors do parts of the overall work simultaneously.

Ont the way, we will earn a few techniques that are common when programming for clusters and other parallel systems, and test them out with sample programs.

> ## Note {.prereq}
>
> Unfortunately, this part of the workshops is a less hands-on than others. This is unavoidable: because we are going to introduce 
> a lot of new concepts before we are ready to apply them, there's going to be a lot more talking and slides. We're trying 
> to put the one or other exercise in there.
>
> The stuff we did in the "Programming with Python course" is pretty much a prerequisit for this. We've got a version of the program we ended up with for you, so you won't have to "start from scratch". For the first few lessons we will use pre-made programs to demonstrate things.

> ## Getting ready {.getready}
>
> All the required data and programs can be found in your home directory on your account
>
> There is a folder (directory) parallel in your home directory.
> You can access this folder from the Unix shell with:
>
> ~~~ {.input}
> $ cd python-parallel
> ~~~
>
> Once you're there you're ready.

## Topics

1.  [Parallel Computing](01-parallel.md)
2.  [Shared Memory and Distributed Memory](02-smdm.md)
3.  [Multicore Machines and Multithreading](03-multicore.md)
4.  [Clusters and Message Passing](04-clusters.md)
5.  [MPI](05-mpi.md)
6.  [Back to Python: MPI4Py](06-mpi4py.md)
7.  [All Together Now: Hello World](07-hello.md)
8.  [Let's Talk: Point-To-Point](08-p2p.md)
9.  [Broadcast and Reduce: Sum of Square Roots](09-rootsum.md)
10. [The Return of Mandelbrot (in parallel)](10-mandel.md)
11. [Running Parallel Programs on HPC Systems](11-hpc.md)
12. [More MPI: The Master Slave Model](12-msm.md)
13. [Memory Distribution: Multiplying matrices](13-matmul.md)

## Other Resources

*   [Reference](reference.md)
