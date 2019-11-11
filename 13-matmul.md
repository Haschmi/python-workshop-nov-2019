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

Matrix multiplications are summed up by the rule: for an element i,j of the result, multiply the elements of row i of one matrix, with the elements of column j of the second matrix, and sum up the results.

In Python, we represent matrices as "two-dimensional" data structures that require two indices, the row index, and the column index. In our case, we use numpy arrays.

