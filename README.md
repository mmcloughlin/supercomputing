<p align="center"><img src="https://i.imgur.com/LdECucq.jpg" /></p>

# SC18

Notes from Supercomputing Conference in Dallas, November 11-16, 2018.

## Themes

* **Performance Portability** is still the holy grail. Directives-based approaches don't quite make it. Hope to see more experimentation in language design in this area.

* With heterogeneous computing firmly establised, we're seeing more **Domain Specific Languages** and compilers. For example [Glow](https://facebook.ai/developers/tools/glow) for Deep Learning and [OP2](https://op-dsl.github.io/) for parallel mesh code.

* **Quantifying Results** A little disappointed by rigor in the way some presentations reported results. Need greater collection of benchmarks, and resources to enable testing over many platforms (where "performance portability" is claimed).

* **Resilience** methodologies are changing due to unprecendented scale.

* **Mixed-precision** is increasingly important in deep learning applications
  and beyond, as bandwidth per compute is driven down.

## Sessions

### [Workshop: Compiler Directives for Accelerator Programming](workshop-directive-programming.txt) ([WACCPD](https://waccpd.org/))

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

### [Workshop: LLVM Compiler Infrastructure in HPC](workshop-llvm.md)

* Talk on **Facebook's ML compiler [Glow](https://facebook.ai/developers/tools/glow)** was a worthy keynote. High level "assembly for ML", upon which various ML optimization passes are performed. Then code generation for x86, OpenCL and various unspecified ASICs. "Profile guided quantization". JIT and Ahead-of-Time (made relatively easy because of LLVM.) GPU backend makes the heaviest use of LLVM, with a couple of custom passes.

* [Pointers in OpenMP Lambda Closures](papers/pointers-lambda-closures.pdf)
  Described the problem of handling lambda functions offloaded to GPU, in
  partiular the case where the lambda closes over a pointer. This can be
  handled by hacky application of some `#pragma omp` directives. They
  demonstrate an implicit method within the compiler, with equivalent
  performance.

* [Clacc: OpenACC in Clang](papers/clacc.pdf) describes initial efforts to
  support OpenACC in Clang. The approach is to map OpenACC to OpenMP, claimed
  to be a "natural" translation because OpenACC is "descriptive" while OpenMP
  is "presscriptive". The implementation builds a "hidden" OpenMP AST from a
  parsed OpenACC ast, which is supported because it doesn't violate Clang AST
  immutability (you can add nodes). Prioritization: focus on C over C++,
  implement "prescriptive" parts of OpenACC first (easier mapping), multi-core
  CPU then CPU.

