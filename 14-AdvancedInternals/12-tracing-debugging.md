# Tracing and Debugging

This guide covers ftrace, perf, eBPF-assisted tracing, postmortem analysis, and the guide reference appendices.

Linux provides multiple layers of introspection, from classic logs to powerful dynamic tracing frameworks.

## 12.1 Why Multiple Tracing Tools Exist

No single tool answers every question.

Different tools optimize for:

- Stability
- Overhead
- Depth of visibility
- Ease of use
- Production suitability

## 12.2 Tracing Stack Overview

| Tool | Best For |
|---|---|
| `strace` | Syscalls |
| `ltrace` | Library calls |
| `perf` | Sampling and performance profiling |
| `ftrace` | Kernel tracing |
| eBPF tools | Dynamic low-overhead observability |
| SystemTap | Dynamic instrumentation |
| `crash` | Postmortem kernel analysis |
| core dumps + gdb | User-space postmortem debugging |

## 12.3 ftrace

`ftrace` is a built-in kernel tracing framework.

Capabilities:

- Function tracing
- Function graph tracing
- Event tracing
- Scheduler and latency tracing

It is typically exposed via `tracefs`:

```bash
/sys/kernel/tracing
```

## 12.4 Useful ftrace Files

| File | Meaning |
|---|---|
| `current_tracer` | Active tracer |
| `available_tracers` | Supported tracers |
| `set_ftrace_filter` | Function filter |
| `trace` | Trace output |
| `trace_pipe` | Streaming trace output |

## 12.5 Basic ftrace Example

```bash
cd /sys/kernel/tracing
echo function > current_tracer
echo do_sys_openat2 > set_ftrace_filter
cat trace
```

## 12.6 perf

`perf` uses hardware counters, kernel events, tracepoints, and sampling.

Use it for:

- CPU hot spots
- Cache misses
- Scheduler latency
- Call graphs
- Syscall statistics

## 12.7 Common `perf` Commands

```bash
perf stat -d ./app
perf top
perf record -g -- ./app
perf report
perf trace -p 1234
```

## 12.8 Sampling vs Tracing

| Method | Strength | Weakness |
|---|---|---|
| Sampling | Lower overhead, statistical view | Misses exact every-event detail |
| Tracing | Exact event stream | Higher potential overhead |

## 12.9 Flame Graphs

Flame graphs visualize sampled stack aggregates to reveal where CPU time accumulates.

## 12.10 eBPF for Debugging

eBPF can answer questions such as:

- Which syscall is slow?
- Why is run queue latency high?
- Which block device operations are stalling?
- Which process triggers TCP retransmits?

## 12.11 SystemTap

SystemTap predates much of the current eBPF ecosystem and provides powerful scripting for kernel and user-space instrumentation, though it is less favored in many modern environments.

## 12.12 Crash Dumps: `kdump`

`kdump` reserves memory for a capture kernel.

If the main kernel crashes, the capture kernel boots and saves a vmcore for analysis.

## 12.13 `crash` Utility

The `crash` tool analyzes kernel crash dumps and live kernels.

Typical tasks:

- Inspect panicking task
- View backtraces
- Examine slab caches
- Inspect memory zones
- Inspect kernel structures

## 12.14 Core Dumps for User Space

User-space crashes can produce core files containing process memory state at crash time.

Control with:

```bash
ulimit -c unlimited
cat /proc/sys/kernel/core_pattern
```

## 12.15 Debugging a Core Dump with gdb

```bash
gdb /path/to/binary core.1234
bt
info threads
thread apply all bt
```

## 12.16 Kernel Oops vs Panic

| Event | Meaning |
|---|---|
| Oops | Serious kernel fault, may recover partially |
| Panic | Kernel decides system cannot continue safely |

## 12.17 Useful Kernel Debug Sources

- `dmesg`
- `journalctl -k`
- `/sys/kernel/debug` where enabled
- tracefs events
- perf events
- BPF tools

## 12.18 Lockup Detectors

Linux has mechanisms for:

- soft lockup detection
- hard lockup detection
- hung task detection

These help diagnose stalls and deadlocks.

## 12.19 Practical Debug Workflow Examples

### High CPU

1. `top` or `pidstat` to identify process.
2. `perf top` or `perf record` for hot code paths.
3. If kernel-heavy, use `perf` or ftrace.
4. Use BPF if detailed latency or event attribution is needed.

### Process Hung in `D` State

1. Inspect `ps` state.
2. Check `/proc/[pid]/stack`.
3. Inspect storage and NFS health.
4. Use `echo w > /proc/sysrq-trigger` carefully if appropriate.

### Memory Leak Suspicion

1. Observe RSS/PSS growth.
2. Inspect `/proc/[pid]/smaps_rollup`.
3. Use allocator profiling or BPF malloc tracing if available.
4. Check cgroup memory accounting.

## 12.20 Practical Example: Scheduler Latency with BCC

```bash
sudo /usr/share/bcc/tools/runqlat 1 5
```

## 12.21 Practical Example: Block I/O Trace with `perf`

```bash
sudo perf record -e block:block_rq_issue -e block:block_rq_complete sleep 10
sudo perf script | head -50
```

## 12.22 Choosing the Right Tool

| Question | Best Starting Tool |
|---|---|
| What syscalls happen? | `strace`, `perf trace` |
| Where is CPU time spent? | `perf` |
| Which kernel function path is active? | `ftrace`, eBPF |
| Why did kernel crash? | `kdump` + `crash` |
| Why did user process crash? | core dump + `gdb` |

## 12.23 Section Summary

Linux tracing and debugging are a spectrum. Start with the lowest-cost, highest-signal tool for the question at hand, then escalate to deeper instrumentation if needed.

---

## Appendix A: Key Kernel Data Structures

This appendix gives quick-reference descriptions of important structures mentioned across the guide.

## A.1 `task_struct`

Represents a task.

Contains links or fields for:

- Scheduling state
- Identity
- Credentials
- Signal state
- Open files
- Memory mapping
- Namespace and cgroup linkage

## A.2 `mm_struct`

Represents a process address space.

## A.3 `vm_area_struct`

Represents a contiguous virtual memory area with common permissions and backing semantics.

## A.4 `inode`

Represents filesystem object metadata.

## A.5 `dentry`

Represents a cached directory entry.

