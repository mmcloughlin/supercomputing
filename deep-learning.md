# Deep Learning

## Exploring Flexible Communications for Streamlining DNN Ensemble Training Pipelines

### Ensemble Training

Group of DNNs training on same dataset. Why?

* Parameter tuning, for example
* Multiple predictors, combine votes for better accuracy

### Open Questions

What parts can be optimized? Minimize training times for large ensembles.

How to minimize trainign time.

Prior work: training large single models. Not ensembles.

### Pipeline

Generally, there's loading, preprocessing and training.

At it's simplest, we just run one pipeline per model.

Problems:

* Can redundant data reads cause an IO bottleneck?
* Didn't observe problems (on Titan). Probably filesystem caching.
* But, duped preprocessing can cause a problem.

Preprocessing (for images):

* random bounding nboxes
* resizing
* cropping
* normalization
* shifting
* random distortions

On Titan, 16-core CPU not fast enough to preprocess for the attached GPU.

**For ensembles, the preprocessing is redundant.**

### Benefits

* Lower CPU usage.
* Since pre-processing can be a bottleneck, can get an overall speedup.

### Challenges

Combining MPI with DNN frameworks (tensorflow).

Horovod (Uber!) is tensorflow adding that allows better scalability in
distributed DNN training.

They contributed Horovod groups. https://github.com/rbpittman/horovod

### All-Shared Pipeline

All run preprocess operation. Then __all-gather__ operation to share.

Or, __single-broadcast__ to broadcast data from single node. (but potential
bottleneck on single node). (one runs preprocess operaton)

So... __Multi-Broadcast__ fixes this problem. Subset of nodes do preprocessing
and multi-broadcast to training nodes.

### CPU Usage

(various experiments)

Usually found All-Shared to be best. That may be different for a system where
all nodes performing IO is inefficient (relying on filesystem caching, large
network).

### Conclusion

* Identified pre-processing bottleneck in ensemble training
* Extended Horovod with MPI sharing mechansim
* Significant gains in CPU usage



## Anatomy of High-Performance Deep Learning Convolutions on SIMD Architectures

CNNs are everywhere! Why?

Often to process images. Nearby pixels have correlation.  Often use feature
detectors.

### Tensors

`Input tensors` convolved with `weights tensors` -> `output tensors`

**Massively nested loop around a accumulated FMA.**

### Implementations

Flattening input data `im2col` operations and call GEMM.

+ user GEMM libs

cons:

* memory footprint overhead
* mem bandwidth

This work tries "direct convolutions".

### Vectorization & "Register Blocking"

Block two loops. Vecorize FMA.

Register blocking on two more loops.

### Cache Blocking and Loop Ordering

Cache blocking in outer loops. Min DRAM traffic.

Change loop ordering for data reuse.

### Convolution Microkernel

Inner loops are a small GEMM: convolutional microkernel.

* Load full vector from weights.
* Broadcast cirrent pixel and multiply with current loaded weights.

Also noted further loops could be incorporated into the microkernel.

### Software Prefetching

Prefetch sub-tensor from weight tensor.

... others

### Convolution Microkernel & Microarch

* RBp RBq depends on convolution
* VLEN depends on architecture

So can't have one version of the microkernel.

**JIT code generation with LIBXSMM**

### Layer Fusion

Modern DNN architectures have laters like ReLI, Bias, Batchnorm.

Fuse layers to maximise data reuse...

### "Kernel Streams"

Think of program as a stream of kernels to be executed.

Each one can take a list of tensors to prefetch (part of interface)

But then we can also "run length encode" the kernel stream.

The "kernel stream" is recorded in the __dryrun phase__.

__Replay phase__: runtime replay of recorded kernels.

### Propagation Order

The above is all for forward propagation.

Training is backward propagation.

Same techniques apply.

### Evaluation

Lightweight sibling of tensorflow (GxM).

Skylake arch.

... results were massive and obfuscated ...

