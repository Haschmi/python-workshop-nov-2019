---
layout: page
title: Parallel Computing
subtitle: Hello World
minutes: 20
---
> ## Learning Objectives {.objectives}
>
> * Get some hands-on experience with writing and running parallel program.
> * Use the MPI_Rank() and MPI_Size() functions.

Now let's get something to run with multiple processes. We try this out with the usual "Hello World" example. Let's make a bunch of processes and have them print out a message to the screen. We can't use the command line approach for this because we need to start everything with an "mpirun" command. So first we have to type in the program in an editor. Let's do it step by step:

First we need a header line that lets the system know that we are using Python 3:

~~~ {.python}
#!/usr/bin/env python3
~~~

The "hash-bang" header is often necessary if we want execute a script without spelling out what language we're using every time. Next we have to import  the MPI interface for python:

~~~ {.python}
from mpi4py import MPI
~~~

Note that this also causes MPI to be initialized, so we won't have to call MPI_INIT. Thanks, MPI4Py.

Let's safe here and try out if anything breaks:

~~~ {.python}
$ mpirun -np 8 ./hello.py
~~~

If we did anything wrong or we're not set up right, then we'll get an error message here. If everything's OK, we get nothing. With larger number of processors we may get a warning like this

~~~ {.python}
$ mpirun -np 12 ./test0.py
~~~
~~~ {.error}
--------------------------------------------------------------------------
A request was made to bind to that would result in binding more
processes than cpus on a resource:

   Bind to:     CORE
   Node:        hc10
   #processes:  2
   #cpus:       1

You can override this protection by adding the "overload-allowed"
option to your binding directive.
--------------------------------------------------------------------------
~~~

That's because the nodes we're playing with have only 8 cores and we're trying to run more than 8 processes on it. Let's stick with 8 or less. 

Let's add what communicator we want to use:

~~~ {.python}
comm = MPI.COMM_WORLD
~~~

Then make MPI calls to determine rank and size:

~~~ {.python}
rank = comm.Get_rank()
size = comm.Get_size()
~~~

Note how the rank and size are part of the communicator "comm" which we have set to the default MPI_COMM_WORLD.
Finally, print out the famous "Hello World" message:

~~~ {.python}
print ("Hello World from rank ",rank," of a total of ",size,"processes")
~~~

Save here and try it out with 8 processors:

~~~ {.python}
$ mpirun -np 8 ./hello.py
Hello World from rank  1  of a total of  8 processes
Hello World from rank  2  of a total of  8 processes
Hello World from rank  4  of a total of  8 processes
Hello World from rank  5  of a total of  8 processes
Hello World from rank  6  of a total of  8 processes
Hello World from rank  7  of a total of  8 processes
Hello World from rank  0  of a total of  8 processes
Hello World from rank  3  of a total of  8 processes
~~~

Note how the ranks are slightly out of order. As mentioned before rank has nothing to do with the order in a parallel program. The only reason we don't get a "jumbled mess" on the screen is because the system has an IO buffer and pre-orders things a little to make them readable.
