# Memory Performance

[Back to guide index](README.md)

Memory analysis is about:

- virtual memory
- page cache
- reclaim
- swapping
- OOM risk
- NUMA locality
- cgroup memory pressure

## 3.1 Memory hierarchy

```mermaid
graph LR
A["Registers"] --> B["L1 Cache"]
B --> C["L2 Cache"]
C --> D["L3 Cache"]
D --> E["RAM"]
E --> F["Swap"]
```

## 3.2 Virtual memory basics

Important concepts:

- virtual addresses
- physical pages
- page tables
- TLB
- anonymous memory
- file-backed memory
- page faults
- copy-on-write

## 3.3 Page faults

Two common types:

- minor faults
- major faults

### 3.3.1 Minor fault

Usually no disk access.

### 3.3.2 Major fault

Usually requires disk access.

Check with:

```bash
pidstat -r 1 5
perf stat -e page-faults,minor-faults,major-faults ./app
```

## 3.4 `/proc/meminfo`

```bash
cat /proc/meminfo
```

Important fields:

- `MemTotal`
- `MemFree`
- `MemAvailable`
- `Buffers`
- `Cached`
- `SwapTotal`
- `SwapFree`
- `Dirty`
- `Writeback`
- `Slab`
- `PageTables`
- `AnonPages`
- `Mapped`
- `HugePages_Total`

### 3.4.1 `MemFree` vs `MemAvailable`

`MemFree` is often low on healthy systems.

`MemAvailable` is more useful.

## 3.5 `free`

```bash
free -h
```

Useful columns:

- total
- used
- free
- buff/cache
- available

Interpret carefully.

High cache is normal.

## 3.6 `vmstat`

```bash
vmstat 1 5
```

Important fields:

- `r`
- `b`
- `swpd`
- `free`
- `buff`
- `cache`
- `si`
- `so`
- `bi`
- `bo`
- `in`
- `cs`

### 3.6.1 Common patterns

| Pattern | Meaning |
|---|---|
| high `si` and `so` | swapping |
| high `b` | blocked tasks |
| low free but high cache | often healthy |
| sustained swap-out | memory pressure |

## 3.7 Slab memory

Kernel objects live in slab caches.

Use:

```bash
slabtop
cat /proc/slabinfo | head
```

Watch for:

- dentry growth
- inode growth
- socket-related slab growth
- unreclaimable slab growth

## 3.8 Page cache

Linux uses memory aggressively for page cache.

Benefits:

- fast reads
- fewer disk operations
- write coalescing

Do not confuse page cache growth with a memory leak.

## 3.9 Dirty pages and writeback

`Dirty` means modified pages not yet written.

`Writeback` means active flushing.

Related tunables:

- `vm.dirty_ratio`
- `vm.dirty_background_ratio`
- `vm.dirty_bytes`
- `vm.dirty_background_bytes`

## 3.10 Swap

Swap is not always bad.

But active working set swap is usually bad for latency.

### 3.10.1 Swappiness

Check:

```bash
sysctl vm.swappiness
```

Set temporarily:

```bash
sysctl -w vm.swappiness=10
```

### 3.10.2 Swap diagnostics

```bash
vmstat 1 5
sar -W 1 5
cat /proc/meminfo
```

## 3.11 OOM killer

When memory is exhausted, Linux may kill processes.

Useful commands:

```bash
dmesg | grep -i -E 'oom|killed process'
journalctl -k --no-pager | grep -i oom
cat /proc/<pid>/oom_score
cat /proc/<pid>/oom_score_adj
```

### 3.11.1 `oom_score_adj`

Range:

- `-1000`
- `1000`

Lower means less likely to be killed.

## 3.12 Huge pages

Types:

- Transparent Huge Pages
- explicit huge pages

### 3.12.1 THP checks

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/defrag
```

THP may help throughput.

THP may hurt latency.

Benchmark first.

### 3.12.2 Explicit huge pages

```bash
sysctl -w vm.nr_hugepages=1024
```

## 3.13 NUMA memory

Memory locality matters.

Check with:

```bash
numastat
cat /proc/<pid>/numa_maps
```

Symptoms of bad NUMA placement:

- remote access growth
- inconsistent latency
- poor scaling

## 3.14 Memory cgroups

Important cgroup v2 files:

- `memory.current`
- `memory.max`
- `memory.high`
- `memory.low`
- `memory.min`
- `memory.events`
- `memory.stat`

### 3.14.1 Meanings

- `memory.max` is a hard cap
- `memory.high` triggers pressure before hard cap
- `memory.low` protects memory from reclaim pressure
- `memory.min` is stronger protection

## 3.15 Pressure Stall Information

Check:

```bash
cat /proc/pressure/memory
```

PSI shows stalled time due to memory pressure.

## 3.16 Process memory inspection

Useful tools:

- `pmap -x <pid>`
- `/proc/<pid>/smaps`
- `/proc/<pid>/smaps_rollup`
- `smem`

## 3.17 Memory leak workflow

1. identify the process
2. check RSS and PSS growth
3. compare heap to mapped files
4. inspect cgroup events
5. check PSI
6. take application heap dump if appropriate
7. verify growth over time

## 3.18 Memory bottleneck patterns

### Pattern A

High swap and latency spikes.

Likely causes:

- memory pressure
- oversized working set
- insufficient limits

### Pattern B

High slab growth.

Likely causes:

- kernel object buildup
- filesystem cache behavior
- network structures

### Pattern C

OOM kills with free host memory.

Likely cause:

- cgroup limit too low

### Pattern D

High major faults.

Likely causes:

- cold working set
- insufficient RAM
- mmap-heavy page-in behavior

## 3.19 Memory tuning practices

- monitor `MemAvailable`
- monitor swap rates
- inspect PSI
- benchmark THP changes
- separate cache growth from leak growth
- use NUMA-aware placement
- prefer controlled cgroup protections

## 3.20 Memory quick commands

```bash
cat /proc/meminfo
free -h
vmstat 1 5
slabtop
pidstat -r 1 5
sar -B 1 5
sar -W 1 5
pmap -x <pid>
numastat
cat /proc/pressure/memory
```

---
