# Kernel Tuning

[Back to guide index](README.md)

Kernel tuning should be:

- measured
- minimal
- documented
- reversible

## 7.1 `sysctl` basics

View:

```bash
sysctl -a
```

Read one value:

```bash
sysctl vm.swappiness
```

Set temporarily:

```bash
sysctl -w vm.swappiness=10
```

Apply persistent settings:

```bash
sysctl --system
```

## 7.2 `/proc/sys`

The same tunables are exposed in `/proc/sys`.

Example:

```bash
cat /proc/sys/vm/swappiness
echo 10 | sudo tee /proc/sys/vm/swappiness
```

## 7.3 Tuning categories

- virtual memory
- networking
- filesystem
- scheduler
- process limits
- NUMA

## 7.4 Essential sysctl table

| Parameter | Purpose | Common use |
|---|---|---|
| `vm.swappiness` | swap tendency | latency-sensitive hosts |
| `vm.dirty_ratio` | dirty page threshold | write-heavy systems |
| `vm.dirty_background_ratio` | flusher threshold | smooth writeback |
| `vm.max_map_count` | mmap area ceiling | mmap-heavy apps |
| `vm.overcommit_memory` | allocation policy | DBs and alloc-heavy apps |
| `vm.min_free_kbytes` | reserve memory | page allocation stability |
| `fs.file-max` | global FD cap | high-connection servers |
| `fs.nr_open` | per-process FD ceiling | very large FD counts |
| `net.core.somaxconn` | listen backlog cap | busy servers |
| `net.core.netdev_max_backlog` | ingress backlog | packet bursts |
| `net.ipv4.tcp_max_syn_backlog` | SYN queue | connection bursts |
| `net.ipv4.ip_local_port_range` | ephemeral ports | many outbound connections |
| `net.core.rmem_max` | max recv buffer | high-BDP paths |
| `net.core.wmem_max` | max send buffer | high-BDP paths |
| `kernel.pid_max` | max PID | high process churn |
| `kernel.threads-max` | max threads | thread-heavy apps |
| `kernel.numa_balancing` | auto NUMA balancing | NUMA-aware tuning |

## 7.5 Virtual memory tunables

### 7.5.1 `vm.swappiness`

Lower values usually reduce swap preference.

Test under pressure.

### 7.5.2 Dirty page settings

Relevant:

- `vm.dirty_ratio`
- `vm.dirty_background_ratio`
- `vm.dirty_bytes`
- `vm.dirty_background_bytes`

### 7.5.3 `vm.overcommit_memory`

Values:

- `0`
- `1`
- `2`

### 7.5.4 `vm.max_map_count`

Important for some mmap-heavy services.

## 7.6 Filesystem and FD tunables

### 7.6.1 `fs.file-max`

Global file descriptor limit.

Check:

```bash
sysctl fs.file-max
```

### 7.6.2 `fs.nr_open`

Upper bound for per-process open files.

## 7.7 Network tunables

### 7.7.1 `net.core.somaxconn`

Maximum listen backlog.

### 7.7.2 `net.core.netdev_max_backlog`

Ingress queue backlog.

### 7.7.3 `net.ipv4.tcp_max_syn_backlog`

Queue for pending SYNs.

### 7.7.4 `net.ipv4.ip_local_port_range`

Ephemeral source port range.

### 7.7.5 `net.ipv4.tcp_rmem`

TCP receive buffer autotuning range.

### 7.7.6 `net.ipv4.tcp_wmem`

TCP send buffer autotuning range.

## 7.8 NUMA tuning

`kernel.numa_balancing` can help or hurt depending on workload.

Benchmark before changing.

## 7.9 Safe rollout pattern

1. document current value
2. define success metric
3. change one parameter
4. test in staging
5. canary in production
6. monitor RED and USE
7. keep rollback ready

## 7.10 Example sysctl file

```conf
# /etc/sysctl.d/99-performance.conf
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 20
fs.file-max = 2097152
net.core.somaxconn = 4096
net.core.netdev_max_backlog = 250000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.ip_local_port_range = 10240 65535
```

## 7.11 Kernel tuning anti-patterns

- copy-paste sysctl bundles
- tuning without baseline
- mixing unrelated changes
- raising limits with no capacity model
- ignoring kernel version differences

## 7.12 Kernel tuning practices

- prefer well-understood settings
- document reasons for every non-default value
- use configuration management
- benchmark every change
- validate after reboot and deploy

## 7.13 Kernel quick commands

```bash
sysctl -a
sysctl vm.swappiness
sysctl net.core.somaxconn
sysctl fs.file-max
cat /proc/sys/vm/swappiness
cat /proc/sys/net/core/somaxconn
sysctl --system
```

---