* [`-fsimdmath` on ARM](papers/fsimdmath.pdf) goal is to "vectorize calls to
  C99 math.h functions". Uses custom name-mangling scheme to describe ARM
  NEON/SVE math functions. [SLEEF](https://github.com/shibatch/sleef) library
  provides the actual implemenations. Clang+LLVM changes required in frontend
  and backend, to commuicate the target vectoried versions through the the
  `LoopVectorizer` pass. Work on upstreaming and combining with existing `-fveclib`.

* [**User-directed Loop Transformations in Clang**](papers/user-directed-loops.pdf) attempts to unify the many disparate loop `#pragma`s. Specifically: take order into account, give loops IDs so pragmas can be separated from the loops themselves (supporting optimizers). Hoping to standardize in OpenMP at some point.

* [**OP2-Clang: A Source-to-Source Translator Using Clang/LLVM
  LibTooling**](papers/op2.pdf) presented a DSL for algorithms on unstructured
  grids (graph structures) where parallelization is a minefield of data races.
  OP2 calls expand to substantial loop structures. This is a Clang/LibTooling
  approach that rewrites the AST. To avoid horiffic AST building code they have "templates" written in C that make calls to stub functions. The tool parses the AST and replaces stub calls with desired function calls, like template expansion. Reminds me of [genny](https://github.com/cheekybits/genny), and brings similar benfits since the template code can be parsed/tested independently. Why didn't they use C++ templates?

* [PInT: Pattern Instrumentation Tool](papers/pint.pdf) left me a bit
  confused, to be honest. Seems to be part of an effort to better-understand
  development and optimization effort in codebases. Provide a mechanism for
  tagging instances of parallel programming "patterns" (map, reduce, critical,
  ...) in code and then gathering some kind of metrics on them. I was left
  somewhat unclear what these metrics were, and the requirement of manual
  tagging seemed disappointing.

* Compiler Optimization for Heterogeneous Locality and Homogeneous Parallelism in OpenCL and LLVM. Described an LLVM-based tool for programming once and running on DSP as well as multi-core CPU architectures. On DSP want to tile loops, fuse kernels, overlap transfers and communication. On multi-core CPU was to optimize for paralellism. Unclear to me how complete this project was, but sounded useful.

### [Keynote](keynote.md)

Very _fluffy_, relative to the rest of the conference. "Most of history has
been boring from an economic perspective," in terms of its impact on the
living standards of regular people. Claims Technology (Industrial Revolution
and beyond) was at the core of the massive growth to 50x the wealth of
subsistence farmers. We are currently experiencing a new revolution:

* **Mind** to **Machine**: Previously decisions made by the HiPPO (highest
  paid person's opinion), now data driven. Rise of AI, first wave "teaching
  machines what we know", second wave "machines that learn".

* **Product** to **Platform**: Every one of the top 7 companies is a platform
  company. Apple in the smartphone space (app store), Uber and Ridehailing.
  Network effects and two-sided marketplaces.

* **Core** to **Crowd**: Crowdsourcing. "The smartest people do not work for
  you."

### [Resilience](resilience.md)

* [GPU Age-Aware Scheduling to Improve the Reliability of Leadership Jobs on Titan](papers/pap262s4-file1.pdf): "Leadership jobs" are a class of high priority jobs on the Titan supercomputer, typically long-running and occupying 20+% of the machine, hence they are far more suceptible to GPU errors. This talk demonstrated improvements by biasing leadership jobs to stable GPUs (maintaining ). Some unintended consequences: when multiple leadership jobs were running, one would get all the stable GPUs. Also potential impact due to de-prioritising network topology in scheduling, but experimentation suggested the effect was tolerable. Reduction of leadership job failures by 30%.

* [FlipTracker: Understanding Natural Error Resilience in HPC Applications](papers/pap109s4-file1.pdf): automated analysis of application error resilience by injecting instruction-level bit errors. Definition of 6 resilient patterns, and demonstrated their use to improve resilience.

* [Lessons Learned from Memory Errors Observed Over the Lifetime of Cielo](papers/pap392s4-file1.pdf): Data analysis of memory errors over the entire lifespan of Cielo. Impressive dataset but more or less a "non result": did not find expected age-dependence, and did not show that correctable errors predicted future uncorrectable errors.

* [Partial Redundancy in HPC Systems with Non-Uniform Node Reliabilities](papers/pap381s4-file1.pdf): exploration of a middle-ground between checkpointing and replication (running two copies) for handling unreliable nodes. At high scale, checkpointing becomes less favorable than replication, but always one of them is preferred over partial replication (replicating subset of tasks). However existing work had assumed equal failure rates of all nodes, which is not borne out in practice. This paper demonstrated that in non-uniform failure rate cases, a (fairly obvious) partial replication scheme can give optimal expected completion time.

* [**Evaluating and Accelerating High-Fidelity Error Injection for HPC**](papers/pap386s4-file1.pdf): explored a mechanism of improving fidelity of error injection methods. We can inject at (from low to high): RTL (register transfer language), microarchitecture and instruction level. At the lowest level these are _extremely_ expensive, but high level approaches are too _unrealistic_. Their contribution is a heirarchical approach which examines how low-level RTL errors manifest at the instruction level (discarding RTL errors that are nullified by circuit masking). With this approach, they generate errors using what they describe as "nested" Monte Carlo. Offered in the [Hamartia Error Injection Suite](http://lph.ece.utexas.edu/users/hamartia/).

### [Doctoral Showcase](doctoral.md)

* In-Memory Accelerator Architectures for Machine Learning and Bioinformatics: Content-addressable-memory (CAM) has a grid of bits. Can execute search and write on grid positions matching a pattern. Memristors allow implementation of CAM with __much smaller bit size__. Demonstrated substantial gains for K-Means, K-nearest-neighbors, Smith-Waterman (similar subsquence finding), among other applicaitons in ML and computational biology. Claims that non-von-neuman architectures (_compute in memory_) will provide the next orders of magnitude improvements.

### [Large-Scale Algorithms](large-scale-algorithms.md)

* [Large-Scale Hierarchical K-Means for Heterogeneous Many-Core Supercomputers](paper/pap171s4-file1.pdf) This talk was delivered far to quickly in my opinion (used 15 minutes of the allowed 30), and I don't think I was the only one left a bit baffled. As far as I can tell, they gained a massive speedup from a new partitioning scheme.

* [**TriCore: Parallel Triangle Counting on GPUs**](papers/pap140s4-file1.pdf) Far better delivered talk than the above, and a concise tale of how a worse complexity algorithm can actually be better on GPU if it's more memory friendly. Triangle finding uses a sparse adjacency matrix representation: for each edge take the lists of adjacent vertices of each end, and look for repeats. Typically this is done with a merge-sort-like algorithm, but this performs poorly on GPU due to warp splitting. Another method for identifying repeats is to do many binary searches (one for every item in the smaller list), which can be more friendly for SIMD. As the search progresses the memory addresses diverge, but they show an optimization by keeping the top levels in shared memory.

* [Distributed-Memory Hierarchical Compression of Dense SPD Matrices](papers/pap141s4-file1.pdf) This talk went too deep to quickly. Due to missing context, I didn't take much away from this one.

### [Resource Management](resource-allocation.md)

* [RM-Replay: A High-Fidelity Tuning, Optimization and Exploration Tool for Resource Management](papers/pap360s4-file1.pdf) demonstrated a system that would run [SLURM](https://slurm.schedmd.com/) inside a container, and use a submitter system to replay jobs based on slurm database logs. Fake system clock was used, along with various patches to enable previous jobs to run in the containerized environment. This technique was far preferred to simulated enviroments as it actually uses the real scheduler and configuration.

* Both [Evaluation of an Interference-Free Node Allocation Policy on Fat-Tree Clusters](papers/pap541s4-file1.pdf) and [**Mitigating Inter-Job Interference Using Adaptive Flow-Aware Routing**](papers/pap311s4-file1.pdf) addressed the question of how to resolve communication bottlenecks in fat-tree networks. The first attempted to resolve it at the scheduler level, by isolating jobs to the switch or pod level (depending on size), along with heuristics to help avoid fragmentation. The second talk demonstrated that typical bottlenecks actually only affected a few links, and this could be addressed with runtime rerouting.

### [Task-based Programming](task-based-programming.md)

* [Dynamic Tracing: Memoization of Task Graphs for Dynamic Task-Based Runtimes](papers/pap490s4-file1.pdf) Presented a technique for replaying subgraphs of a task graph using dymanic tracing. Showed how to achieve similar performance to static graph execution but in the dynamic case. The [legion programming system](https://legion.stanford.edu/) looks worthy of further investigation.

* [Runtime-Assisted Cache Coherence Deactivation in Task Parallel Programs](papers/pap338s4-file1.pdf) presents a technique at the hardware-software boundary to improve performance by disabling cache coherence for certain data. In particular, in a task-based system we know the inputs to single functions will be private to the program, therefore do not need cache coherence. Specifically this modifies cache directores to add "non-coherent bit" and an additional hardware table containing address ranges which do not require coherence. The non-coherence table can be controlled from software, therefore can be integrated into a task-based runtime. Moreover, an adaptive directory approach was described which would save power by dynamically turning off unoccupied parts of the non-coherent addresses list.

* [A Divide and Conquer Algorithm for DAG Scheduling Under Power Constraints](pap547s4-file1.pdf) tackles the (theoretical) question of upper bounds on schedulers that must optimize for the DAG and power constraints. Optimizing for the DAG alone, greedy techniques will always be within 2x of optimal, and for power alone will always be within 3x. They have shown a join optimizer that must be within `O(log(n))` of optimal. At a high level, the approach optimizes for the DAG first, then performs a bin-packing-like algorithm to satisfy power constraints at the midpoint of execution, then recurses on the halfs, ... hence the `log(n)` component. They show substantial improvements on greedy schedulers under strict power constraints.

### [Arithmetic and Optimization](arithmetic.md)

* [**Associative Instruction Reordering to Alleviate Register Pressure**](papers/pap431s4-file1.pdf) You would think register allocation was solved by now, and it is for almost all general-purpose code. However this talk presents classes of large stencils where the register allocator fails to allocate optimally in arithmetically dense inner loops. Unnecessary register spills can cause high arithmetic density code to be memory bound. Typically this is because heuristics do not consider a long enough range of code. The paper presents some specific scheduling heuristics that perform far better on these types of arithmetically dense codes. Critically, they exploit associativity of arithmetic operators, which _can change floating point results_. Implemented as an [LLVM pre-pass called LARS](https://github.com/pssrawat/LARS).

* [**Harnessing GPU's Tensor Cores Fast FP16 Arithmetic to Speedup Mixed-Precision Iterative Refinement Solvers**](papers/pap464s4-file1.pdf) Excellent overview of mixed-precision floating point operators and tensor cores in modern GPUs. Demonstrated applicatons in traditional matrix algorithms. Specifically, how to use iterative approaches to gain `double`-precision solutions with tensor-core and `fp16` operators.

* [**ADAPT: Algorithmic Differentiation Applied to Floating-Point Precision Tuning**](papers/pap503s4-file1.pdf) tackles the question of which operations can be _safely_ replaced with lower precision floating point operations in order to achieve a certain tolerable error threshold. Uses a technique called Algorithmic Differentiation to effectively "differentiate a computer program" (for example [CoDiPack](https://github.com/SciCompKL/CoDiPack). This allows them to estimate the change in output by precision reduction of certain variables. Then a greedy approach is used, iteratively reducing precision. Relies on a dynamic trace. Demonstrated 1.2x speedup in LULESH, for example. Nice talk.

### [Programming Systems Tools](programming-tools.md)

* [**ParSy: Inspection and Transformation of Sparse Matrix Computations for Parallelism**](papers/pap256s4-file1.pdf) Representing sparse matrix operations (example Cholesky Decomposition) as a DAG to be scheduled. Presented a "H-level inspection" algorithm for optimizing DAG scheduling for parallelism _and_ locality. Depends on the pattern of sparsity byut not on the exact values. Significant speedups over Intel MKL, even when the optimization is done at runtime. Part of the [sympiler project](https://github.com/sympiler/sympiler).

* [Detecting MPI Usage Anomalies via Partial Program Symbolic Execution](papers/pap382s4-file1.pdf) "asan" for MPI. Symbolic execution for bug-finding. Partial execution (allowing jumping in at any point) enabled substantial speedups by avoiding irrelevant symbolic execution forks due to non-MPI code. However, jumping in at any point also required handling "lazy memory allocation", essentially `malloc`-ing "shadow memory" later if a pointer is dereferenced. Found bugs that existing solutions could not.

### [Deep Learning](deep-learning.md)

* [**Exploring Flexible Communications for Streamlining DNN Ensemble Training Pipelines**](papers/pap425s4-file1.pdf) Ensemble training is training muliple models on the same data. This talk described extensions to Uber's Horovod framework to allow for preprocessing work to be shared. Supported models are for all nodes to share preporcessing, or a subset. Tested on ORNL Titan.

* [**Anatomy of High-Performance Deep Learning Convolutions on SIMD Architectures**](papers/pap322s4-file1.pdf) ML Kernels look like FMA inside many layers of nested loops. Described multiple classical approaches to optimization: vectorization, register blocking, re-ordering for cache friendliness, prefetching. Arrived at an interesting "kernel streaming" abstraction.

### [File Systems: Data Movement and Provenance](storage.md)

* [Dac-Man: Data Change Management for Scientific Datasets on HPC Systems](pap407s4-file1.pdf) is a change management system for large datasets. Leverages HPC for creating and maintaining indicies. The most interesting aspect was the ability to describe changes of specific file formats, with builtin support for: matricies, HDF, csv, images.

### [Workshop: Extreme-Scale Programming Tools](tools.md) ([ESPT](https://www.vi-hps.org/front_content.php?idart=343))

* Advanced Event Sampling Support for PAPI. Not as much new here as was hoping. Discussed the PAPI interface to performance counters supplied by Intel PEBS. Asked for feedback on how to expose the Linux perf counter sampling in the PAPI API, either with "event type codes" or by differectly accepting a `perf_event_attr` struct.

* [**PARLOT: Efficient Whole-Program Call Tracing for HPC Applications**](https://pdfs.semanticscholar.org/21f8/596781a02b847612d95e14ef01100668f9b2.pdf)
  Described a whole-program binary tracing method using [Intel Pin](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool) for dynamic instrumentation and compression derived from the [CRUSHER framework](https://userweb.cs.txstate.edu/~mb92/papers/sc16.pdf). ([Related PhD Thesis](https://pdfs.semanticscholar.org/21f8/596781a02b847612d95e14ef01100668f9b2.pdf).)

* [**Gotcha: A Function-Wrapping Interface for HPC Tools**](https://github.com/LLNL/GOTCHA) Describes problems with traditional `LD_PRELOAD` approach: ABI compatibility, "all or nothing" replacement of a library, and also control at runtime rather than build time. Described the GOTCHA library offering a C library for wrapping, which modifies the "global offset table". Also allows for multiple libraries to hook into the same calls, with a priority mechanism.

## Birds of a Feather

* [**Distributed and Heterogeneous Programming in C++ for HPC
  2018**](bof-cpp-heterogeneous.md) C++20 will have a notion of an `executor`,
  something with an `execute()` method. This is the first step of a journey to
  support execution on a variety of backends. The new standard will also bring
  semaphores and atomic wait. SIMD library was also accepted so "You should
  never have to write an intrinsic again in your life." The SIMD library is a
  starting point: control flow is still a lot of work for the programmer, for
  example remainder loops are still not handled. Discussion on the `mdspan`
  abstraction of multi-dimensional array layout which specifies how a
  multi-dimensional index is mapped to a 1-dimensional memory address. Kokkos
  has introduced this concept allowing a compile-time choice of data layout,
  as well as "views" into data.

* [The ARM HPC Experience: From Testbeds to Exascale](bof-arm.md) Recent
  launch of
  [Astra](https://www.top500.org/news/sandia-to-install-first-petascale-supercomputer-powered-by-arm-processors/),
  the first petascale ARM supercomputer. Discussion of
  [Isambard](http://gw4.ac.uk/isambard/) supercomputer at University of
  Bristol. Broadly: favorable performance/power comparison to server-class
  x86, but much better _performance for price_. Challenges with memory
  bandwidth and compiler immaturity (bugs and slow compilation times). Cloud
  adoption still poor, but hoping increased adoption will drive down costs for
  HPC too.

* [Achieving Performance on Large-Scale Intel Xeon-Based Systems](bof-xeon-scale.md) Extremely limited notes, as I was only able to catch the very end.

## Followups

* Parallelism abstractions: [RAJA](https://github.com/LLNL/RAJA), [kokkos](https://github.com/kokkos/kokkos) ([example](https://github.com/UoB-HPC/advanced-hpc-examples/blob/master/Kokkos/vecadd-kokkos.cpp)), [SYCL](https://www.khronos.org/registry/SYCL/specs/sycl-1.2.pdf)
* ARM: NEON and
  [SVE](https://developer.arm.com/docs/dui0965/latest/sve-overview/introducing-sve),
  [SLEEF](https://github.com/shibatch/sleef), AVVPCS, [Cavium](https://cavium.com/)
* [`bfloat16`](https://developer.arm.com/docs/dui0965/latest/sve-overview/introducing-sve) (generally `int8`, `fp16` in ML)
* Projects: [atJIT](https://github.com/kavon/atJIT), [OP2](https://op-dsl.github.io/), [PInT](https://github.com/RWTH-HPC/PInT), [TraceR](https://github.com/LLNL/TraceR), [legion](https://github.com/StanfordLegion/legion), [regent](http://regent-lang.org/), [LARS](https://github.com/pssrawat/LARS), Tensor cores, [CoDiPack](https://github.com/SciCompKL/CoDiPack), [tapenade](http://tapenade.inria.fr:8080/tapenade/index.jsp), [klee](https://klee.github.io/), [Caliper](https://github.com/LLNL/Caliper)
* http://tacc.github.io/SDVis/ https://sites.utexas.edu/jdm4372/
* [x86 machine check architecture](https://en.wikipedia.org/wiki/Machine_Check_Architecture)
* [LIBXSMM](https://github.com/hfp/libxsmm)
* Intel PEBS, [PAPI](http://icl.utk.edu/papi/), Caliper, [Darshan](https://www.mcs.anl.gov/research/projects/darshan/)

