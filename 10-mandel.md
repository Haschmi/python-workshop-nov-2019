---
layout: page
title: Parallel Computing
subtitle: Parallelizing the Mandelbrot Program
minutes: 30
---
> ## Learning Objectives {.objectives}
>
> * Using a serial program as a template for a parallel one.
> * Stepwise introduction of parallel framework.
> * Parallelizing Loops.

OK, now we made multiple processes, made them talk to each other through point-to-point communication, so we're ready to try to exploit this to speed up our Mandelbrot program. At this point, that should look something like this:

~~~ {.python}
#! /usr/bin/env python3

import cmath
import matplotlib.pyplot as plt
import numpy as np
import sys

nr=   int(sys.argv[1])
ni=   int(sys.argv[2])
rei=float(sys.argv[3])
ref=float(sys.argv[4])
imi=float(sys.argv[5])
imf=float(sys.argv[6])
imax= int(sys.argv[7])
dr=(ref-rei)/nr
di=(imf-imi)/ni

def point (ir,ii):
    rl=ir*dr+rei
    im=ii*di+imi
    return rl+im*1j

def mandit (b):
    z=0.0
    for it in range(1,imax+1):
        z=z*z+b
        if (abs(z)>2.0):
            break
    return it

manrow=np.zeros(nr)
mandel=np.zeros((nr,ni))
for iindex in range(0,ni):
    for rindex in range(0,nr):
        b=point(rindex,iindex)
        manrow[rindex]=mandit(b)
    mandel[iindex][:]=manrow[:]

image=plt.imshow(mandel)
plt.show(image)
~~~

If it doesn't, no worries. There is a hidden version of the program that looks exactly like this in your "mandel" directory and you can copy it to your current mandel program like this:

~~~ {.python}
cp .mandel_serial MPImandel.py
~~~

Then edit MPImandel.py. We have various segments in the code:

* Hash-bang header to tell the system we're using Python3
* Importing various packages
* Taking dimensions etc from the input line
* Little routine "point" to convert index numbers into real and imaginary part
* Other routine "mandel" to performm and count iterations
* Main loops going through every pixel row-wise
* Calls to plt package to make and display image

To turn this parallel, we need to:

* Import the MPI package
* Set a communicator
* Determine rank and size
* Insert if/else statements to separate what needs to be done by only one process (rank 0 prints and plots stuff and handles the total of the results) from what needs to be done by all of them (computing the proper share of actual iterations).
* Insert MPI function calls to send results from rank>0 processes to rank 0 process
* Throw in timing routine so we can check if this scales.
	
OK, let's get to it. First the simple stuff. To the import list on the top of the program we add:

~~~ {.python}
from mpi4py import MPI
comm=MPI.COMM_WORLD
rank=comm.Get_rank()
size=comm.Get_size()
~~~

That are the first two bullet points above. This needs to be done anyway, irrespective of the content of the program. As usual we can safe here and test if we broke anything by doing this. We should test with mpirun but use only one process, since we haven't really made any accomodation for multiple processes yet.

~~~ {.python}
mpirun -np 1 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
~~~

We've chosen some reasonable arguments that display most of the Mandelbrot set with a medium resolution. If this works out alright we know that the MPI package works and we can do basic MPI calls. Next step is to add the stuff that needs to be done by only one process (let's say rank 0 because that s always available). We can take this opportunity to do the timing as well. So let's put an "if" statement in front of the line where we make our "mandel" data structure by filling it with zeroes. And add a start marker for timing:

~~~ {.python}
manrow=np.zeros(nr)
if (rank==0):
    mandel=np.zeros((nr,ni))
    start=tm.time()

~~~

Don't forget the finishing blank line or else we end up having everything in our if statement. Note that the "manrow" structure (which contains a line of values) is needed by all processes, so it has to be outside the if statement. The "start" line is calling a timing routine first. Let's leave the main loops in peace for now because we're not quite ready yet for actual work distribution. But we can put another "if" in front of the stuff at the bottom, because printing/plotting results is going to be done only by rank 0:

