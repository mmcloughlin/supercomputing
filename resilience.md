# Resilience

## GPU Resilience in Titan

In Titan, had ~8500

NVIDIA model:

* Position in rack (top get warmer air)
* Position in

This informs which ones to replace.

## "Leadership" jobs are Affected more than others

Stabilization to 10-30 failures per week.

55-65% of application failures in leadership jobs. These are taking > 20% of
the machine, so not surprising that they 

Leadership jobs:

* Run longer (24 hours)
* Run larger (20+% of machine)
* Wait longer (days/weeks)

## Influence Scheduling

* Increase use of stable GPUs in Leadership Jobs

## Current Algorithm

* Moab First Fit: top down, low index nodes used first
* Alps traditionally orders for tight network clustering (hilbert curve)

## Tweaks

* Stable GPUs in low indexes
* Maintaining Hilbert Curve as much as possible (?)

Also:

* Note smaller jobs are less impacted by failure
* Therefore give shorter jobs less stable GPUs.

## Evaluation

* Use Moab simulator.
* Trace of applications.
* Buckets of jobs that took 20%, 40%, 60% of machine.

Observations:

* Unintended side-effects: when multiple large jobs are running, one of them
  will take all stable GPU resources.

## CPU Only jobs.

Further separated CPU-only jobs, which do not require stable GPUs. They don't
require GPUs at all!

## Unintended Consequences?

This change makes the network topology of jobs worse.

Used software acceptance suite (representative of Titan workloads) to evaluate
difference between:

* Sole use of the machine
* Performance when scheduled by the new algorithm.

Acceptable performance hit.

## Summary

Even after initial fixes, still had higher than expected failure rate.

Modified scheduling to prioritize stable GPUs for high-priority jobs.

Reduction of Leadership Job Failures by 30%.


# FlipTracker

## Soft Errors

* Danger of soft errors increases as HPC systems scale.
* Cannot predict them. Electronic noise. Flip bits in storage.

## Application Natural Resilience

> Capability of Application to Tolerate Soft Errors

Capability is natural or inherent.

"Natural" means no changes are required.

Previous studies have shown many applications with this properties:

* Multi-grid solvers
* Monte Carlo
* Deep Learning
* Clustering

No framework to determine which applications are

## Goal

Break down code in to code-regions.

Framework for fine-grained analysis of

* error propogation
* failt tolerance

### Fault Model

Random fault injection, to mimic effect of real soft errors.

Define three possible error manifestations:

* Success
* Silent error
* Interupt / crash

### Regions

Identify code regions.

Classify them as resilient and non-resilient.

For each region:

* Generate clean instruction trace.
* Then inject faults, to get dirty instruction traces.

Generate dynamic data dependence graph (DDDG) for both.

* If they agree, it's a resilient code section.

### Dynamic Data Depenedence Graph

Helps identify the input and output variables of a code region.

### Resilient Regions

Considered resilient if:

* Value is equal
* Or the error magnitude is reduced by the computation.

## ACLs: Alive Corrupted Locations

Liveness analysis of variables.

Track the region of the code where the value has an error in it.

Graph of the number of alive corrupted locations over time.

### Patterns in Resilient Code

Pattern 1: Dead Corrupted Locations

  Just not used any more.

Pattern 2: Repeated Addition

  Through repeated addition, the error magnitude decreases.

Pattern 3: Conditional

Pattern 4: Shifting

  Errors but shifted out.

Pattern 5: Truncation

  Corrupted data truncated

Pattern 6: Overwriting

### Applying Resilience Patterns

Example of applying "Dead Corrupted Locations", overwriting, Truncation:

  Overall 32% improvement in resilience

### Conclusions

Designed framework that enables fine-grained analysis of error propagation to
capture application resilience.

Developed library of patterns that can help with resiliency.


## Lessons Learned from Memory Errors Observed Over the Lifetime of Cielo

Cielo: 8,500 nodes. chipkill-correct ECC

System logs:

* Resorce management logs
* Memory error logs (x86 machine check architecture)
* Kernel panic logs
* Hardware inventory logs

### Caveats

Talk about __faults__ not __errors__. Fault is an underlying problem, errors
are manifestations. One fault could cause many errors.

