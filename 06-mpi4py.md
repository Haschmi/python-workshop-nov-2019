---
layout: page
title: Parallel Computing
subtitle: MPI for Python (MPI4Py)
minutes: 20
---
> ## Learning Objectives {.objectives}
>
> *   Learn about MPI bindings for Python.
> *   Learn abouut the simplified form of basic MPI features in Python.

MPI was originally designed for FORTRAN and C. Later (in version 2) C++ was added. Meanwhile, some implementations of MPI cover Java. It was only a matter of time that someone would supply us with a Python version. The MPI interfaces we will work with are called <a href="http://pythonhosted.org/mpi4py/">MPI4Py</a>. This is not the only package with this purpose, but it is simple to use and therefore ideally suited for an introductory course.

One nice thing about MPI4Py is that many simple applications don't require as detailed specification as is needed for the native bindings of MPI. For 
beginners this means that they don't have to worry about a lot of details that are required when dealing with MPI in C or C++. On the other hand, once moving on to bigger projects, some of these details become important and one has to re-visit them.

## What we will use

Here's a list of MPI functions that we will be using in our next few mini-projects. They are the absolute minimum needed to write a working MPI program

| Action | Name of Command | Simplest Implementation |
|-----------|----------|-----------------------------------------|
Package Load| | from mpi4py import MPI 
Initialization | MPI_Init | Not required in Python, happens when loading MPI4Py
Finalization | MPI_Finalize | Not required, happens automatically at program end
Communicator | Set directly | comm=MPI.COMM_WORLD
Rank | MPI_Comm_rank | comm.Get_rank()
Size | MPI_Comm_size | comm.Get_size()
Send (blocking) | MPI_Send | comm.Send(data,  dest=drank, tag=itag) (automatic)
Receive (blocking) | MPI_Recv | comm.Recv(data, source=srank, tag=itag) (automatic)
Broadcast | MPI_Bcast | comm.Bcast(data, root=rrank) (automatic)
Reduce | MPI_Reduce | comm.Reduce(pdata, tdata, op=MPI.op, root=rrank) (automatic)
Barrier | MPI_Barrier | Comm.Barrier()

## Initialization and Finalization

There is an actual intialization routine included (Init()) but we won't have to use it, as it automatically called when we load the MPI4Py package. Youi can check with MPI.Is_initialized(). If you're trying to do the initialization again, it throws an error:

~~~ {.python}
from mpi4py import MPI
MPI.Is_initialized()
~~~
~~~ {.output}
True
~~~
~~~ {.python}
MPI.Init
~~~
~~~ {.error}
--------------------------------------------------------------------------
Calling MPI_Init or MPI_Init_thread twice is erroneous.
--------------------------------------------------------------------------
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "MPI/MPI.pyx", line 113, in mpi4py.MPI.Init (src/mpi4py.MPI.c:144518)
mpi4py.MPI.Exception: MPI_ERR_OTHER: known error not in list
~~~

Same with Finzalization: not need to do it explicitely, as it is automatically done when the program quits. If you must for some reason you can with MPI.Finalize(). Of course, you can't finalize twice either. In fact trying to do that crashes the whole program:

~~~ {.python}
MPI.Finalize()
MPI.Is_finalized()
~~~
~~~ {.output}
True
~~~
~~~ {.python}
MPI.Finalize()
~~~
~~~ {.error}
*** The MPI_Finalize() function was called after MPI_FINALIZE was invoked.
*** This is disallowed by the MPI standard.
*** Your MPI job will now abort.
[(null):17185] Local abort after MPI_FINALIZE completed successfully; not able to aggregate error messages, and not able to guarantee that all other processes were killed!
~~~

## Communicators, Rank and Size

As mentionedd before, Rank and Size are "communicator specific". Communicators are easily specified in MPI4Py. Just set a variable to the communicator you want to use, and call all functions as members of that communicator:

~~~ {.python}
from mpi4py import MPI
mycomm=MPI.COMM_WORLD
print ("My rank is ",mycomm.Get_rank())
~~~
~~~ {.output}
My rank is  0
~~~
~~~ {.python}
print ("The size is ",mycomm.Get_size())
~~~
~~~ {.output}
The size is  1
~~~

## Send and Receive

We're only going to deal with blocking send/receive calls, as they are fairly simple. We can even test them out from inside the interpreter with one process only who sends itself a message:

~~~ {.python}
message=657
mycomm.send(message, dest=0, tag=11)
print("The received message is ",mycomm.recv(source=0, tag=11))
~~~
~~~ {.output}
The received message is  657
~~~

Sort of silly, but it illustrates the principle. Careful with the tag. If the one in the receive doesn't match the one in the send, you're sitting there 'til kingdom come:

~~~ {.python}
mycomm.send(message, dest=0, tag=11)
print("The received message is ",mycomm.recv(source=0, tag=12))
~~~
~~~ {.output}
[...nothing to see here, folks...]
~~~

## Broadcast, Reduce, Barrier

These are collective operations and sort of require more than one process to demonstrate. So let's postpone that to the next lessons. However, there's one thing we need to bring up here: the simplest way to use collective communication in MPI4Py is by asending stuff as arrays. So even if it's just a simple number you want to broadcast (i.e. to send from one process to all the others) it's best to do something like:

~~~ {.python}
import numpy as np
number=np.zeros(1)
mycomm=MPI.COMM_WORLD
mycomm.Bcast(number, root=0)
~~~

Not much happening since a single process "broadcasts" a single number ot itself. But at least it doesn't crash. Note that we're using np.zeroes(1) to define the shape of number, i.e. an array of length 1, i.e. a single number. Barriers for a single process are not much more exiting:

~~~ {.python}
mycomm.Barrier()
~~~

No output because the oonyl process we're waiting for already called the barrier.

Note that for most of these funtion calls there are several ways of specifying details by adding more arguments. For how to use this, you need to study the package in detail. The most compelling aspect of these interfaces is that Python usually can figure out what is the type and shape of the data that are being transmitted. As long as things match on both sides of the communication, you're usually good.

