---
layout: page
title: Parallel Computing
subtitle: Memory Distribution: Matrix Multiplication
minutes: 
---
> ## Learning Objectives {.objectives}
>
> * Understanding the importance of memory distribution
> * Learning about different methods of distributing memory
> * Learning the application of Scatter/Gather for collective MD
> * Tracking timing and memory usage of programs

Early in this course we briefly mentioned the fact that by default, MPI replicates the full memory space of the program. We also pointed out that often, an iomportant reason for "going parallel" is to make use of the larger memory on a cluster. However, for this to be efficient, memory has to be explicitly distributed.

In this portion of the course we use the example of a matrix multiplication to illustrate how we can "scatter" pieces of data to the processes, so that every process only has those parts of the data it needs to do its portion of the work. In the opposite direction, we will see how results can be "gathered" from the processes for IO or further processing.

This is probably the most difficult part of this workshop. Memory distribution is somewhat tedious and error prone because it requires the proper computation of the size and position of the pieces that need to be sent out or received. Let's get to it.

Matrix multiplications are summed up by the rule: for an element i,j of the result, multiply the elements of row i of one matrix, with the elements of column j of the second matrix, and sum up the results. That means if we multiply a 300x400 matrix with a 400x500 matrix we obtain a 300x500 matrix.

In Python, we represent matrices as "two-dimensional" data structures that require two indices, the row index, and the column index. In our case, we use numpy arrays.

Following the above rule, we are looking at three data structures, two input matrices, and a result matrix: A X B = C

<img src="fig/matmul.PNG" width="600">

In the figure we have already "striped" the result matrix C and the input matrix A and labeled the indiviual blocks with the rank of the process that will be assigned to them. To compute the first, say 100 rows of the result C, we only need the first 100 rows of A. This offers an obvious path to parallelization. Let each of the processes only handle a block of rows for A and C. All of B will be used by all processes, though.

In this course we are looking at two versions of the matrix multiplication code. One does not distribute the memory but rather uses the "broadcast/reduce" method that we have already encountered in the "sum of suare roots" example. That's the simplest way of doing it, but it wastes memory and communication. The other distributes the memory and uses "scatter" and "gather". Before we get into that, let's look at the first one.

We can broadcast both input matrices A and B to the processes as a whole if we don't mind communicating and storing information that is not needed. For Matrix A : 

<img src="fig/matmul-broadcast.PNG" width="600">

Then we have each of the processes (ranks) work only in a range of rows that was assigned to it (roughly 1/Nth of the rows), and store the results in local version of the result matrix. Let's call this "partial result matrix" P. The elements of that the process is not using are set to zero. At the end, all the matrices P from the processes are sent back to the root process (rank 0) through a "reduce" operation that sums over them. Because the elements that haven't been computed by a specific process are zero, the won't contribute to the sum, so the result comes out correctly.

<img src="fig/matmul-reduce.PNG" width="600">



