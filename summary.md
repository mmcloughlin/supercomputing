# Summary

## Followups

* Parallelism abstractions: [RAJA](https://github.com/LLNL/RAJA), [kokkos](https://github.com/kokkos/kokkos) ([example](https://github.com/UoB-HPC/advanced-hpc-examples/blob/master/Kokkos/vecadd-kokkos.cpp)), [SYCL](https://www.khronos.org/registry/SYCL/specs/sycl-1.2.pdf)
* ARM NEON + SVE. [SLEEF](https://github.com/shibatch/sleef). AVVPCS
* `bfloat16` (generally `int8`, `fp16` in ML) https://software.intel.com/sites/default/files/managed/40/8b/bf16-hardware-numerics-definition-white-paper.pdf
* Projects: https://github.com/kavon/atJIT, https://op-dsl.github.io/,
  https://github.com/RWTH-HPC/PInT, https://github.com/LLNL/TraceR, https://github.com/StanfordLegion/legion, http://regent-lang.org/, https://github.com/pssrawat/LARS, Tensor cores, https://github.com/SciCompKL/CoDiPack, http://tapenade.inria.fr:8080/tapenade/index.jsp, https://klee.github.io/
* http://tacc.github.io/SDVis/ https://sites.utexas.edu/jdm4372/
* x86 machine check architecture?
* https://cavium.com/
* GEMM (General Matrix Multiply), [LIBXSMM](https://github.com/hfp/libxsmm)
* Interesting companies/teams: [CodePlay](https://www.codeplay.com/), Arm Development Solution Group


## Overall Thoughts

* **Performance Portability** is still the holy grail. Directives-based approaches don't quite make it. Hope to see more experimentation in language design in this area.
* **DSLs** Glow, OP2
* **Quantifying Results** A little disappointed by rigor in the way some presentations reported results. Need greater collection of benchmarks, and resources to enable testing over many platforms (where "performance portability" is claimed).

## Sessions

### Workshop: Compiler Directives for Accelerator Programming ([WACCPD](https://waccpd.org/))

Intro:

* Heterogenious computing now well-established. Accelerators, heirarchical
  memory.
* Far improved performance for the power, but harder to program, hence the
  focus on programming models.
* Workshop focussed on OpenACC / OpenMP (vs CUDA / OpenCL)

Thoughts:

* For me, this workshop could have benfitted from more __raw numbers__. The
  multiple presentations were a little too anecdotal. The keynote presentation
  from DOE already hinted at this problem, where he highlighted the CAAR (Center for Accelerated Application Readiness) who are responsible for collecting critical applications for acceleration. Moreover, he hinted at projects for static analysis to better understand workloads on National Lab HPC.
* Nonetheless I was slightly underwhelmed by reported OpenACC experiences. The talk on [Kernels from Molecular Simulation](https://sc18.supercomputing.org/presentation/?id=ws_waccpd110&sess=sess155) reported 1.4-2x slowdowns from CUDA to OpenACC, and compiler frustrations actually made the programming slightly harder than CUDA. The claimed benefit is performance portability, but this was not quantified.
* One of the later [case studies](https://sc18.supercomputing.org/presentation/?id=ws_waccpd109&sess=sess155) directly contradicted the "performance portability" case, citing huge slowdowns of the GPU-optimized OpenMP code on CPU (and vice versa).
* The [OpenMP Target Offloading](https://sc18.supercomputing.org/presentation/?id=ws_waccpd104&sess=sess155) talk was also interesting but disheartening. Essentially it focussed on various optimizations that a compiler could apply (in theory). Unfortunately the OpenMP spec is unclear in whether these are allowed, and moreover they are difficult to perform at in clang (OpenMP later is handled in the Frontend, wheras complier analyses are at the LLVM/IR level)
* The [Plane Sweep Algorithm](https://sc18.supercomputing.org/presentation/?id=ws_waccpd108&sess=sess155) was difficult to follow, but interesting as an example of using OpenACC for non-standard applications.

### Workshop: LLVM Compiler Infrastructure in HPC

TL;DR; for selected talks

* The keynote on Facebook's ML compiler [Glow](https://facebook.ai/developers/tools/glow) was worthy of the name.

---

* Pointers in OpenMP Lambda CLosures
* Clacc: OpenACC in Clang
* -fsimdmath on ARM
* Function/Kernel Vectorization via Loop Vectorizer
* User-directed Loop Transformations in Clang
* OP2: DSL in Clang/LibTooling
* PInT: Pattern Instrumentation Tool
* Compiler Optimization for Heterogeneous Locality and Homogeneous Parallelism in OpenCL and LLVM

---

### Keynote

**TL;DR;**

### Resilience

* GPU Age-Aware Scheduling to Improve the Reliability of Leadership Jobs on Titan: "Leadership jobs" are a class of high priority jobs on the Titan supercomputer, typically long-running and occupying 20+% of the machine, hence they are far more suceptible to GPU errors. This talk demonstrated improvements by biasing leadership jobs to stable GPUs.
* FlipTracker: Understanding Natural Error Resilience in HPC Applications: automated analysis of application error resilience by injecting instruction-level bit errors. Definition of 6 resilient patterns, and demonstrated their use to improve resilience.

* Lessons Learned from Memory Errors Observed Over the Lifetime of Cielo: Data analysis of memory errors over the entire lifespan of Cielo. Impressive dataset but more or less a "non result": did not find expected age-dependence, and did not show that correctable errors predicted future uncorrectable errors.
* Partial Redundancy in HPC Systems with Non-Uniform Node Reliabilities: exploration of a middle-ground between checkpointing and replication (running two copies) for handling unreliable nodes. At high scale, checkpointing becomes less favorable than replication, but always one of them is preferred over partial replication (replicating subset of tasks). However existing work had assumed equal failure rates of all nodes, which is not borne out in practice. This paper demonstrated that in non-uniform failure rate cases, a (fairly obvious) partial replication scheme can give optimal expected completion time.
* **Evaluating and Accelerating High-Fidelity Error Injection for HPC**: This explored a mechanism of improving fielity of error injection methods. We can inject at (from low to high): RTL (register transfer language), microarch and instruction level. At the lowest level these are __extremely__ expensive, but high level approaches are too __unrealistic__. TODO

### Doctoral Showcase

* In-Memory Accelerator Architectures for Machine Learning and Bioinformatics: 

### Large-Scale Algorithms

* Large-Scale Hierarchical K-Means for Heterogeneous Many-Core Supercomputers
  ...
* **TriCore: Parallel Triangle Counting on GPUs**
  this was good!
* Distributed-Memory Hierarchical Compression of Dense SPD Matrices
  ...

### Task-based Programming

* Dynamic Tracing: Memoization of Task Graphs for Dynamic Task-Based Runtimes
* Runtime-Assisted Cache Coherence Deactivation in Task Parallel Programs
* A Divide and Conquer Algorithm for DAG Scheduling Under Power Constraints

### Resource Management

* RM-Replay: A High-Fidelity Tuning, Optimization and Exploration Tool for Resource Management 
* Evaluation of an Interference-Free Node Allocation Policy on Fat-Tree Clusters 
* **Mitigating Inter-Job Interference Using Adaptive Flow-Aware Routing**

### Arithmetic and Optimization

* **Associative Instruction Reordering to Alleviate Register Pressure**
* **Harnessing GPU's Tensor Cores Fast FP16 Arithmetic to Speedup Mixed-Precision Iterative Refinement Solvers**
* **ADAPT: Algorithmic Differentiation Applied to Floating-Point Precision Tuning**

### Programming Systems Tools

* **ParSy: Inspection and Transformation of Sparse Matrix Computations for Parallelism**
* Detecting MPI Usage Anomalies via Partial Program Symbolic Execution



### Deep Learning

* **Exploring Flexible Communications for Streamlining DNN Ensemble Training Pipelines**
* **Anatomy of High-Performance Deep Learning Convolutions on SIMD Architectures**


# File Systems: Data Movement and Provenance

* Dac-Man: Data Change Management for Scientific Datasets on HPC Systems 
* Stacker: An Autonomic Data Movement Engine for Extreme-Scale Data Staging-Based In Situ Workflows 
