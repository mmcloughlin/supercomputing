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

