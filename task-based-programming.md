## Dynamic Tracing: Memoization of Task Graphs for Dynamic Task-Based Runtimes

Transiton from dependence analysis to subgraph replay.

Introduce fence.

Fence not enough. Need to remember preconditions for subgraph replay.

If we replay that subgraph, subcomponents of subgraph are now unknown by task
dependence (they didn't actually run, they were replaced by an). Solve this by
adding a "dummy" summary node that makes it look like they did.

Graph language called "graph calculus":

* Events are edges...
* "op"
* "merge" names into a single event
* "fence"

### Optimizations

Idempotent recordings:

* When postcondion subsumes precondition.

Fence elision:

...

### Summary

Dynamic tracing brings benefits of explicit task graph construction to dynamic
graphs.


## Runtime-Assisted Cache Coherence Deactivation in Task Parallel Programs

Scaling cache coherence to large core counts is hard.

Runtime assisted approach to deactivate cache coherence in task-based
programming models.


### Architectures

* Shared main memory + caches.
* Successful due to programability provided by hardware cache coherence.

Directory based cache is the most scalable type. But requires meta-information
for.

- K bits per entry: metadata state and owner
- L entries

### Optimizing K

Identify data that does not need coherence.

- Private
- Read-only
- Temporarily private

Don't track these in the directory.

### How do we deactivate coherence

Page table:

- Monitor page faults and mark pages as private/public based on which cores
  they come from
- But no temporal aspect.

TLB temporal aware:

- TLB miss check all TLBs and classify page as private if its only in local TLB.
- Needs complex support.

### Task-based Dataflow Programming Model

Model computation as discrete tasks, explicit input outputs.

Task dependencies **do not need coherence during execution of the task**.

### Runtime approach

Can identify 79% of all potentential cache coherence deactivations.

Compared with 29% for TLB-based approach.

### How does it work?

Hardware/software codesign.

1. Before task, runtime comminicates address range of tasks dependencies.
2. During task execution microarch deactivates coherence.
3. On completion, re-enables coherence for the address range.

Added RACT activate/deactivate instructions to ISA.

### Hardware

L1D-cache add a non-coherent bit (NC) per entry in the cache.

Also adds a non-coherent region table (NCRT), holds start and end addresses for
non-coherent ranges.

If NCRT becomes full, normal behavior resumes.

Accessed on a cache miss, in which case it needs to know if that address needs
to be coherent or not (NC bit set).

### Recovery

Runtime issues `ract_invalidate` on task finish.

HW flushes cached lines. Now continue as usual.

### Adaptive Hardware

This doesn't make as much sense for regular programming models.

So make the cache directory modifiable, so it can be made larger/smaller.
Parts can be powered off. Thresholds on occupancy govern when it us
grown/shrunk. Recofigurations can be expensive though.

"Adaptive Directory Reduction"

### Evaluation

Evaluated at various (simulated) cache directory sizes.

In some benchmarks, RaCCT massively reduces performance hits at lower
directory sizes.

### Conclusion

Small changes in SW/HW, but very effective.


## A Divide and Conquer Algorithm for DAG Scheduling Under Power Constraints

Have a set of tasks. Ordering constraints represented as a DAG.

Need to schedule it on a parallel computer.

Increasingly have power constraints.

Make the scheduler aware of the power constraints?

Early work on this has been heuristics. Can we get an optimal solution in the
case of DAGs?

### Existing Work

DAG scheduling:

* greedy is within 2x of optimal ['66]

Optimal is NP hard. We'd need another supercomputer to schedule!

Power constrained scheduling:

* Greedy is within 3x of optimal.

Pack tasks into a power budget. Like bin-packing.

What happens when we combine these?  No formal bounds known.

### Result

Within O(log(n)) of optimal using a divide and conquer technique.

### How

n tasks on m parallel machines to minimize runtime.

Constraints:

* DAG gives ordering, task finish time must be less than next task start time
* At any time step, total power has to be less than a limit

It's like bin packing on a timeline.

1. Start with a schedule that satsfies the DAG, ignoring the power limit.
2. Look at the midpoint, see what's running. Now (greedily) schedule these to
   satisfy the power constraint.
3. Recurse to each half.

Showed we incur a constant factor. Divide and conquer to `O(log(n))`.

### Greedy Approaches

* G1: fist in, first out
* G2: best fit to power budget
* G3: non-increasng runtime order

Tried on simulator and compare Divide and Conquer.

* Strict power caps, divide and conquer wins by a large margin.
* As the power cap is relaxed, starts to lose out to greedy.

Why?

* Higher power utilization
* Aligns similar run-time tasks in teh same batch


