## ParSy: Inspection and Transformation of Sparse Matrix Computations for Parallelism

https://sympiler.github.io/

### Compiler Optimization in Sparse Kernels

Dependency analysis difficult because of indirect memory access.

### Wavefront

How would wavefront DAG algorithms work on loop parallelism.

> Group nodes which can be executed in parallel.

- Node imbalance.
- Doesn't provide locality.

### ParSy

Faster than Intel MKL!

### Sympiler

"Sympiler" DSL for high performance code for **sparse solvers**.

Uses symbolic information about he sparsity (not actual values).

Generated fast code for single core (vectorized etc...)

ParSy implemented on top of Sympiler, but works on parallel systems.

Process:

* Extract sparsity pattern
* Convert to DAG (Symbolic Inspector)
* Inspector-Guided Transformations

### "H-level Inspection"

Example of Cholesky Factorization: sparse A to LL^t

"Elimination tree" represents the computation.

ParSy creates a "H-level" set by

  "Load Balanced Level Coarsening" (LBC)


#### Step 1: Find L-partitions

* Initial cut is coming from source node in DAG
* Continue up while max number of connected components is at least #cores
* Windowing heuristic: allows you to move initial cut based on load-balance
* "Proportional cost model" for load balancing = #participating-non-zeros

#### Step 2: W-partitioning

* Within the sub-DAGs for each level...
* Cost model for load balancing
* "Space parition constring" ensures that threads in different W-partitions do
  not need to syncronize

#### Step 3: Reordering

* Reduce intra-partition cost

Produces H-level partition.

#### Transformations

Start with unoptimized code. ParSy makes modifications to use the H-level
sets, and inserts the `#pragma omp` lines too.

### Results

1.2-2x improvements against MKL.

Domain-specific code generator.


## Detecting MPI Usage Anomalies via Partial Program Symbolic Execution

Symbolic execution can be slow, path explosion.

### Approach

Model MPI calls in symbolic process.

Allow starting execution from any point: "partial 

### Targetting specific MPI anomalies

* Buffer type mismatch
* Buffer data race
* Request overwriting
* Unmatched way or Test
* Unmatched p2p call

### Internal

* Compile to LLCM bitcode.
* LLVM static analysis.
* Plugin to Klee https://klee.github.io/

Record:

* Record ongoing non-blocking MPI calls
  - this allows detection of buffer data race, request overwriting, unmatched
    WAIT or REST
* Fork symbolic execution stated for MPI_Test calls

### Problem

* Large part of applications not in MPI calls
* States forked in this code is not useful for MPI debugging

Solution:

* start execution after those regions
* now can start execution at any point

### Lazy Initialization

If you start at any point, unknown memory state.

* When pointer is dereferenced, allocate memory for it.
* Pointers into that reference the same "memory region"
* Call it "shadow memory"

### Results