~~~ {.python}
if(rank==0):
    end=tm.time()
    print ("Done in ",end-start,"seconds.")
    image=plt.imshow(mandel)
    plt.show(image)

~~~

We also call the timing again and print out the difference as the time it took. Again, don't forget the blank line. Now let's test one more time with only one process to make sure.

~~~ {.python}
$ mpirun -np 1 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
Done in  3.833190441131592 seconds.
(...plot display...)
~~~

Now we have pretty much put up the framework and we need to tackle the loops. The outermost of them (the one over iindex, which corresponds to rows of pixels that correspond to a fixed imaginary part) is the one we want to "parallelize". An easy way to do this is to compute a bunch of them on one process and another on another. 

But there is a problem: It turns out that the stuff inside of the Mandelbrot set takes longer to compute than the areas outside it. Which means if we are grouping the rows together like that we end up having one process (the one that works near an imaginary part of zero) do a lot more work than the others, and that isn't good. Remember the "workload balance" principle mentioned earlier. 

So it's much better to go like this: instead of going through the loop one-by-one we "skip" values of iindex. Exactly as many as there are processes. And each of the processes computes one of the rows in between. So rank 0 get row 0, 4, 8, etc… and rank 1 gets 1,5,9, etc… if there are 4 processes in total. This  means that the new loops looks like this:

~~~ {.python}
for iindex in range(0,ni,size):
    for rindex in range(0,nr):
        b=point(rindex,iindex+rank)
        manrow[rindex]=mandit(b)

~~~

Note that all we did was add "size" as a stride in the range call,, and added "rank" in the point() computation so that each of the processes (ranks) compute a different set of points. Now we need to modify the core of the loops. Rank 0 will first use its own version of "manrow" which contains one row of points to fill the proper part of the total data structure "mandel"

~~~ {.python}
    if (rank==0):
        mandel[iindex][:]=manrow[:]
~~~
        
Then it gets the versions on the other processes one-by-one through a "MPI.Recv" call and slots them into the proper spot in "mandel":

~~~ {.python}
        for irank in range(1,size):
            comm.Recv(manrow, source=irank, tag=11)
            mandel[iindex+irank][:]=manrow[:]
~~~

Note that the loop index irank is "the other process' rank". The last thing that remains to do is to have the individual ranks call the MPI.Send function to send their "pixel rows" to rank 0. 

~~~ {.python}
    else:
        comm.Send(manrow, dest=0, tag=11)
    comm.Barrier()
~~~

For good measure, we have called an MPI.Barrier so that all processes are done with everything before we move on to the next set of rows. We didn't even have to specify the data types of manrow, or the size of the array we're sending around. MPI (and Python) is smart enough to do this for us. We also don't need a different tag for every communication, because between a given set of processes (i.e. 0 and another one) there's at any given time only one around, and no extra labeling is required. This is one of the reasons for having that barrier, so that multiple communications can't "run into each other".

Let's see if it works, first with one processor:

~~~ {.bash}
$ mpirun -np 1 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
Done in  3.4627363681793213 seconds.
~~~

Then with 2, 4, and 8:

~~~ {.bash}
$ mpirun -np 2 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
Done in  1.801743745803833 seconds.
$ mpirun -np 4 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
Done in  0.8913767337799072 seconds.
$ mpirun -np 8 ./MPImandel.py 200 200 -1.5 0.5 -1. 1. 1000
Done in  0.48674607276916504 seconds.
~~~

That is pretty good scaling for a simple program like this. Welcome to the wild and wonderful world of high-performance computing.

> ## Same thing different way {.challenge}
>
> This is probably going to be home work. Write a program that does the same thing as our MPImandel program, but uses a Bcast()/Reduce() approach to do it. Hints: You won't need an "mrow" array that contains a specific pixel row. Instead you have a copy of "mandel" (i.e. all the pixels) on each process. First you fill it all with zeroes, then you ucompute only those rows in the total structure that the specific process needs. Finally you use Reduce() to send them to Rank 0. For each process there will be a lot of zeroes, but that won't matter because zeroes will not contribute to the MPI_SUM.