## A.6 `file`

Represents an open file object.

## A.7 `super_block`

Represents a mounted filesystem instance.

## A.8 `sk_buff`

Represents packet metadata and payload through network stack paths.

## A.9 `bio`

Represents block I/O segments.

## A.10 `page` and `folio`

Represent memory-backed units in kernel MM internals. Newer kernels increasingly use folio abstractions in some paths.

---


## Appendix B: Useful Commands

## B.1 Kernel and Boot

```bash
uname -a
cat /proc/cmdline
dmesg | tail -100
lsmod
```

## B.2 Processes and Scheduling

```bash
ps -eo pid,ppid,stat,ni,pri,cmd --sort=-%cpu | head
cat /proc/1234/sched
pidstat -w 1
taskset -p 1234
```

## B.3 Memory

```bash
free -h
vmstat 1
cat /proc/meminfo
slabtop
numastat
```

## B.4 Filesystems and Storage

```bash
findmnt
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,SCHED
cat /proc/mounts
iostat -xz 1
```

## B.5 Networking

```bash
ss -tulpn
ip addr show
ip route show
ethtool -k eth0
nstat
```

## B.6 Tracing and Profiling

```bash
strace -c ./app
perf stat -d ./app
perf top
sudo bpftool prog show
```

---


## Appendix C: Suggested Reading Path

## C.1 If You Are a Systems Engineer

Read in this order:

1. Kernel Architecture
2. Process Internals
3. Memory Management
4. I/O Subsystem
5. Tracing and Debugging

## C.2 If You Work on Containers

Read in this order:

1. Process Internals
2. Namespaces and Cgroups Deep Dive
3. System Calls
4. File System Internals
5. Network Stack

## C.3 If You Work on Performance

Read in this order:

1. Process Internals
2. Memory Management
3. I/O Subsystem
4. Network Stack
5. eBPF and BPF
6. Tracing and Debugging

---


## Detailed Deep-Dive Notes

The sections above provide structure. The following extended notes expand the material into operationally useful depth. These notes are intentionally long-form so the guide serves as both tutorial and reference.

## D.1 Kernel Architecture Deep-Dive Notes

- Linux keeps a broad set of services in kernel space because crossing protection boundaries is expensive and because direct subsystem cooperation can be highly efficient.
- “Monolithic” does not mean “static.” Loadable modules give Linux runtime extensibility without changing the fundamental protection model.
- Kernel APIs are not stable for out-of-tree modules in the same way user-space ABIs are stable. This is a deliberate design choice that favors in-tree evolution.
- The Linux kernel is preemptive in many common configurations, meaning a running kernel path may be interrupted in favor of another task under allowed conditions.
- Not all kernel code can sleep. Knowing whether your current context can block is a foundational systems skill.
- Driver code often registers callbacks with bus, network, character, block, or filesystem subsystems rather than exposing ad hoc entry points.
- The kernel build system combines feature selection, dependency enforcement, architecture-specific options, and build orchestration at enormous scale.
- System calls are only part of the interaction boundary. Signals, faults, and device interrupts also move control between user and kernel contexts.
- The kernel exposes rich runtime state through pseudo-filesystems and tracing frameworks because operational transparency matters in production.
- A modern Linux system often includes kernel components that were not present in early Unix-like systems, such as cgroups, namespaces, eBPF, and advanced tracing hooks.

## D.2 Process Internals Deep-Dive Notes

- Linux’s task model is flexible enough that “process” and “thread” are policy distinctions based on shared resources, not entirely separate primitive kinds.
- Parent-child relationships matter for signal delivery, exit handling, and resource cleanup, but threads inside a process share many structures.
- A thread group gives user-space the familiar concept of a process while preserving per-thread scheduling and signal details.
- The scheduler chooses tasks from per-CPU runnable structures. That design is critical for scalability on multi-core systems.
- Load balancing tries to reconcile fairness and locality. Migrating tasks improves balance but can hurt cache warmth and NUMA locality.
- Real-time scheduling can starve ordinary work if configured poorly.
- The `D` state is particularly important operationally because it often indicates a task waiting in uninterruptible sleep for I/O or lower-level completion.
- Zombie tasks are usually not dangerous individually, but large accumulations indicate broken process supervision or wait handling.
- `fork()` plus `execve()` is still a core Unix pattern, though higher-level runtimes may use `posix_spawn()` or specialized process launch paths.
- Watching `/proc/[pid]/sched` and context-switch counters often reveals lock contention or CPU overcommit issues quickly.

## D.3 Memory Management Deep-Dive Notes

- Memory performance is often more important than raw CPU instruction throughput in modern systems.
- Virtual memory lets processes believe they own contiguous ranges, but actual physical layout may be fragmented and dynamically managed.
- TLB efficiency is a first-order performance issue for some workloads, especially with large memory footprints and random access patterns.
- Page faults are not necessarily “errors”; they are the mechanism by which lazy allocation and demand paging work.
- Major faults are more expensive because they typically require storage I/O.
- The page cache is a memory consumer and a performance accelerator at the same time.
- Reclaim policy is a balancing act between preserving cache usefulness and satisfying new allocations.
- NUMA effects become more pronounced as systems scale in socket count and memory size.
- THP can improve throughput but may worsen tail latency in some applications due to compaction or fault handling behavior.
- `mmap()` enables elegant zero-copy-like patterns conceptually, but page faults and writeback semantics still matter.

## D.4 Filesystem Internals Deep-Dive Notes

- Path lookup is a major source of filesystem overhead in metadata-heavy workloads.
- The dentry cache exists because names are expensive to resolve repeatedly.
- Inode and file objects are not the same thing. One inode can have many open file objects, and multiple pathnames may link to one inode.
- Journaling improves recoverability but does not eliminate the need for explicit durability calls when application correctness depends on persistence.
- ext4 balances broad compatibility and strong performance for general-purpose workloads.
- XFS often excels in parallel and large-scale environments.
- File deletion semantics in Unix-like systems are often misunderstood; namespace removal and storage reclamation are related but distinct.
- Filesystem performance is tightly coupled to page cache and memory pressure behavior.
- `rename()`-based atomic replacement is fundamental to robust software update and config-write patterns.
- Pseudo-filesystems are not second-class citizens; they are central to Linux observability and control.

