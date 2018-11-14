# Arithmetic

## Associative Instruction Reordering to Alleviate Register Pressure

Register management considered satisfactorally solved in GCC and Clang.

* Mapping of data to hardware registers
* Instruction reordering to minimize spills

Good for __most__ programs. But, not for some edge cases. For example:

* COmplex many-to-many reuse patters are not well handled.
* LLVM "blows up"

Heuristics not always right.

### Contribution

LARS: list-based adaptive register pressure aware scheduler

* Exploits associativity for reording accumulative updates
* At this point, source-to-source transform
* Significantly reduced spills

### Examples

Conviolution stencil

(see paper!)

* Double loop
* 9 inputs
* 17 flops per point
* 16 bytes per point
* 1.06 FLOPs per byte

Memory bandwidth limited on most processors.

But, on a __higher order stencil__ will get a much higher FLOPs per byte.
Should be compute bound.

But... register allocator fails, we get spills, and it becomes bandwidth
bound.

(Example from SW4)

Many-to-many. Each one uses in many places. When unrolled, complex reuses
**baffle standard schemes**.

Theoretically 179 bytes memory access per loop but in gc get 600+.

### Observations

Compilers focus on local register pressure.

Compilers failt o perform pattern specific reordering to reduce pressure.

### Solution

"Pattern specific source-to-source transform"

* Lower to (some kind of) IR
* Leverage associativity

Then pass to the compiler, and the allocator will handle it much better.

* List-based scheduler.
* Starts with instructions not dependent on anything.
* Live set maintained: instructions whose operands are in registers.
* Heuristics to prioritize live set.

Instructions have state:

* blocked: not all oeprands available
* unblocked: not blocked
* fireable: unblocked, and all labels live

How to prioritize:

1. Its ability to release other labels. Prioritize by how many values can be removed from registers after this has been executed.
2. How many instructions will become fireable.
3. ALso some affinity score (less clear on this)

Produces cost tuple.

### Algorithm

Custom-sort the cost tuple. Greedy schedule.

Initial priority based on critical path.

### Evaluation

* Original
* Unrolled
* LARS

Large improvement for stencil code.

### Summary

Better performance by

* Disabling pre-pass scheduling
* Replacing it with custom cost-based instruction scheduler


## Harnessing GPU's Tensor Cores Fast FP16 Arithmetic to Speedup Mixed-Precision Iterative Refinement Solvers

Deep Learning is a thing! In case you haven't been paying attention.

**Small Matrix Operations**

Can get away with 16-bit floating point (fp16). Even int8.

Hardware vendors investing in half-precision.

Google has `bfloat16` (not IEEE).

### Formats

IEEE half precision is 5 bit exponent and 10-bit fraction.

Google is 8-bit exponent, 7-bit fraction (bfloat16). Larger range, lower
precision.

### Volta / Tensor Core

Tensor core: 4x4 mixed-precision matrix multiply.

Tensor core fp16: 120 Tflops.

* Product
* Accumulator (32-bit)
* Convert

^ designed for linear algebra, eg dot products

### LU Decomposition

Modern algorithms do "panel methods".

* Factor panel, apply to whole matrix.
* Application is a matrix multiply.

"If we can speed up matrix multiply, we can speed up LU."

Speed:

* fp16 is approx 4x double precision
* tensor core is 16x
* (for square matrices)

### Leveraging

* Bulk of the work in 16-bit tensor cores.
* Improve the accuracy later.
* Mixed precision.

"Iterative refinement" https://en.wikipedia.org/wiki/Iterative_refinement

* Compute solution in lower precision. Residual is r = b - Ax (double
  precision)
* Solve for Az = r, again at lower precision. (reuse LU factors)
* Iterate.

Improvement: use another method to solve the iterative (residual) problems.

### Performance

4x faster than double (even accounting for the iterations)

However, there are cases where lower-precision iterative approaches fail to
converge.


## ADAPT: Algorithmic Differentiation Applied to Floating-Point Precision Tuning

Example: example of arithmetic error in a missile guidance system (patriot)!

These errors are common in HPC as well. How do we know if we're getting the
right answer?

## Goal

Understand impact of these errors.

IEEE: 32-bit single, 64-bit double, 128-bit quad, 16-bit half

Usually programmers just pick the highest.

Use the lowest necessary to get the accuracy we need.

Mixed-precision: multiple levels in the same program.

### Contribution

Tools to idenfiy variables which need high precision, to guide decision of which variables should be mixed-precision version.

### Existing Work

Automatically discover unstable ops and apply transforms.

Search-based approaches which try multiple precisions. But the state space is
huge.

Rigorous error analysis. Do not scale well.

### How does output change?

View entire program as a function f(x).

**Differentiate a computer program.**

Use algorithmic differential to estimate derivative of f'(x).

View program as a sequence of operations. Apply chain rule.

Numerous AD tools available: CoDiPack, Tapenade.

### Mixed-precision

Estimate error due to lowering precision.

Aggregate the error over all dynamic instances of the variable.

Greedy approach: switch variable with the least impact on 

### Evaluation

- LULESH: 1.2x speedup

### Limitations

* Based on inputs used. Needs to be represenative.
* Control flow divergence: 
* Memory requirements (changing to higer precision could use more)
* Overhead of type cast operations: not considered right now

### Future Work

* Automating the source-to-source transform.

### Summary

* Use algorithmic differentiation to estimate error incurred by reduced
  precision.
* Scales better than previous work.
