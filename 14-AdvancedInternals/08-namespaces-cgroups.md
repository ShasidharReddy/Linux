# Namespaces and Cgroups

This guide covers Linux isolation primitives, cgroup resource controls, and container foundations.

Namespaces and cgroups are core building blocks of Linux containers. Namespaces provide isolation. Cgroups provide accounting, control, and resource policy.

## 8.1 Namespaces Overview

Namespaces isolate views of specific global resources.

The canonical Linux namespace types are:

1. Mount namespace
2. PID namespace
3. Network namespace
4. UTS namespace
5. IPC namespace
6. User namespace
7. Cgroup namespace
8. Time namespace

## 8.2 Mount Namespace

Isolates mount points and filesystem view.

Useful for containers so each sees its own root filesystem and mount topology.

## 8.3 PID Namespace

Isolates process ID numbering.

A process may have different PIDs in different namespaces.

## 8.4 Network Namespace

Provides independent network stack instances:

- Interfaces
- Routing tables
- Firewall rules
- Socket spaces
- `/proc/net`

## 8.5 UTS Namespace

Isolates hostname and domain name.

## 8.6 IPC Namespace

Isolates System V IPC and POSIX message queues.

## 8.7 User Namespace

Maps user and group IDs inside the namespace to different IDs outside.

This enables unprivileged container mechanisms and refined privilege models.

## 8.8 Cgroup Namespace

Virtualizes cgroup path visibility.

## 8.9 Time Namespace

Allows isolation of some time-related offsets.

## 8.10 Inspect Namespace Links

```bash
ls -l /proc/self/ns
readlink /proc/self/ns/pid
```

## 8.11 Creating Namespaces

Tools:

```bash
unshare --mount --uts --ipc --net --pid --fork bash
nsenter --target 1234 --mount --uts --ipc --net --pid
```

## 8.12 Namespaces in the Kernel

A task’s namespace membership is represented by namespace-specific structures referenced from its task state.

## 8.13 User Namespace Nuance

User namespaces are powerful and subtle.

Inside a user namespace, a process may appear as UID 0 while being unprivileged on the host.

## 8.14 Cgroups Overview

Control groups organize processes into hierarchical resource management domains.

Cgroups provide:

- Accounting
- Limits
- Weights
- Protection thresholds
- Event reporting
- OOM scoping in modern environments

## 8.15 cgroup v1 vs cgroup v2

| Feature | v1 | v2 |
|---|---|---|
| Hierarchy | Multiple disjoint hierarchies | Unified hierarchy |
| Controller model | Separate mounting often used | Unified and cleaner |
| Modern systemd support | Partial and legacy | Preferred |
| Delegation | More complex | Better defined |

## 8.16 cgroup v2 Design

The unified hierarchy simplifies resource control and accounting.

Mounted typically at:

```bash
/sys/fs/cgroup
```

## 8.17 Important cgroup v2 Controllers

| Controller | Purpose |
|---|---|
| `cpu` | CPU bandwidth and weights |
| `cpuset` | CPU and memory node placement |
| `memory` | Memory accounting and control |
| `io` | I/O control |
| `pids` | Process count limits |
| `rdma` | RDMA resources |
| `hugetlb` | Huge page accounting |

## 8.18 CPU Controller

Examples:

- `cpu.max`
- `cpu.weight`
- `cpu.stat`

## 8.19 Memory Controller

Important files:

- `memory.current`
- `memory.max`
- `memory.high`
- `memory.low`
- `memory.min`
- `memory.stat`
- `memory.events`

## 8.20 I/O Controller

Controls or weights device I/O using cgroup-v2 interfaces.

## 8.21 PID Controller

`pids.max` limits the number of tasks in the cgroup.

## 8.22 Resource Accounting

Cgroups let you answer questions like:

- Which container is consuming memory?
- Which service is causing I/O pressure?
- Which workload is CPU throttled?

## 8.23 systemd Integration

On many modern systems, systemd is the cgroup manager.

Units map into cgroups such as:

- `system.slice`
- `user.slice`
- `machine.slice`

Inspect:

```bash
systemd-cgls
systemd-cgtop
```

## 8.24 Example cgroup Paths

```text
/sys/fs/cgroup/system.slice/sshd.service
/sys/fs/cgroup/user.slice/user-1000.slice
```

## 8.25 CPU Throttling

Quota-based CPU control can throttle tasks when budget is exhausted within a period.

This is a frequent container performance issue.

## 8.26 Memory Protection and Pressure

In cgroup v2, `memory.low` and `memory.min` can protect workloads from reclaim pressure relative to others.

## 8.27 PSI

**Pressure Stall Information** reports time stalled on CPU, memory, and I/O pressure.

Files:

```bash
/proc/pressure/cpu
/proc/pressure/memory
/proc/pressure/io
```

## 8.28 Practical Example: Inspect cgroup Membership

```bash
cat /proc/self/cgroup
systemd-cgls --no-pager | head -100
```

## 8.29 Container Runtime Interactions

Container runtimes combine:

- Namespaces for isolation
- Cgroups for resource policy
- Bind mounts and overlay filesystems for rootfs
- Capabilities and seccomp for security controls

## 8.30 Common Namespace and Cgroup Pitfalls

| Symptom | Cause |
|---|---|
| Container sees PID 1 internally but not on host | PID namespace translation |
| CPU limited despite low host usage | cgroup quota throttling |
| Memory OOM inside container only | cgroup-local memory limit |
| Missing devices or mounts | Namespace or mount propagation issue |

## 8.31 Section Summary

Namespaces and cgroups together provide the isolation-and-control model that makes containers possible. Understanding them deeply is essential for debugging orchestration platforms, systemd services, and multi-tenant Linux environments.

---