## D.5 I/O Subsystem Deep-Dive Notes

- Buffered I/O means applications frequently interact with RAM first and storage later.
- Benchmark methodology must account for cache effects, queue depth, and sync behavior.
- blk-mq reflects the reality that modern storage devices expose much more parallelism than legacy single-queue assumptions allowed.
- I/O schedulers matter differently on HDDs, SATA SSDs, and NVMe devices.
- `io_uring` is not merely “faster async I/O”; it is a flexible interface that reduces submission/completion overhead and broadens async design options.
- Direct I/O can reduce cache pollution but increases application responsibility.
- Dirty-page accumulation and writeback throttling can create surprising latency spikes.
- Device saturation can appear as good throughput with unacceptable p99 latency.
- Readahead helps sequential readers but can hurt or waste memory on random workloads.
- Observability should combine application timings with kernel and device statistics.

## D.6 Network Stack Deep-Dive Notes

- Packet processing cost is shared across hardware offload, driver, softirq, protocol, filtering, and application wakeup paths.
- Network performance work often reduces to careful queue management and CPU affinity tuning.
- NAPI exists because interrupt-per-packet behavior does not scale under load.
- Offloads improve throughput but can obscure packet-level troubleshooting and affect latency characteristics.
- The socket API provides a stable abstraction, but the path from `send()` to wire or wire to `recv()` is highly layered.
- Network namespaces make Linux’s networking stack effectively cloneable per container or sandbox.
- Netfilter hooks enable stateful firewalls and NAT but add processing cost and state complexity.
- XDP is powerful because it moves decision-making earlier in the receive path.
- Conntrack tables are operationally important limits in many Kubernetes and firewall-heavy environments.
- `ss`, `ethtool`, `ip`, `nstat`, and BPF tools together provide a strong practical toolkit.

## D.7 eBPF Deep-Dive Notes

- eBPF is best understood as a constrained in-kernel execution environment plus data-sharing primitives, not just a tracing gimmick.
- The verifier’s restrictions are what make production use feasible.
- High-rate event tracing needs aggregation strategies to stay safe and useful.
- BPF maps enable shared state but can themselves become performance bottlenecks if misused.
- BTF and CO-RE greatly improved portability, moving BPF from expert-only to broadly deployable.
- Tracing, networking, and security are now first-class BPF domains.
- High-level tools like `bpftrace` accelerate exploration, while `libbpf` supports production-grade tooling.
- Stable tracepoints are usually preferable to brittle kprobes when available.
- XDP programs trade generality for very early packet processing efficiency.
- BPF is part of the modern Linux platform engineer’s core toolkit.

## D.8 Namespaces and Cgroups Deep-Dive Notes

- Namespaces answer “what can this process see?”
- Cgroups answer “what can this process use, and how much is it using?”
- Containers are composition, not magic: namespaces, cgroups, mounts, capabilities, seccomp, and often overlay filesystems.
- User namespaces complicate privilege reasoning in a powerful but subtle way.
- cgroup v2 significantly improved coherence of resource control.
- systemd is not separate from cgroups on modern Linux systems; it is often the main orchestrator of them.
- CPU throttling from quotas is a common hidden cause of latency problems in containers.
- Memory pressure inside a cgroup can lead to container-local OOM even when the host still has memory elsewhere.
- PSI is extremely valuable for understanding saturation, not just instantaneous usage.
- cpusets influence both scheduler placement and NUMA memory locality, making them powerful but easy to misuse.

## D.9 System Call Deep-Dive Notes

- Syscalls are not merely “functions implemented by the kernel”; they are ABI-stable boundaries with architectural calling conventions.
- `openat()` style syscalls exist partly to improve race resistance and capability-style directory-relative operations.
- `epoll()` scales by maintaining kernel-side state instead of forcing repeated full scans.
- `ioctl()` exists because dedicated syscalls for every device-specific action would be unwieldy, but it comes with discoverability drawbacks.
- Syscall tracing is a wonderful first step because it shows what a program asks the kernel to do, not what developers think it does.
- vDSO exists because even optimized syscalls are still more expensive than in-process function calls for some fast paths.
- Error codes are a core part of Linux design; robust software treats `EAGAIN`, `EINTR`, and partial results as normal cases, not exceptions.
- One file descriptor integer can hide surprisingly rich kernel state and sharing behavior.
- Descriptor inheritance across fork/exec boundaries is a common source of subtle bugs and leaks.
- Tracing syscalls often reveals inefficient polling, excessive metadata churn, or unnecessary sync patterns.

## D.10 IPC Deep-Dive Notes

- Shared memory is usually the fastest local IPC for bulk data, but it shifts complexity into synchronization and memory ordering.
- Pipes and Unix sockets are easy to reason about and integrate naturally with the fd model.
- Unix domain sockets are significantly more capable than many developers realize because they can pass credentials and descriptors.
- Message queues preserve boundaries, which simplifies some protocols.
- Signals are best for notification, not payload transport.
- `eventfd` and friends integrate events into pollable fd-based loops, reducing special-case designs.
- D-Bus is effectively a structured brokered IPC layer, not just “desktop stuff.”
- Many IPC bugs are really protocol design bugs.
- Blocking semantics matter; deadlocks often arise from full buffers, circular waits, or mismatched framing assumptions.
- Local IPC choices influence security boundaries, observability, and failure handling as much as they influence throughput.

## D.11 Device Driver Deep-Dive Notes

- Drivers sit on subsystem contracts. Very little kernel code is truly standalone.
- sysfs reflects the Linux device model and is often the fastest way to understand how the kernel sees hardware relationships.
- `udev` is a user-space companion that turns kernel events into usable device nodes and policy.
- DMA bugs are among the hardest classes of driver problems because they involve device, CPU, memory, and cache coherency assumptions.
- Power management is no longer optional in many classes of devices.
- Interrupt handling must balance urgency with minimal top-half work.
- Character drivers are conceptually simpler than block or network drivers, which is why they are common teaching examples.
- Driver lifetimes and reference counts are critical; unload paths often expose bugs hidden during steady state.
- The kernel module model makes experimentation possible but does not sandbox mistakes.
- Even for non-driver engineers, understanding how hardware gets surfaced through `/sys` and `/dev` improves troubleshooting dramatically.

## D.12 Tracing and Debugging Deep-Dive Notes

