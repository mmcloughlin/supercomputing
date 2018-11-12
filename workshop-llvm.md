# Workshop on LLVM in HPC

https://llvm-hpc5-workshop.github.io/

## Related Events

* BoF: LLVM in HPC, Tues Nov 13
* BoF: C++ and Heterogeneous Programming
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
