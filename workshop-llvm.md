# Workshop on LLVM in HPC

https://llvm-hpc5-workshop.github.io/
http://conferences.computer.org/scw/2018/#!/toc/4

## Related Events

* [BoF: LLVM in HPC](https://sc18.supercomputing.org/?post_type=page&p=3479&id=bof111&sess=sess372), Tues Nov 13
* [BoF: C++ and Heterogeneous Programming](https://sc18.supercomputing.org/?post_type=page&p=3479&id=bof175&sess=sess429)
* BoF: OpenMP

## Keynote: Glow: An Optimizing Compiler for High-Performance Machine Learning

Facebook
Compiling Neural Networks to Accelerators

https://github.com/pytorch/glow
https://arxiv.org/abs/1805.00907

### Performance over Time

Steady moore's law improvements

* CISC
* RIASC
* ILP
* Pipelining

around 2005-10, starting to have to 

* Multicore
* GPGPU

Cannot rely on more transistors. Need to change how we write software.

* Also going to see increasingly specialized designs.
* Very expensive. Needs to be the right application given the cost.
* Machine Learning at Scale
* Perfect target for accelerators.

### Machine Learning

Fuzzy, ill defined tasks.

Data flow graph of linear algebra operators.

"Great if you love FLOPS"

### Specialization -> Efficiency

* Matrix Multipliers
* Dedicated SRAM banks
* Narrow arithmetic bit-width: fp16, u8 (google custom bfloat)
* Explicit DMA
* Specialized Programming Model

### Fast-Moving Software

Need to fit in with PyTorch framework.

"Path from research to production."

"Don't want people to have to become experts in ASIC programming."

Trying to build "ML Hardware Ecosystem"

Glow is a bridge between PyTorch and Hardware Acceleration.

### Arch

Inputs (ML frameworks):

* ONNX
* PyTorch
* C++

Mapped to "Glow Core".

* Fed through ML optimizer.

CodeGen for:

* x86
* OpenCL
* ASICs...

Pipeline:

* High level: graph & linear algebra optimizations
* 

### Graph Lowering

ML frameworks have:

  100s of ops * N accelerator backends * custom assembly

Strategy is canonicalization:

* Lower to smaller set of operations
* But you need a compiler that can recognize high-level patterns

### High-level Graph

* Pure dataflow.
* Static types.
* Domain specific optimization:
  - fuse batchnorm and conv
  - eliminate transpose
* Classical optimization
  - Dead code elimination
  - Constant propagation

### Linear IR

"Assembly language for ML"

* Linearized
* Not SSA

Typed memory regions. Here we can **optimize memory use**:

* Re-use regions
* Concatenation in place
* Minimize memory footprint

### Quantization

Neural nets are resilient, use narrow bit-width math (u8, fp16).

"Profile-guided Quantization"

* Glow uses a "instrumented network".
* Run with training data
* record histogram at each edge

Recompile:

* Quantize types
* Optimize scaling

### JIT vs. Ahead-of-Time

JIT:

* Use dynamic information
* Interactive use
* Simplify deployment

AOT:

* Powerful optimizations
* Deployment of few models

Glow does both. LLVM makes this easy.

### CPU Backend

LLVM-based: optimizer, code generator, JIT

Import math kernel library as LLVM bitcode.

* Written as C/C++
* Apply runtime optimizations.

Lower Glow linear IR to bitcode (which calls the kernel library).

* Custom LLVM passes
* Regular passes
* LLVM codegen

#### Custom Pass: Function Specialization

Common to have constant parameters to functions.

This pass builds separate functions with constant parameters.

Much more friendly to the optimizer.

#### Custom Pass: Kernel Fusion

Combine kernels for memory locality.

Example: element-wise mul-sub -> fused mul-sub

A lot like loop fusion, but doesn't actually construct the loop. At the high
level it already knows it's an elementwise kernel.

### Memory Management

Accelerator memory management.

  DMA --- SRAM Scratchpad --- ALU

**Static** memory allocation. Hardware often does not even have pointer dereference.

Similar to register allocation.

* Minimize eviction
* DMA prefetch
* Variable size buffers

### Future

* Accelerator Code Generation
* Specialized tenso operators
* Graph partitioning (multiple cards...)
* Memory prefetching
* Pipelining
* Operator scheduling


## Pointers in OpenMP Lambda CLosures

"Performance Portability Frameworks"

Frameworks using higher-order functions for loop constructs have become more
ergonomic with the introduction of lambdas. Example:

* RAJA
* Kokkos

These include OpenMP backends.

Until OpenMP 5 the spec did not say what should be done with lambda functions
called in offload regions.

### How do Lambdas Work?

Conceptually, transformed to anonymous class for the closure, with overloaded
call operator for the body itself.

On the host, pointers exist in the same address space, so no complication.

If you offload the anonymous class, you'll have __host__ pointer addresses.

You can workaround this by wrapping the lambda creation point in an openmp
pragma. But this **breaks the abstraction** that frameworks like RAJA are
trying to create.

The compiler needs to map any pointers in lambdas if they are called from
device functions.

### Design

Do not recursively map pointers in objects.

Each pointer mapped requires 2 loads and 2 stores... poor perf.

Unfortunately this is arguably confusing behavior.

### Results

Tested on LULESH & TeaLeaf.

Pure OpenMP implementations are substantially faster (25%), compared to RAJA.

RAJA uses STL-style iterators (begin, end) which has a **lot of overhead**.

But, performance of the explicit vs. compiler-assisted offload was nearly
identical. This was confirmed by TeaLeaf too.

### Conclusion

* Implicit compiler mapping for lambda closures gives equivalent performance.
* If the compiler is able to optimize loops correctly (RAJA can cause issues),
  lambda performance is equivalent.


## Clacc: OpenACC in Clang

* OpenACC in Clang
* Map to OpenMP

### OpenACC

Portable programming model for heterogeneous accelaerators.

Looks like OpenMP... directive based.

Aimed at incremental development of 

### Difference with OpenMP

OpenACC is for multi-core CPU **and** GPUs.

* OpenACC is "descriptive"
* OpenMP is "prescriptive"

Now OpenACC features are making their way into OpenMP.

Now OpenMP is **large**. OpenACC is a tractable smaller subset.

Also architecture differences. Translator is valuable.

### OpenACC compilers

* Commercial: PGI
* Open Source: GCC
* Academic: OpenARC, ROSE, ...

Why LLVM? Licensing. Also easier to build new tools on top.

### Objectives

* Production quality compiler.
* Enable research and development of source-level OpenACC tools.
* Contribute to Clang upstream (currently a fork)
* Throughout development: contributing patches

### Design: Translate OpenMP to OpenACC

Pragmatic choice. Build on the OpenMP implementation.

Makes sense: descriptive to prescriptive.

OpenMP is an "IR". Should not be exposed to the user. Requires full validation
of OpenACC upfront.

Also enables:

* Resusing OpenMP tools.

### Design Alternatives

A:

* No OpenACC AST
* Hard to build source-level OpenACC tools

B:

* No OpenMP AST
* Hard to support non-traditional user

C:

* OpenACC AST to OpenMP AST

Adopted C. But:

* What about cases where OpenACC does not map to OpenMP well
* Represent OpenACC in LLVM IR for improved analyses
* What about source-to-source?

### acc2omp alternatives

1. Replace with OpenMP. NO: clang AST immutable
2. Annotate with OpenMP. Use clang rewrite, reparse to build OpenMP AST. NO:
   poor compilation performance.
3. Add hidden OpenMP subtree. Doesn't violate immutability, it's adding nodes.
   Use clang `TreeTransform` facility.

### Long Term Plan

* Focus on C. Later C++
* Prescriptive then descriptive interpretation.
  - "prescriptive" = one-to-one mapping to OpenMP
  - "descriptive" = analyses needed to determine best mapping
* Shared memory multi-core then GPUs and other accelerators

### Evaluation against PGI

Suggests next steps:

* Support for GPUs
* Descriptive OpenACC interpretation


## -fsimdmath on ARM

### Arm Development Solution Group

Developer tooling:

* compilers
* simulators
* debuggers

Compilers for HPC:

* LLVM-based and GCC
* C/C++/Fortran
* SIMD (NEON + SVE)

Libraries:

* BLAS / FFTW
* libamath / libm

### Problem Statement

"Vectorize calls to C99 math functions"

```
#include "math.h"

...

  for(int i = 0; i < N; i++) {
    y[i] = sin(x[i])
  }
```

### Ingredients

Need to know:

* Name and signature of scalar function
* Vectorization factor
* Masking
* Target vector extension

Also need a vectorized math library. Use SLEEF, packaged as `libsimdmath`.

### Vector function ABI for AArch64

Vector procedure call standard "AVVPCS"

Name manging for SIMD vector functions.

* `_ZGV` for vector function
* neon or SVE
* masked or now
* vectorization factor

### SLEEF

Vectorized versions of *all* C99 functions. Many supported arch/OS/ISA.

"Meta" language in C. Easy to extend with helper header files.

### Implementation

Split between front-end and backend.

* Directives in custom "math.h" header
* Global declarations map from scalar functions to expected vector signature

Backend:

* Loop Vectorizer
* TargetLibraryInfo
* Global declarations are lost.

Built with extensibility in mind. Could be used for other backends too (as
long as there is a naming convention for the vectorized).

Very hard to unit test.

### Driver

```
-fsimdmath -> -fopenmp-simd -D__SIMDMATH=1 -lsimdmath
```

Extendable to other libraries FOO.

### Results

Promising. Also not promising.

* Significant speedup on `qmcpack`.
* No significant speedup on other codebases.
* Reason: no Arm specific optimizations in the library.
* Reason: used 1 ULP verison of SLEEF rather than 3.5

### Tried to upstream

* Already have `-fveclib`.
* Already have OpenMP `declare simd`
* Need to decouple vectorization from the frontend

Working on replacing impl. of `-fveclib`.

* OpenMP 5 adds `declare variant`


## Function/Kernel Vectorization via Loop Vectorizer

(missed part of this)

### Problem

Loop contains library call. Can't be vectorized.

### Proposal

Library functions can declare SIMD versions (with directives).

These are recognized and enable the loop to be vectorized.

### Implementation

Trunk has 3 vectorizers:

* Loop Vectorizer
* SLP Vectorizer
* Load-Store Vectorizer

No need to create one more. Reuse loop vectorizer.

### Resources

* http://lists.llvm.org/pipermail/cfe-dev/2016-March/047732.html
* https://reviews.llvm.org/D22792


## User-directed Loop Transformations in Clang

### Examples

```
#pragma unroll
for(int i = 0; i < 4; i++) {
  f(i)
}
```

Or

`#pragma unroll 4`

will unroll into a block, with duff's device.

**Many** many more types of pragmas.

### Advantages

* Separation of sematics ans optimization.
* Different optimizations for different platforms (with preprocessor)
* Safe semantics. Loop autotuning.
* Easy to experiment.

### Downsides

* Fragile (compiler may not be able to to apply it.
* Missing domain knowledge
* Incompatible with OpenMP
* Compiler-specific

### Proposal

OpenMP proposal.

```
#pragma omp loop(...) transformation switch option(...)
```

* `#pragma` order is taken into account
* loops can have identifiers
* `#pragma fuse` (and others) can take multiple input loops (as ids)
* or pragmas can produce multiple loops
* with ids... pragmas can be moved away from the loop itself

### Prototype

Based on Clang/LLVM/Polly pipeline.

Passes loop metadata in IR.

Uses `#pragma clang loop() ...` since not in OpenMP yet.

### Internals

Preprocessor:

* Replaces `#pragma ...` with a special "annotation token" for the parser.
* Prototype has its own (similar) token

Parser:

* In OpenMP, Sema::ActOnOpenMP<Construct>Directive()
* In prototype, creates Loop attribute.

AST:

* Adds `Loop<Type>Attr` AST nodes. Wrapped in `AttributedStmt`.
* Table gen'd `Attr.td`

IR:

* `LoopInfo` collects `LoopTransformation` objects
* Updates `llvm.looptransform` function metadata


## OP2: DSL in Clang/LibTooling

Accelerators are rapidly changing. "Future proof" HPC applications:

* DSL to **declare** the problem
* Without specifying its implementation
* Create a lower implementation level.

Large companies already use these kinds of DSLs.

Need tools that are:

* Robust
* Maintainable
* general methods
* Open source

... to foster development of DSLs.

Propose skeleton technique for writing DSLs with Clang/LibTooling.

### OP2

* Oxford Pralllel Libary for Unstructured Solvers
* Operates on unstructured grids - graph structure
* Explicit connections
* Parallelization is difficult to avoid races

OP2 helps by offering:

* Defining sets
* Datasets on sets
* Mappings
* Parallel loops

Looks like C. Workflow:

* OP2 -> C -> regular compiler

### Implementation

Python-based translater so far. But... with Clang we would get:

* Syntax checking
* Semantic checks
* Automatic macro and expression evaluation
* Generalizes better to other domains

Use LibTooling.

### Libtooling

* Search in AST for changes in source code.
* Good for small/local changes.
* Hard to handle significant structural changes.

Transformation is:

1. Collect data a modify the OP2 files.
2. Code generation

### Technique

Write functions in C that call a known "skeleton" (marker) function. Parse it
to get the AST. This is a template. Call it "invariant code" because it
doesn't change.

We can now process the template and replace the skeleton with the actual
function to be called.

Advantages:

* Easy to extend
* Easy to write
* More robust code generation
* "Invariants" or "templates" are themselves code, so are verified by clang

### Summary

Targets OpenMP, vectorized, CUDA

### Future

* Generalize to other structured applications
* Propagate information to IR
* Optimization passes at IR level


## PInT: Pattern Instrumentation Tool

* Understand development & optimization efforts
* Consider parallel patterns in code and calculate metrics
* Specifically, how much effort to parallelize

```
Source code -> Detect Patterns -> Structure Extraction -> Metrics
```

Manual pattern recognition is difficult and time-consuming.

### Example Patterns

Map / Reduce / Critical / ...

Mattson et. al.

* Finding concurrency
* Algotihm structure
* Supporting structure
* Implementation level

### Tool Goals

Should work with most HPC code: C/C++, Fortran, OpenMP, MPI

Clang LibTooling

* Exchangable pattern definitions
* Visualization
* Data export

### Steps

* Code has to be instrumented manually by the user.
* Clang used to get AST.
* -> PInT pattern graph
* -> User-defined metrics.

### Instrumentation

`Pattern_{Begin,End}` calls.

Have to manually identify patterns in the code.

### Implementation: Vistor

`VisitCallExpr` finds the string literal in pattern calls.

Processed into a pattern graph.


## AIWC: OpenCL-Based Architecture Independent Workload Characterization

(missed)

## Compiler Optimization for Heterogeneous Locality and Homogeneous Parallelism in OpenCL and LLVM

Optimize for locality (on DSP)

* Tile loops.
* Fuse kernels.
* Overlap transfers and computation.

On multi-core CPU want to optimize for parallelism.

* SPIR optimized for parallelism.

Want to **write once** such that we can:

* optimize for locality on DSP
* optimize for parallelism on their developer workstation

Implemented in LLVM.

## A Study of OpenMP Device Offloading in LLVM: Correctness and Consistency

Various "bugs" / spec inconsistency between OpenMP code on CPU / GPU:

* Barrier
* Memory Fence
* Shadowed Data Race

Planning TSAN-like tool to automatically find these problems. Contributing
back to LLVM.

## Challenges of C++ Heterogeneous Programming Using SYCL Implementation Experience: the Four Horsemen of the Apocalypse

C++ Heterogeneous

- OpenCL
- OpenML
- SYCL

with

- kokkos
- raja
- boost compute
- cuda

Road to standardization?

* Execution models, and methods of execution.
* Need a unified layer between them

Data representation:

* Implicit or explicit data movement?
* Row/col major
* SoA or AoS

Affinity:

* Near or far?

Directions... can it be done in C++

* Preference for small movement not large changes

Or:

* SYCL?
* https://www.khronos.org/spir/