- Start with the narrowest, cheapest tool that can answer the question.
- Logs tell you what components chose to say. Tracing tells you what actually happened.
- Sampling shows where time is spent. Tracing shows when and why events occur.
- `perf` is foundational because it bridges hardware counters, kernel events, and call-graph profiling.
- ftrace is indispensable for kernel path inspection.
- eBPF often provides the sweet spot between specificity and production safety.
- Crash dumps are slow-path gold: when the system is gone, vmcore analysis preserves reality.
- User-space core dumps remain essential despite modern tracing.
- Tool overhead and observer effect are real. Always interpret results with instrumentation cost in mind.
- A disciplined workflow beats tool enthusiasm every time.

---


## Advanced Reference Tables

## R.1 Process State Reference

| Code | Meaning | Typical Cause |
|---|---|---|
| `R` | Runnable/running | On CPU or ready queue |
| `S` | Interruptible sleep | Waiting for event or timeout |
| `D` | Uninterruptible sleep | Waiting on I/O or kernel completion |
| `T` | Stopped | Signal or debugger stop |
| `Z` | Zombie | Exited, not yet reaped |
|
## R.2 Scheduling Policy Reference

| Policy | Area | Notes |
|---|---|---|
| `SCHED_OTHER` | Normal | CFS-based |
| `SCHED_BATCH` | Throughput | Batch-oriented |
| `SCHED_IDLE` | Very low priority | Background work |
| `SCHED_FIFO` | RT | No timeslice among equals |
| `SCHED_RR` | RT | Round-robin among equals |
| `SCHED_DEADLINE` | RT/Deadline | Runtime-deadline-period model |

## R.3 Memory Reference

| Topic | Key File or Tool |
|---|---|
| Overall memory | `/proc/meminfo` |
| Per-process maps | `/proc/[pid]/maps` |
| Detailed map stats | `/proc/[pid]/smaps` |
| NUMA stats | `numastat` |
| Slab usage | `/proc/slabinfo`, `slabtop` |
| Pressure | `/proc/pressure/memory` |

## R.4 Filesystem Reference

| Topic | Key Object |
|---|---|
| Mounted instance | Superblock |
| File metadata identity | Inode |
| Name lookup | Dentry |
| Open state | File |
| Cached contents | Page cache |

## R.5 Networking Reference

| Topic | Tool |
|---|---|
| Sockets | `ss` |
| Routes | `ip route` |
| Links | `ip link` |
| NIC features | `ethtool` |
| Net stats | `nstat` |
| Queueing | `tc` |

## R.6 Tracing Reference

| Need | Tool |
|---|---|
| Syscalls | `strace`, `perf trace` |
| CPU profile | `perf` |
| Kernel functions | `ftrace`, eBPF |
| Block latency | BCC/eBPF, `perf` |
| Crash dump | `crash` |

---


## Practical Walkthroughs

## W.1 Investigating a Slow File Read

1. Verify whether data is page-cache hot.
2. Use `strace -T` to see syscall timing.
3. Use `iostat -xz 1` to inspect device latency.
4. Use `perf` or eBPF to trace block request latency.
5. Check filesystem mount options and readahead behavior.
6. Confirm whether the process is blocked in `D` state or just CPU-bound copying data.

## W.2 Diagnosing Container CPU Throttling

1. Identify service cgroup path.
2. Read `cpu.stat` and `cpu.max`.
3. Compare request rate with throttling counters.
4. Inspect PSI CPU pressure.
5. Reduce quota or increase period/weight according to workload characteristics.
6. Re-measure tail latency, not just average CPU usage.

## W.3 Understanding a Burst of OOM Kills

1. Read kernel logs for OOM messages.
2. Determine whether OOM was global or cgroup-local.
3. Inspect `memory.current`, `memory.max`, and `memory.events`.
4. Check swap availability and reclaim pressure.
5. Evaluate leak, cache growth, or overcommit pattern.
6. Use per-process `oom_score` and memory maps to identify victim rationale.

## W.4 Debugging High Softirq CPU in Networking

1. Check CPU utilization split in `top` or `mpstat`.
2. Inspect NIC interrupts and affinity.
3. Review NAPI, GRO, and offload settings with `ethtool`.
4. Use `perf top` or eBPF on network receive paths.
5. Verify RSS queue distribution.
6. Check firewall, conntrack, and traffic control complexity.

## W.5 Debugging a Process Stuck in `D`

1. Confirm process state with `ps`.
2. Inspect `/proc/[pid]/stack` if permitted.
3. Correlate with storage, NFS, or device logs.
4. Use `echo w > /proc/sysrq-trigger` only with care and proper access.
5. Inspect blocked kernel path using ftrace or eBPF when feasible.
6. Treat persistent `D` state as a kernel or lower-level subsystem symptom, not merely an application bug.

---


## Frequently Asked Advanced Questions

## Q.1 Is Linux really monolithic if it supports modules?

Yes. Modules are dynamically loadable kernel-space code, not user-space servers. The protection boundary remains monolithic.

## Q.2 Is a thread just a lightweight process?

Conceptually yes, but in Linux the deeper truth is that both are tasks with different resource-sharing relationships.

## Q.3 Why does deleting a large file not free space immediately?

Because open file descriptors may still reference the inode after the pathname is removed.

## Q.4 Why does `free` show little free memory on Linux?

Because Linux aggressively uses available RAM for useful caching. `MemAvailable` is usually the more relevant field.

## Q.5 Why can a container OOM when the host still has memory?

Because memory cgroups can enforce local limits independent of total host free memory.

## Q.6 Why is `strace` sometimes misleading for performance?

Because it adds overhead and sees syscalls, not all CPU-intensive user-space work or kernel background activity.

## Q.7 Why do databases often prefer direct I/O?

To avoid double caching and to maintain tighter control over flushing and memory use.

## Q.8 Why is XDP faster than ordinary packet processing?

Because it can act on packets very early, often before skb allocation and higher-level stack processing.

## Q.9 Why are huge pages useful?

They reduce TLB pressure and page-table overhead for suitable workloads.

## Q.10 Is `/proc` a real filesystem?

Yes, but it is virtual and generated by the kernel rather than backed by ordinary disk files.

---


## Key Takeaways by Domain

## Kernel Architecture Takeaways

