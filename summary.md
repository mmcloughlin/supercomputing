# Summary

## Followups

* Parallelism abstractions: [RAJA](https://github.com/LLNL/RAJA), [kokkos](https://github.com/kokkos/kokkos), [SYCL](https://www.khronos.org/registry/SYCL/specs/sycl-1.2.pdf)
* ARM NEON + SVE. [SLEEF](https://github.com/shibatch/sleef). AVVPCS
* Projects: https://github.com/kavon/atJIT, https://op-dsl.github.io/, https://github.com/RWTH-HPC/PInT
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
