

Testing of job

### RM-Replay

Use container to isolate resource-manager stack.

Recreate set of users.

Create adjustable clock to replace system clock.

### Slurm Replay

Widely used.

Complex software. There is a simulator, but it modifes the codebase and it's
very complicated.

Objective is to replay slurm as a black box.

### How does it work?

Containerized system.

Extract trace of workload execution from the slurm database. Replay it later.

Fake system clock.

The submitter replays the job submissions in the trace.

Clock doesn't move until you start the ticker.

### Problems

* No root access inside container.
* Wrap `setuid`, `setgid`, `chown` inside container.
* Wrap `sleep`, `gettimeofday` to use a mock clock. Counter in `/dev/shm`.
* Extract `/etc/passwd`, `/etc/group` from users in workload trace.
* Data missing from slurm DB: dependencies, node topology, ...

### Accuracy

Can run up to 16x.

### Questions

Use case. "Will my job be scheduled earlier if I provide a better estimate of
runtime?"

### Conclusion

Useful for testing what-if scenarios: parameter, verson, feature, policy

Conigured with the same rules as the real system. It is a slurm system.


## Evaluation of an Interference-Free Node Allocation Policy on Fat-Tree Clusters

Understanding performance variability.

Fat tree clusters.

Most resource managers do not consider network topology.

Can we improve performance if we use isolated network partitions for jobs.

Collect OTF2 traces for comms proxies.

Model a 3-level 10k node fat tree with 44 ports.

Simulate randomly generated multi-jonb workloads.

### Same results in practice?

Simulated 1.2 - 1.7x

Speedups 1.26-1.55x in real life

Not the same, but still clear speedups.

### Isolation

Usually not a good idea. Fragmentation causes utilization issues.

But these "rules of thumb" don't account for improved performance of
isolation.

### Fat-tree networks

https://en.wikipedia.org/wiki/Fat_tree

L1 / 2 / 3 level switches

* Nodes share a L1 switch.
* Pods share a L2 switch.
* Pods connected by L3 switches.

Independent pods connected at the L3 level.

Trying to avoid L3 congestion that you get when jobs cross pods.

### Objective

If a job is small enough to need only switch (in a pod) then only use one pod.

### Method

Classificaiton:

* T1: single switch
* T2: pod
* T3: multi-pod

### Implemented in Flux

https://github.com/flux-framework/flux-core

- node allocation plugin
- hardware topology file

Heuristics:

- T1 jobs place in pod/switch with the fewest available (that's still enough)
- T2 jobs: place in pod with fewest nodes available that's still enough
- T3 jobs: podxs/switches with largest number of nodes available

### Simulator

Offline simulator.

Simulate multiple system sizes.

Used hypothetical machines too.

### Conclusion

Observing poor performance due to scheduling that doesn't account for network
partitions.

Gains with topology aware scheduling.

Not correct that other quality of service metrics will degrade significantly.

Increased performance makes up for degraded fragmentation. Overall, job
submission time to job completion is improved.


## Mitigating Inter-Job Interference Using Adaptive Flow-Aware Routing

Dedicated nodes. But sharing compute network.

Hence potential inter-job network interference.

Most systems now use following topologies (rather than Torus)

* Dragonfly
* Fat-tree (previous talk)


### Contributions

1. Measured inter-job interference on Dragonfly and Fat-tree. Can be **up to
   2x worse**

2. Performed analysis of the contention. Usually a few **hotspots**.

3. Developed a routing based solution. **Adaptive Flow-Aware Routing**

### Types of Routing

* Dragonfly: adaptive. Multiple paths. Routers decide at runtime which one to
  use. Can account for runtime hotspots.
* Fat-tree: usually static. Only one path, deterministically in advance.

### Interference Experiment

Benchmarks:

* Bisection bandwidth
* Nearest Neighbors
* Random Pairs
* FFT Proxy

Methodology:

* 70-75% of time on computation
* In isolation and competition

### Results

Bisection and NN types were more susceptible. Median slowdown 30-50%

High variance.

### Dig In

Look at link bandwidth in low-slowdown and high-slowdown cases.

Clear relationship between runtime and max (bottleneck) link load.

Clearly a step change in the graph: starting at 60GB traffic. That's 80% of
advertised maximum.

Only a few links cross this threshold.

If we could **reduce load on a few links**, it would make a massive difference.

### AFAR: Adaptive Flow-Aware Routing

Idea: periodically reroute to alleveiate hotspots.

Param: desired max link load.

1. Calculate current link-load
2. Look at max load. If they are all below threshold, stop.
3. Reroute one flow crossing link to a less utilized link.
4. Repeat.

(Timeouts if it cannot be satisfied after some time.)

### Prototype

In OpenSM, an InfiniBand subnet manager.

* It's responsible for computing routing tables and distributing them.
* Open-source.

Had AFAR algorithm running in a separate process.

Current work on integrating with OpenSM.

### Results

Massive improvements on bisecton and NN benchmarks.

### Conclusion

Network interference can cause up to 2x slowdowns

AFAR targets network hotspots at runtime

Performance improvements in the 30-40% range