- Linux is monolithic and modular.
- System calls are controlled entry points.
- Subsystems cooperate inside kernel space for speed.
- Source tree organization mirrors subsystem boundaries.

## Process Takeaways

- `task_struct` is central.
- Threads are tasks with shared resources.
- Scheduling is class-based, not one-size-fits-all.
- `/proc` is your live process microscope.

## Memory Takeaways

- Virtual memory is the core abstraction.
- Page faults and TLB behavior matter in practice.
- NUMA and THP can radically affect performance.
- OOM decisions are policy plus pressure, not random events.

## Filesystem Takeaways

- VFS is the common layer.
- Inodes, dentries, superblocks, and files are distinct concepts.
- Journaling improves consistency, not magical durability.
- Buffered I/O and page cache dominate many workloads.

## I/O Takeaways

- Cache effects can invalidate naive benchmarks.
- blk-mq and NVMe changed storage assumptions.
- `io_uring` is a major modern interface.
- Tail latency matters more than headline throughput in many systems.

## Networking Takeaways

- NAPI, offloads, and queueing are part of the stack story.
- Namespaces make networking per-container.
- Netfilter and conntrack are powerful but stateful and costly.
- XDP and eBPF bring programmability into packet paths.

## eBPF Takeaways

- eBPF is an in-kernel programmable platform.
- Verifier safety is central.
- Use stable hooks when possible.
- Great power requires careful event-rate discipline.

## Namespaces and Cgroups Takeaways

- Namespaces isolate views.
- Cgroups control and account for resources.
- systemd commonly manages cgroups on modern systems.
- PSI is essential for saturation analysis.

## System Call Takeaways

- Syscalls define the user-kernel contract.
- `strace` and friends reveal real behavior quickly.
- ABI details matter for low-level tooling.
- `epoll` and `io_uring` are key scaling primitives.

## IPC Takeaways

- Pick IPC based on workload and semantics.
- Shared memory is fast but synchronization-heavy.
- Unix sockets are far more capable than simple streams.
- Signals are notifications, not data channels.

## Driver Takeaways

- Drivers attach devices to subsystem contracts.
- `/sys` explains device topology.
- `/dev` exposes access handles, not the whole model.
- DMA, interrupts, and power management are recurring themes.

## Tracing Takeaways

- Start simple, escalate carefully.
- `perf` is foundational.
- eBPF is the modern observability powerhouse.
- Crash analysis remains indispensable.

---


## Extended Command Cookbook

## CPU and Scheduling

```bash
ps -Leo pid,tid,psr,ni,pri,stat,comm --sort=psr
chrt -p 1234
perf sched record -- sleep 10
perf sched map
```

## Memory

```bash
grep -E 'Mem|Swap|Dirty|Writeback|Slab' /proc/meminfo
cat /proc/zoneinfo | head -100
cat /proc/buddyinfo
cat /sys/kernel/mm/transparent_hugepage/enabled
```

## Filesystems

```bash
stat -f .
findmnt -no TARGET,SOURCE,FSTYPE,OPTIONS /
cat /proc/sys/vm/dirty_ratio
cat /proc/sys/vm/dirty_background_ratio
```

## Storage I/O

```bash
cat /sys/block/nvme0n1/queue/scheduler
cat /sys/block/nvme0n1/queue/nr_requests
iostat -xz 1
```

## Networking

```bash
cat /proc/softirqs
cat /proc/interrupts | egrep 'eth|ens|eno'
ethtool -S eth0 | head -50
ss -s
```

## eBPF

```bash
sudo bpftool prog show
sudo bpftool map show
sudo bpftrace -l 'tracepoint:syscalls:*'
```

