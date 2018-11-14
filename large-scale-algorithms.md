# Large Scale Algorithms

## Large-Scale Hierarchical K-Means for Heterogeneous Many-Core Supercomputers

### State of the Art

Parallel

* Parallel K-Means
* Up to 512 centroids

Two-level K-means:

* Partition with Shared Memory

### Problems

n points
d dimensions
k centroids

n*d and n*k limited by shared memory size

Not possible to have large scale dimensions and

### Goal

Fully partitioned method.

Target Sunway TaihuLight.

3rd Place in Top500

40k processors, 10M RISC cores

Heterogenous Heirarchical hardware

### Approach

Multi-level data partition.

...



### Conclusion

First ever 3-level fully partitioned.

Multi-level design for heirarchical architectures.


## TriCore: Parallel Triangle Counting on GPUs

Triangle counting in graphs.

### Triangle Counting

Applications in social media networks.

Spam detection.

### Algorithm

Three nodes connected to each other.

Use sparse adjacency matrix, storing the non-zero entries.

* For each edge.
* ... compare neighbors of both verticies.
* Find shared neighbors.
* Every shared neightbor is a triangle.

### Characteristics

O(E^2) operations and memory accesses.

Heavy memory access.

### Why GPUs?

* Many core.
* High **memory bandwidth**.

### Challenges

SIMD: warps execute same instructions

- Warp = 32 threads
- without SIMD, can get low thread utilization

Small cache:

- low data utilization

### List Intersection

Example: Merge-based.

Effectively merge sort, but only output repeats.

O(m+n) where the list lengths are m, n

Not good on GPU.

### Techniques

1. Binary search based

2. Shared memory caching

### Binary search-based

Given two lists A and B.

- Suppose A is the shorter one.
- B is sorted

For every element of A: binary search to find it in B.

On GPU:

* Each interation is 32 lookups in parallel.
* Fine-grained GPU paralellism.
* Much better memory performance.

Now... as the search progresses the 32 searches will access different areas of
memory.

They diverge more and more as the search goes on.

**Top levels are cached in shared memory**

### Why is this better than merge approach?

Sizes m and n.

When small m and big n:

```
m * log(n) < m + n
```

When both big, the memory access pattern still makes the GPU version faster.

### Scaling

* Edge list streaming buffer.

* Multi-node typically degrades performance. Want to fix this.

### Edge streaming

Ring buffer to load edge list iteratively. (?)

Buffer size needs to be large enough to saturate the GPU.

Load balance across multiple GPUs with **work stealing algorithm**.

## Also distributed MPI version

### Experiments

- CPU 16-48 core
- K40 and Titan X Pascal

Graphs up to 30B edges.

- vs. GPU work: 5.5x om K40, 2.2x on Titan X
- vs. CPU work: 2.5x vs Shun

Compared with 2017 graph challenge: 2.7x and 1.9x speedups.

Scalability:

* 4.4x on 6 GPUs
* 24x on 32 GPUs


## Distributed-Memory Hierarchical Compression of Dense SPD Matrices

Contributions:

* Data distribution
* Efficient distributed tree-based algorithms
* Runtime dependency analysis, task scheduling
* Async MATVEC

Goal of GOFMM is to decompose M as

```
(block diagonal) + (low-rank sparse matrix)
```

Recurses on sub-matricies.

Creates a binary tree of computation.


... lost the thread of this talk ...
