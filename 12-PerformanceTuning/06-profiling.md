# System Profiling

[Back to guide index](README.md)

Profiling shows where time and waits actually go.

This section covers:

- perf
- flame graphs
- strace
- ltrace
- bpftrace
- SystemTap

## 6.1 Monitoring vs profiling

Monitoring tells you:

- what happened
- when it started
- how bad it is

Profiling tells you:

- where time is spent
- which path is hot
- which syscall dominates
- which wait blocks progress

## 6.2 `perf`

`perf` supports:

- event counting
- sampling
- call graphs
- scheduler analysis
- hardware counters
- tracepoints

## 6.3 `perf stat`

```bash
perf stat -d -d -d ./app
```

Useful counters:

- cycles
- instructions
- cache misses
- branch misses
- task-clock
- context switches
- page faults

## 6.4 `perf top`

```bash
perf top
```

Use for live hotspots.

## 6.5 `perf record`

```bash
perf record -F 99 -g -p <pid> -- sleep 30
```

## 6.6 `perf report`

```bash
perf report
```

Inspect:

- inclusive cost
- exclusive cost
- user/kernel split
- call chains

## 6.7 `perf sched`

```bash
perf sched record sleep 10
perf sched timehist
```

Use for run queue and wakeup analysis.

## 6.8 Flame graphs

Flame graphs show aggregated stacks.

Interpretation:

- width = total samples
- height = stack depth

Example workflow:

```bash
perf record -F 99 -g -p <pid> -- sleep 30
perf script > out.perf
stackcollapse-perf.pl out.perf > out.folded
flamegraph.pl out.folded > flame.svg
```

## 6.9 `strace`

Use to trace syscalls.

### 6.9.1 Summary mode

```bash
strace -c -p <pid>
```

### 6.9.2 Follow forks

```bash
strace -ff -o trace.out ./app
```

### 6.9.3 Filter examples

```bash
strace -e trace=network,read,write -p <pid>
```

## 6.10 `ltrace`

Use to trace user-space library calls.

```bash
ltrace -c -p <pid>
```

## 6.11 `bpftrace`

`bpftrace` is powerful and flexible.

Example:

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

Example histogram:

```bash
bpftrace -e 'kprobe:do_sys_open { @start[tid] = nsecs; } kretprobe:do_sys_open /@start[tid]/ { @us = hist((nsecs-@start[tid])/1000); delete(@start[tid]); }'
```

## 6.12 SystemTap

SystemTap provides powerful tracing.

It is older than eBPF-based tooling.

Use it where your environment supports it and the operational cost is acceptable.

## 6.13 Off-CPU analysis

Slow applications are often waiting, not computing.

Common waits:

- disk I/O
- network I/O
- locks
- scheduler delay
- futex waits

## 6.14 Lock analysis

Useful tools:

- `perf lock`
- `perf sched`
- application profiler
- `bpftrace`

## 6.15 Profiling safety

- prefer sampling first
- keep captures short
- filter by PID or cgroup
- test profiler overhead in staging
- avoid broad tracing during severe incidents unless needed

## 6.16 Profiling workflow

1. scope the target
2. choose low-overhead method
3. capture representative interval
4. inspect hottest stacks or slowest waits
5. correlate with service metrics
6. test the fix
7. re-profile

## 6.17 Profiling quick commands

```bash
perf stat -d ./app
perf top
perf record -F 99 -g -p <pid> -- sleep 30
perf report
perf sched timehist
strace -c -p <pid>
ltrace -c -p <pid>
bpftrace -e 'profile:hz:99 { @[ustack] = count(); }'
```

---