Manufacturers anonymized.

### Manufacturers

ALmost entirely static proportion across 3 manufacturers throughout the
lifetime of the machine.

### Root Causes

Node down events:

* ~25% memory errors
* Some kernel panics
* Approx two thirds unknown

### Independence of Uncorrectable Memory Faults

Compare log-plot-ratio of intervals between memory faults.

Based on faults by node, actual distribution close to predicted.

QQ Plots to fit distribution.

Exponential, Weibull, Gamma

Not good for DRAM, much better for SRAM.

[Bayesian Information Criteria](https://en.wikipedia.org/wiki/Bayesian_information_criterion) for choosing between distribitions.

Selected Weibull.

### Reliability over Time

No clear relationship.

But... 2012 had 2014 had abnormal spikes in DRAM uncorrectable failures.
Not sure why, but previous analysis suggests it's within the realm

### Correctable Faults

Doesn't appear to foretell uncorrectable faults.

### Conclusion

* No discernable aging... surprising
* Correctable faults not predictive of uncorrectable faults
* Challenges involved in reconciling disparate data sources


## Partial Redundancy in HPC Systems with Non-Uniform Node Reliabilities

Checkpointing, or process replication?

As scale increases: higher failure rates, more checkpoints, more repeated work.

At some scale point, replication is better.

Is there something inbetween? Yes, partial replication.

Distribute across 70%. Replicating __some__ of the other nodes in the other
30%.

```
replication factor = (# total nodes) / (# applciation visible nodes)
```

Naive replication is 2x replication factor. Partial replication is between 1
and 2.

### Contribution

Optimizing for time to failure always converges to one of the extremes:
replicaiton factor 1 or 2.

But... assumption in all fault tolerant schemes has been that all nodes fail
at the same rate.

Is that a fair assumption? But... "the spatial distdibution of failures is not
uniform at any compute granularity across systems"

Not true.

Modify this assumption:

* Failures are still independent.
* But failure rates are not the same.

Does partial replication "pay off" with this new assumption. They decide:

* Which nodes to replicate
* How to form replica groups
* Does partial replication pay off?

### Method

Given fixed replication ratio.

1. Replicate least reliable nodes.
2. Pair them with node of opposite reliability: pair least reliable with most
   reliable.

Note: doesn't rely on actual reliability values,just ordering.

### Does it "pay off"

For that, we need to know the factor of r.

Model completion time with value of r.

Form optimization problem: linear programming.

In toy examples, we do see cases where values of r between 1 and 2 are
optimal.


## Evaluating and Accelerating High-Fidelity Error Injection for HPC

High fidelity error injection is too slow.

* RTL level: Months per application.
* Microarch level

Instruction level: more reasonable.

In this talk:

* When do we need to model at hardware level.

### Existing Models

Single-event upsets that effect execution units. Directly effect application data.

Hidden soft errors (inside execution units) are usually not modelled.

Characteristics:

* 78% of hidden soft errors are single-bit errors. But not all.
* Bit flip count distribution depends on application.

Takeaway:

Simple error models to not maytch reality of bit
But full hardware simulation 

###

Heirarchical injection.

Gather distribution of error types based on RTL errors.

Care about **instruction level** errors, but realistic ones.

### Monte Carlo

Problem:

* Pre: Pick random instruction.
* Inject hardware error.
* If masked by circuit, stop.
* Post: Play forward.

But, almost all time in pre-post.

Optimization, "nested monte carlo"

* In injection, continue injecting errors **until we find an unmasked error**.

Does this break monte carlo?

No... because we care about errors at instruction level.

### Injection & Detector Types

RB1: flip one bit
RB2: flip 2
RND: random value
RTL: gate-level failure
model+: previous model, plus residue checker
model++: two residue checkers

### Outcome

- Masked: identical
- DUE: Crash
- DUE: detected error (eg. assertion error)
- SDC: silent data corruption

(see paper)

## Conclusion

How to enable fast and high fidelity injection:

* Heirarchical & Nested Monte Carlo

Hamartia:

* http://lph.ece.utexas.edu/users/hamartia/
* https://bitbucket.org/lph_tools/hamartia_suite
