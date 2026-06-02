# Application Performance

[Back to guide index](README.md)

Application behavior often dominates system behavior.

This section covers:

- ulimits
- cgroups v1 vs v2
- nice and ionice
- OOM controls
- queueing and thread pools

## 8.1 `ulimit`

Check:

```bash
ulimit -a
```

Important limits:

- open files
- max user processes
- stack size
- locked memory
- core file size

### 8.1.1 Open files

Check:

```bash
ulimit -n
cat /proc/<pid>/limits
```

Low FD limits can cap scale.

## 8.2 `nice`

Adjust CPU priority for normal tasks.

Examples:

```bash
nice -n 10 batch_job
renice -n -5 -p <pid>
```

## 8.3 `ionice`

Adjust I/O priority.

Examples:

```bash
ionice -c2 -n0 -p <pid>
ionice -c3 -p <pid>
```

Useful for background jobs.

## 8.4 cgroups v1 vs v2

### 8.4.1 cgroups v1

Characteristics:

- multiple hierarchies
- inconsistent semantics
- older tooling model

### 8.4.2 cgroups v2

Characteristics:

- unified hierarchy
- cleaner semantics
- better protections and pressure reporting

## 8.5 CPU cgroup controls

Important files in v2:

- `cpu.max`
- `cpu.weight`
- `cpu.stat`
- `cpu.pressure`

### 8.5.1 `cpu.max`

Example:

```bash
echo "200000 100000" > cpu.max
```

## 8.6 Memory cgroup controls

Important files:

- `memory.current`
- `memory.max`
- `memory.high`
- `memory.low`
- `memory.events`

## 8.7 I/O cgroup controls

Important files:

- `io.max`
- `io.weight`
- `io.stat`
- `io.pressure`

## 8.8 pids controller

Important files:

- `pids.max`
- `pids.current`

Useful to prevent runaway forks.

## 8.9 OOM score adjustment

Check:

```bash
cat /proc/<pid>/oom_score
cat /proc/<pid>/oom_score_adj
```

Set example:

```bash
echo -500 > /proc/<pid>/oom_score_adj
```

Use sparingly.

## 8.10 Thread pools

Common mistakes:

- too many threads
- too few threads
- blocking work in CPU pools
- unbounded queues

Practical guidance:

- separate CPU-bound and I/O-bound pools
- measure queue wait time
- set bounds
- monitor task rejection and timeout rates

## 8.11 Connection pools

Connection pools should be:

- bounded
- instrumented
- sized to backend capacity

## 8.12 Allocators

Allocator choice can affect:

- RSS
- fragmentation
- latency
- CPU overhead

Examples:

- glibc malloc
- jemalloc
- tcmalloc

## 8.13 Startup and warmup

Performance may differ between:

- cold start
- warm cache
- post-failover
- autoscaled new instance

Benchmark all important phases.

## 8.14 Containers and orchestration

Consider:

- requests vs limits
- CPU throttling
- memory limits
- storage class latency
- sidecar overhead
- service mesh cost

## 8.15 Application tuning practices

- tune app and OS together
- track queue time
- keep limits version-controlled
- validate cgroup behavior under load
- protect critical services thoughtfully

## 8.16 Application quick commands

```bash
ulimit -a
cat /proc/<pid>/limits
nice -n 10 ./batch_job
renice -n -5 -p <pid>
ionice -c2 -n7 -p <pid>
cat /proc/<pid>/oom_score
cat /proc/<pid>/oom_score_adj
cat /sys/fs/cgroup/cpu.max
cat /sys/fs/cgroup/memory.current
cat /sys/fs/cgroup/io.stat
```

---
