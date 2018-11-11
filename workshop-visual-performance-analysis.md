# Workshop: Visual Performance Analysis

Speaker: current director of ParaTools

https://vpa18.github.io/

## Keynote

### Intro

  Parallel Performance Engineering

  It's a process:

  Improve knowledge of performance
  Only then can we optimize
  Improve performance problem solving


### Visualization

  "Graphics reveal the analysis"

  Not just because it's pretty!

### Eras

  Observability (1991-1998)

    Accurately capture information about parallel software performance.

    Balance the *need* with the *cost* of obtaining the information.

  Diagnosis

  ...

  Productivity

### Evolution

  HyperView - visualizing hypercubes

  Traceview - trace visualization

    "view" on the trace
    "displays" against the view

  ParaGraph

    http://www.netlib.org/paragraph/manual.ps

### Usability

  Towards the end of this era, more of a focus on usability.

  "What to Draw? When to Draw? An Essay on Parallel Program Visualization",
    ftp://ftp.cs.wisc.edu/paradyn/technical_papers/visualize.pdf

  "Angry fruit salad"

  Abstraction:

  - Performance analysis abstractions
  - "Views" of the data
  - Combined with performance data displays

  Led to

  - Pablo
  - Vampir
  - TAU performance system started

  Starting to develop abstractions for performance data.


### Era: Diagnosis (1998-2007)

  Not just observation. Make "meaning" out of it.

  Help to explain what we're seeing.

  Automated analysis.

  Example:

    Semantic connection beyween low level compiled code and the original
    source/operations, to visualize which parts of execution correspond to
    high-level computations.

  Data mining:

    PerfExplorer tool
    https://www.cs.uoregon.edu/research/tau/docs/perfexplorer/

### Era: Complexity (2008-2012)

  Need to correlate with *many* different factors.

  Greater core counts.
  Heirarchical memories.
  Heterogeneous computing. Accelerators.

  Example: need to integrate CPU/GPU profiling

### Era: Productivity (2012-Present)

  HPC should be useful!

  Much greater core counts.
  Exotic programming methodologies.

  ALPINE: in-situ performance analysis

  Apex: Autonomic Performance Evaluation

    Online performance analysis
    Dynamic data capture depending on performance
    Capturing performance anomalies

### Thoughts

  Advances are going to come from automated analysis

  "Analysis determines visualization"

## Visualizing Calling Context Trees (CCTs)

Stack trace provides very useful context.

How does this work in HPC.

- Call tree notes O(1k-10k)
- Processes / threads O(10k)
- Iterations O(1k)
- Runs

Tools for a large number of large trees.

### Classic

  Score-P / Cube

    Uses file-explorer-like tree view.

    Problems: all uniform

  HPCToolkit

    Callers view (icicle)

  Flamegraph / Icicle Graph

  Ring View

    "Sunburst plot"

  Sankey Diagrams

    https://en.wikipedia.org/wiki/Sankey_diagram

### How to Handle Large CCTs

Operations

- filter: remove unimportant nodes

  filtering is easy, but deciding what to filter is not

- grouping:

  for example, group functions into libraries
  might turn your tree into a graph (multiple parents)

Data overlay:

- multiple metrics to visualize

Comparison:

- Matching
- Union
- Subtraction

### Prototypes

[Demo]

Multiple interacting views.

### Alternative Function View

Emphasize functions based on how much time is spent there on their own.

But still maintain call graph.

Color edges by how much of total time is accounted for by that caller.

### Summary

Claims gap in the space here.


## Using Deep Learning for Automated Communication Pattern Characterization

### Background

Unfamiliar application, want to explain its communication pattern.

Trace is like "drinking from a firehose" here.

Feed it into a tool and have it give a concise description of communication.

Inspiration: Astonomy, Sky Subtraction. Take away the stuff you know.

### AChax: Components

Library of known communication patterns + search engine.

Collected communication pattern.

Now, describe what we have with a tree.

- Start with the comms pattern.
- Identify one we know about.
- Subtract it, that's a child node.
- Recurse.

### Known problems

- greedy matching
- not resilient to single missed links, non-exact pattern matching

Discussion:

- can we use deep learning here?

## Trend Visualizer for Scalability

No easy way to exascale

* Larger number of cores total, and cores per node
* Larger performance penalties for
  - sync
  - comms
* Need developer tools to expose reasons for performance loss
  - simulators
  - debuggers
  - profilers

### Profilers

Focussed on *single-execution* on fixed number of cores, input size,
frequency, dataset.

How can we generate data useful to exascale?

Idea:

- take smaller amounts of data from many runs.
- visualize variability

Parallel Scalability Viewer (PaScal)

- https://www.semanticscholar.org/paper/PaScal-a-new-parallel-and-scalable-server-IO-for-in-Grider-Chen/332eb59e53607a9ec3d4facd6e65a06ec463f2cf
