## In-Memory Accelerator Architectures for Machine Learning and Bioinformatics

Motivation: overcome Von Neumann Bottleneck

* Mem <-> CPU/GPU
* Memory transfer is majpr bottleneck
* Compute **in** memory

## Content Addressable Memory

Grid of bits.

What can we do?

* Apply a pattern: search
* Write to matching patterns.
* For example: lookup table.

Example: associative vector addition.

**Takeaway:** fixed number of cycles regardless of data size.

## Memristors

Change resistance with applied voltage.

Not volatile.

Small footprint.

Allow **much smaller bit size** in content-addressable-memory (CAM).

## Smith-Waterman Algorithm

Dynamic programming

O(n^2)

Finds longest "similar" subseqnece within 2 given sequences.

## ML Algorithms

K-Means.

Extended ReCAM architecture to have a reduction tree on the bit array.

Orders of magnitude speedup by power efficiency.

Similar speedups for K-nearest-neighbors.

## Batch write NAND ReCAM

Allow batch associative processing writes.

Allows full-adder... not just single bit ops.

Faster Geonome Assembly.


...

## Conclusion

Non von neuman arch can provide the next orders of magnitude improvements.
