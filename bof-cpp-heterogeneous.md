## C++20 Concurrency and Parallelism

Features:

* Will have subset of executors. An executor has an `execute()` method. It's
  fire and forget... doesn't give you something to wait on.
* Semaphores.
* Atomic wait.

Coroutines rejected at this point. May return.

## Status of Future CUDA

"There will be one."

People have formed an opinion about CUDA 5,6,7. But it's changed a lot and
people haven't realized that.

General story arc:

* Don't have to annotate everything. Only things you intend to optimize.
* Communicating semantic informstion about the program to lower levels.
* Vague I know...

## SIMD in C++

Proposal to add explicit SIMD programming to C++ that has been **accepted**.

"You should never have to write an intrinsic again in your life."

Portability in SIMD!?

Data layout transformations, from arrays of structs in scalar to struct of
arrys for SIMD.

"It'll all just work and be great."

There are still tasks left for programmers!

Control flow is still a lot of work for the programmer.

Starting point for another round of higher-level constructs. One example,
remainder loops are not handled.

## ... another way SIMD makes its way into C++20: parallel algorithms

Now we have parallel vectorize.

## mdspan

Multi-dimensional arrays.

Concept of "layout". How do you map a multi-dimensional index to a 1-dim
address into memory.

Fortran-like things: slicing.

Compile time choice of data layout.

Also the concept of views into the data.

Idea comes from kokkos.

