## Advanced Event Sampling

PAPI is an API for performance counters.

Extreme scale? The bigger the machine, harder it is to gain insights.

### Current Counters

CPU counters:

* Cycles, instructions
* Cache hit/miss
* Branch predictors

* Typically aggregate counts

### Sampling

Target single program.

Periodically log performance info while running.

Statistically extrapolate to get entire program behavior.

Tradeoff between sampling rate.

### Example

Counter to overflow every 100,000 cycles.

Most hardware supports this.

Can be done with a timer instead.

### Advanced Features

Intel Precise Event-Based Sampling (PEBS)

AMD IBS.

### Low Latency Sampling

Hardware can write out data for you.

When the buffer is full, OS notifies you to handle it.

Intel PEBS supports. AMD does not.

### Hardware Profiling

* Regular intervals, random instructions are sampled and details logged.
* Register states.
* Transactional memory info.
* Load and stores
* TLB info

### Low-Skid Interrupts

Modern out-of-order processors take time to stop to handle interrupts.

Can lead to results being offset.

Low-skid support returns exact cause of overflow.

### Last branch sampling

Intel LBR (last branch record)

* Info on last 4 to 32 branches
* Basic block vectors.

### Intel PT

Record program execution traces.

* Power events
* Basic block vectors

Sizes are prohibitive.

### Linux perf_event

Support for most but not all.

Low latency sampling from PEBS.

Low-skid interrupts: set the "precise" value.

### New PAPI Interface?

Abstract everything to one "sample type" field.

If `perf_event` changes, may have to change the PAPI interface.

_Or_, could expose the `perf_event_attr` struct in PAPI. But...

1. Perf event is complex.
2. PAPI is supposed to be cross-platform.

First interface was chosen.

### Future

PAPI will output to `perf.data`. But, not well documented.

May try the lower-level interface too.


## ParLOT

Whole-program call tracing.

Mostly for debugging.

Trade-off between overhead and visibility.

Method:

* Binary instrumentation
* Compression

### Debugging

HPC bugs are **expensive**

### Goals

Always on tracing

No source-code modification

No recompilation

Dynamic instrumentation

Portability

Low overhead (runtime and storage)

### Contribution

* Use Pin, dynamic binary instrumentation tool.
* Incremental compression tool.

### Binary Instrumentation

* ParLOT:
  - Every thread launch/terminate
  - FUnction entry/exit
* Separate file for each thread.

Per thread:

* TID
* Function ID
* Call stack
* Current SP value

### Incremental Compression

COnventional approaches

* Written to buffer
* Sporadiacally thread blocks to compress - non-uniform latency

* Every trace element is compressed before writing to memory
* Predictable latency

### Compression

Used CRUSHER:

* Auto compression algorithm synthesis tool
* LZ followed by ZE
* High compression / low overhead

Trace entries:

* LZ is word level
* ZE is byte level

### Call-stack correction

PIN sometimes does not identify function exit.

Solution:

* Record stack-pointer
* Pop call stack until find a match

### Evaluation

Measured:

* Tracing overhead
* Tracing bandwidth
* Compression ratio

Compared with callgrind.

Compression massively reduces overhead (factor of 4).

But PIN init is still massive overhead.

Question mentioned some old work: using sequeter(?)

## Function Wrapping for HPC Tools

GOTCHA

Replacement for LD_PRELOAD

### Why wrapping?

Example of wrapping `MPI_Send()` to log extra information.

### LD_PRELOAD

Write a shared library matching functions they want to wrap.

Set LD_PRELOAD and run the application.

Dynamic linker prioritized.

### Issues

* ABI Compatibility
* "All or nothing". Has to be the entire library.
* Controlled by the person running the applicaiton, not the one running it.

Need a replacement.

### Elements

1. Name of function to wrap.
2. Wrapper function.
3. Way to call the original.

### GOTCHA

Has a C interface.

How does it work?

Libs have a global offset table. These structures are exposed and can be
modified.

### Multi-tool Support

GOTCHA supports multiple wrappers.

Can have priority, if order matters.

### Build

You can always build it into your application, even if you don't use it.