## Cgroups and Pressure

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
systemd-cgtop
```

## Crash and Core Dumps

```bash
ulimit -c unlimited
coredumpctl list
coredumpctl info PID
```

---


## Mini Case Studies

## Case Study 1: The Hidden Cost of Forking a Large Process

A service process with a large address space periodically forks to launch helper commands. Average launch time appears acceptable, but p99 latency spikes occur under memory pressure.

What is happening?

- `fork()` uses COW and does not copy all pages immediately.
- But the kernel must still duplicate page tables and metadata.
- If memory pressure is high, page table allocation or related reclaim can be slow.
- If child or parent quickly writes to many shared pages, COW faults multiply.

Lessons:

- Large-memory forking is not free.
- `posix_spawn()` or worker pools may reduce cost.
- Measure page faults and process launch latency under realistic pressure.

## Case Study 2: “Disk Is Slow” but the Device Is Fine

An application reports slow writes. Device metrics show healthy throughput.

Investigation reveals:

- Application uses buffered I/O.
- Dirty pages accumulate.
- Later `fsync()` calls stall waiting for writeback.
- The problem is perceived at sync boundaries, not every write call.

Lessons:

- Buffered write latency is often deferred.
- Per-call behavior and durability behavior can differ drastically.
- Observe dirty/writeback state and device flush latency.

## Case Study 3: Container Latency Despite Low Host CPU

A containerized API shows high response latency even though host CPU usage is moderate.

Investigation reveals:

- `cpu.max` quota is tight.
- The workload bursts heavily.
- It gets throttled within each quota period.

Lessons:

- Average host CPU is the wrong metric.
- Cgroup throttling can cause severe p99 latency at low average utilization.
- Inspect `cpu.stat`, especially throttling counters.

## Case Study 4: Deleted Log File Still Filling Disk

A log rotation script removes a file, but disk usage stays high.

Investigation reveals:

- The logging process still holds the old file open.
- Namespace entry is gone, but inode remains referenced.

Lessons:

- `unlink()` removes a name, not necessarily the underlying storage immediately.
- Use `lsof +L1` to find open deleted files.
- Reopen logs correctly on rotation.

## Case Study 5: High Packet Loss During Traffic Surge

A service sees drops during burst traffic.

Investigation reveals:

- IRQ affinity is imbalanced.
- RSS queues are not well distributed.
- One CPU handles too many receive queues.

Lessons:

- Networking bottlenecks are often CPU and queue-topology problems, not just bandwidth limits.
- NAPI, RSS, and softirq placement matter.

---


## Advanced Glossary

## ABI

Application Binary Interface. Defines low-level calling conventions and binary compatibility rules.

## CFS

Completely Fair Scheduler. Linux’s primary scheduler for normal tasks.

## COW

Copy-on-write. Optimization where memory is shared until modified.

## Dentry

Directory entry cache object used in path lookup.

## DMA

Direct Memory Access. Hardware data transfer without CPU copying each byte.

## IRQ

Interrupt Request. Hardware or virtual signal indicating an event.

## MMU

Memory Management Unit. Hardware component performing virtual-to-physical translation.

## NAPI

Linux mechanism combining interrupt and polling behavior for efficient network receive processing.

## NUMA

Non-Uniform Memory Access. Memory latency varies by node locality.

## OOM

Out of Memory. Kernel condition where memory cannot be sufficiently reclaimed.

## PSI

Pressure Stall Information. Time-based view of resource stall pressure.

## RCU

Read-Copy-Update. Synchronization primitive optimized for read-heavy access patterns.

## RSS

Receive Side Scaling in networking, and also resident set size in memory discussions. Context matters.

## skb

Socket buffer structure representing a packet in kernel networking.

## THP

Transparent Huge Pages. Automatic huge page promotion mechanism.

## TLB

Translation Lookaside Buffer. Cache of virtual-to-physical translations.

## VFS

Virtual Filesystem Switch. Abstraction layer across filesystem implementations.

## VMA

Virtual Memory Area. Region in a process address space with uniform mapping attributes.

---


## Final Notes

Linux internals reward layered understanding.

If you remember nothing else, remember these principles:

1. **The kernel is a set of cooperating subsystems, not a black box.**
2. **Performance problems are usually boundary problems:** CPU to memory, user to kernel, cache to device, packet to queue, task to scheduler.
3. **Most abstractions leak under pressure.** `/proc`, `/sys`, perf, ftrace, and eBPF help you see the leaks.
4. **Containers do not remove internals; they compose them.**
5. **Correctness and performance both depend on understanding durability, synchronization, and resource isolation semantics.**

Keep this guide nearby when debugging production Linux systems, designing low-level software, or reading kernel source.

---


## Line-Fill Reference Addendum

The following concise bullets intentionally expand topic coverage so this guide can serve as a long-form reference document.

## Addendum 1: Kernel Architecture Bullets

- Kernel boot parameters strongly influence runtime behavior.
- Initramfs bridges early boot and real root filesystem mount.
- Architecture code under `arch/` defines low-level entry, interrupt, and memory setup behavior.
- The scheduler relies on clocks and timers implemented through architecture and timer subsystems.
- Security modules hook many kernel decision points.
- Namespaces, cgroups, and BPF each have hooks spread across multiple core subsystems.
- Some kernel threads exist solely to perform deferred housekeeping work.
- Workqueues allow sleepable deferred processing unlike softirqs.
- RCU callbacks often run asynchronously after grace periods.
- Lock debugging options can change kernel performance characteristics.
- Symbol visibility and export policy matter for modules.
- `/boot/config-*` often preserves the running kernel configuration.
- `modprobe` resolves module dependencies while `insmod` does not.
- Kernel taint flags indicate unsupported or risky runtime conditions.
- `panic_on_oops` and related settings influence failure mode policy.
- `kexec` allows booting another kernel without full firmware reset.
- Live patching exists in some enterprise environments.
- Interrupt affinity affects overall system balance.
- Per-CPU variables avoid global contention on hot paths.
- Lockless algorithms are common where contention would be disastrous.

## Addendum 2: Process Internals Bullets

- PID allocation is namespace-aware.
- Thread IDs differ from process IDs in multi-threaded programs.
- Scheduler statistics can show wait time vs run time.
- Kernel threads often lack user-space mappings.
- `clone3()` adds a more extensible interface in newer kernels.
- CPU affinity changes may interact with cpuset cgroups.
- CFS fairness is weighted, not equal by task count alone.
- Wakeup placement attempts to preserve locality.
- High run queue latency can exist without high total CPU usage.
- PID recycling can confuse naive tooling.
- `ptrace` affects signal and syscall behavior.
- `seccomp` can filter or trap syscalls per task.
- `prctl()` exposes many per-task configuration knobs.
- Session and process group semantics matter for job control.
- TTY ownership influences signal delivery like `SIGINT`.
- `waitid()` offers richer child status reporting than simple `wait()`.
- `RLIMIT_NPROC` and cgroup pids limits both constrain task creation.
- Thread stacks are ordinary mappings with guard pages.
- `pthread_create()` usually ends up in `clone()`-family syscalls.
- `execve()` resets more process state than many developers assume.

## Addendum 3: Memory Bullets

- Anonymous pages and page cache pages compete for reclaim priority.
- Dirty throttling may directly slow writers.
- Swapin readahead can influence anonymous memory performance.
- Page reclaim behavior differs for clean cache vs dirty or unevictable pages.
- Mlocked memory resists reclaim.
- `mlock()` is powerful but dangerous in excess.
- Overcommit heuristics are workload-dependent.
- THP defrag policy can influence latency.
- Page migration helps NUMA balancing and compaction.
- Automatic NUMA balancing can move pages based on access sampling.
- Page table memory itself can be substantial in huge processes.
- KSM trades CPU for RAM savings.
- `MADV_DONTNEED` and `MADV_FREE` change reclaim behavior.
- `mprotect()` changes VMA permissions and may trigger TLB shootdowns.
- TLB shootdowns are cross-CPU coordination events with real cost.
- Forking many threads duplicates page tables for the shared address space just once per process, but task metadata still grows.
- Memory hotplug exists on some platforms.
- `ZONE_MOVABLE` helps with migration and hotplug scenarios.
- `vmstat` counters are cumulative and need interval interpretation.
- Page fault rates must be read in context of workload phase.

## Addendum 4: Filesystem Bullets

- `atime` update policy affects metadata write frequency.
- `relatime` is a common compromise.
- Delayed allocation complicates direct mapping from write call to physical placement.
- Sparse files may have large logical size with little actual allocated storage.
- `fallocate()` can reserve space efficiently.
- Extent fragmentation can still occur under adverse patterns.
- Metadata-intensive workloads may be CPU-bound more than disk-bound.
- Directory indexing strategies matter for large maildir-style trees.
- Case sensitivity is usually preserved in Linux filesystems.
- `fsnotify` underpins inotify and fanotify mechanisms.
- Overlay filesystems add additional lookup and copy-up semantics.
- `tmpfs` is memory-backed and swap-backed rather than disk-backed.
- `procfs` entries often compute results on demand.
- `debugfs` is powerful but not designed as a stable ABI.
- Filesystem freeze mechanisms exist for snapshots and backup coordination.
- Quotas add accounting and enforcement overhead.
- Extended attributes support ACLs, SELinux labels, and application metadata.
- Inode exhaustion can occur before block exhaustion on some layouts.
- Mount propagation controls matter heavily in containers.
- Bind mounts are fundamental namespace tools.

## Addendum 5: I/O Bullets

- HDDs suffer dramatically from random seek workloads.
- SSDs remove seek cost but still have internal erase/program behavior.
- NVMe emphasizes queues and parallel submission/completion.
- IO scheduler selection should match workload and hardware.
- Merging adjacent requests matters more on rotational media.
- Latency percentiles matter more than averages for user experience.
- Write amplification exists below the filesystem in flash translation layers.
- Flush and barrier semantics are critical for durability-sensitive apps.
- `fio` is the standard workload generator for storage benchmarking.
- `direct=1` in `fio` changes cache interaction drastically.
- Submission queue polling can reduce latency in tuned environments.
- Read-after-write timing can be misleading with page cache.
- Background discard or trim may affect SSD behavior.
- Queue depth tuning is workload-specific.
- Single-thread benchmarks may underutilize modern storage.
- Readahead windows can be tuned per block device.
- Per-process I/O priority exists via `ionice` in some contexts.
- `io.cost` and related cgroup controls may appear in tuned environments.
- Filesystem journaling interacts with device flush behavior.
- Metadata sync patterns can dominate small-write workloads.

## Addendum 6: Networking Bullets

- Loopback traffic still traverses much of the stack.
- TCP small queue mechanisms help control bufferbloat in local stack behavior.
- Nagle’s algorithm and delayed ACKs can interact badly for latency-sensitive tiny writes.
- `SO_REUSEPORT` enables scalable listener sharding.
- Accept queue depth matters for bursty traffic.
- SYN cookies protect under backlog pressure with tradeoffs.
- Reverse path filtering affects multihomed routing behavior.
- Policy routing enables source-based or mark-based decisions.
- VRFs provide routing-domain separation.
- `tc` qdiscs shape egress and some ingress-adjacent paths.
- BQL helps with NIC driver queue sizing.
- GRO can reduce per-packet overhead at the cost of some immediacy.
- Busy-polling can reduce latency but consumes CPU.
- Netlink is the canonical interface for modern network configuration.
- Bridge, veth, and overlay devices build container networking topologies.
- Conntrack timeouts influence state table pressure.
- UDP receive buffer overruns can silently drop telemetry.
- TCP autotuning grows buffers with workload demand.
- IRQ and RPS/XPS tuning influence multi-core packet distribution.
- Packet capture tools may observe pre- or post-offload views differently.

## Addendum 7: eBPF Bullets

- Always verify kernel support before assuming a BPF feature exists.
- `bpftool feature` is invaluable on heterogeneous fleets.
- BPF ring buffers simplify efficient event delivery.
- Per-CPU maps reduce contention for hot counters.
- Stack trace capture is powerful but expensive.
- Uprobes bridge user-space visibility gaps.
- LSM hooks enable policy and audit extensions.
- Tail calls let BPF programs compose larger logic.
- Verifier log output is often essential for development.
- Object pinning allows persistent BPF objects in bpffs.
- bpffs is commonly mounted at `/sys/fs/bpf`.
- Cgroup hooks allow policy by workload boundary.
- Tracepoint arguments are more stable than internal struct layouts.
- CO-RE depends on BTF availability and correct relocation use.
- kretprobes can capture return values and latency.
- Histogram aggregation is often more useful than raw event logging.
- Sampling can be more production-friendly than exhaustive tracing.
- Use per-event stack capture sparingly.
- BPF observability must be designed, not just attached.
- The best BPF script is often the one that aggregates near the source and exports little.

## Addendum 8: Namespaces and Cgroups Bullets

- Mount propagation bugs are common in nested container setups.
- PID namespace init process has special semantics.
- User namespace mappings are written via uid_map and gid_map interfaces.
- Network namespace cleanup depends on reference lifetimes of interfaces and sockets.
- Cgroup delegation must preserve safe subtree control rules.
- `memory.high` throttles before `memory.max` kills.
- `memory.events` is a goldmine for pressure diagnosis.
- cgroup I/O controls may behave differently across device types and kernels.
- systemd slices structure large service trees cleanly.
- Transient units create transient cgroups automatically.
- `systemd-run --scope` is useful for experimentation.
- cpuset constraints influence scheduler load balancing.
- `pids.max` protects against fork bombs.
- Cgroup namespaces affect what paths a process sees for itself.
- PSI can show saturation even when raw utilization appears modest.
- Container runtimes often create layered cgroup subtrees.
- Unified hierarchy semantics differ meaningfully from v1 mental models.
- Resource protection can be as important as resource limits.
- `oom.group` influences cgroup kill behavior in some setups.
- Delegation mistakes can create confusing accounting gaps.

## Addendum 9: System Calls Bullets

- Syscall tables are architecture-specific.
- Compat syscalls handle 32-bit user space on 64-bit kernels.
- `copy_from_user()` and `copy_to_user()` embody the need for explicit safe boundary copying.
- Invalid user pointers must be detected safely by the kernel.
- Many syscalls can sleep, and that influences lock rules inside implementations.
- `restart_syscall` machinery exists for interrupted blocking operations.
- `openat2()` adds richer path resolution controls.
- `pidfd_*` syscalls modernize process handle semantics.
- `memfd_create()` enables anonymous in-memory file semantics.
- `splice()`, `vmsplice()`, and `sendfile()` support zero-copy-like data movement patterns.
- `ioctl()` command encoding often includes direction and size metadata.
- `seccomp` operates at the syscall boundary.
- Audit and tracing frameworks also hook syscall activity.
- Some library calls never invoke syscalls if data is already buffered in user space.
- Repeated small syscalls can dominate workload overhead.
- Batched interfaces often exist to reduce syscall count.
- `epoll` edge-triggered mode requires disciplined drain loops.
- `poll()` and `select()` readiness are not guarantees that large operations will complete fully.
- Syscall timing outliers often point to deeper subsystem behavior, not the syscall mechanism itself.
- Understanding return codes is essential to writing robust low-level software.

## Addendum 10: IPC Bullets

- Pipe buffer sizes can matter for throughput and deadlock behavior.
- `socketpair()` is a handy full-duplex local IPC primitive.
- POSIX shared memory objects live in a filesystem-like namespace.
- `memfd_create()` avoids persistent filesystem paths for shared-memory-like designs.
- Semaphore misuse often causes liveness bugs rather than crashes.
- Realtime signals can queue, unlike many traditional signals.
- Signal disposition is inherited across fork and partly reset on exec.
- `signalfd` can simplify event-loop integration.
- `timerfd` similarly avoids signal-based timers in many designs.
- D-Bus traffic can be inspected with dedicated tooling for debugging service interactions.
- Netlink message formats can be nested and family-specific.
- Ancillary data in Unix sockets supports much more than plain payload bytes.
- Message framing must be explicit on byte-stream channels.
- Shared memory requires cache-coherent synchronization design, not just locking.
- Ring buffers in shared memory are common high-throughput IPC structures.
- Security labeling and credentials may affect IPC permission semantics.
- Namespaces change visibility of some IPC objects.
- Resource limits apply to queues, shm segments, and open fds.
- IPC debugging often benefits from both syscall tracing and application logs.
- Many “networking” control operations on Linux are actually local netlink IPC.

## Addendum 11: Device Driver Bullets

- Device major/minor numbers are part of the user-kernel interface.
- `mknod` can create device nodes manually, though `udev` usually handles it.
- Platform devices are common on SoCs without discoverable buses.
- Device tree `compatible` strings drive binding on many embedded systems.
- ACPI plays a similar descriptive role in many PC/server contexts.
- Probe and remove callbacks define driver lifecycle around binding.
- Deferred probing resolves dependency ordering problems.
- Interrupt storms can cripple a system rapidly.
- MSI/MSI-X changed interrupt handling characteristics on modern PCIe devices.
- Runtime PM lets drivers suspend idle hardware dynamically.
- Sysfs attributes often provide useful knobs and state exposure.
- Debugfs may expose additional driver internals when enabled.
- Error handling paths are as important as happy-path probe logic.
- Hotplug safety requires robust reference counting.
- DMA mask configuration matters for addressing constraints.
- IOMMU presence changes DMA and isolation behavior.
- Character driver `read()` and `write()` callbacks are only the surface of a broader lifetime model.
- Block drivers integrate deeply with blk-mq.
- Network drivers integrate deeply with NAPI, queues, and offloads.
- Driver bugs can manifest as filesystem, memory, or network anomalies far above the hardware layer.

## Addendum 12: Tracing and Debugging Bullets

- `perf stat` is an excellent first stop for hardware counter perspective.
- `perf record -g` plus call graphs often reveals real bottlenecks quickly.
- Frame pointers improve stack quality in some environments.
- DWARF unwinding can add overhead but improve stack fidelity.
- ftrace function graph tracer shows call nesting and duration patterns.
- Tracepoints are stable and structured.
- `trace_pipe` provides live consumption of ftrace output.
- Dynamic debug can selectively enable kernel log statements.
- `sysrq` keys provide emergency debugging hooks.
- Hung task detector points toward blocked kernel paths.
- `lockdep` can detect locking issues in debug kernels.
- `kmemleak` helps with kernel memory leak debugging in appropriate kernels.
- `drgn` is another modern introspection option in some environments.
- `coredumpctl` integrates user-space core dump management with systemd.
- Reproducing issues under controlled load is often the hardest part of debugging.
- Always correlate traces with workload phase and environment conditions.
- Measuring both rate and latency distributions yields better insight than a single average.
- Observer effect can invalidate tight latency investigations if ignored.
- Build IDs and symbols matter for trustworthy postmortem analysis.
- A saved trace without timestamps, environment, and reproduction details is often far less valuable later.

---


## Ultra-Concise Recap Lines

- Linux is monolithic but modular.
- Tasks, not magical “process objects,” are the core execution abstraction.
- Virtual memory is the illusion; page tables and faults are the machinery.
- VFS hides filesystem diversity behind common objects.
- The page cache is everywhere.
- Block I/O is layered and queue-driven.
- Networking is queueing and policy as much as protocol.
- eBPF is programmable kernel observability and control.
- Namespaces isolate views; cgroups isolate resources.
- Syscalls are the narrow waist of Linux.
- IPC choices shape system behavior.
- Drivers map hardware into kernel abstractions.
- Tracing tools turn mystery into mechanics.

---


## Supplemental Command Index

1. `uname -a`
2. `lsmod`
3. `modinfo`
4. `cat /proc/cmdline`
5. `ps -ef`
6. `pidstat -w 1`
7. `taskset -p`
8. `chrt -p`
9. `cat /proc/[pid]/sched`
10. `cat /proc/[pid]/maps`
11. `cat /proc/meminfo`
12. `vmstat 1`
13. `slabtop`
14. `numastat`
15. `findmnt`
16. `stat`
17. `iostat -xz 1`
18. `lsblk`
19. `ss -tulpn`
20. `ip addr`
21. `ip route`
22. `ethtool`
23. `nstat`
24. `bpftool prog show`
25. `bpftrace -l`
26. `systemd-cgls`
27. `systemd-cgtop`
28. `strace -f -tt -T`
29. `perf stat`
30. `perf record -g`
31. `perf report`
32. `cat /proc/pressure/cpu`
33. `cat /proc/pressure/memory`
34. `cat /proc/pressure/io`
35. `coredumpctl list`
36. `journalctl -k`
37. `dmesg`
38. `lspci -nnk`
39. `udevadm info`
40. `readlink /proc/self/ns/pid`

---


## Closing Summary

Advanced Linux internals are best learned as a connected system:

- Process behavior depends on scheduler and memory state.
- Filesystem behavior depends on page cache and writeback.
- Networking behavior depends on queues, offloads, and CPU topology.
- Containers depend on namespaces, cgroups, mounts, capabilities, and syscalls.
- Modern observability depends on perf, tracepoints, and eBPF.

Master the relationships, not just the terms.

End of guide.
