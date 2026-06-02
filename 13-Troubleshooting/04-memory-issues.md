# Memory Issues

## 4.1 Understand Linux memory reporting

Key concepts:

- Free memory alone is not enough.
- Linux uses memory for caches aggressively.
- `available` memory is more useful than `free`.
- Swap use is not always a problem.
- Sudden growth patterns matter more than absolute numbers.

## 4.2 First commands

```bash
free -h
vmstat 1 5
cat /proc/meminfo
ps -eo pid,user,comm,%mem,rss,vsz --sort=-rss | head -20
smem -tk 2>/dev/null | head -20
```

## 4.3 Key `/proc/meminfo` fields

| Field | Meaning |
|---|---|
| `MemTotal` | Total RAM |
| `MemFree` | Completely unused RAM |
| `MemAvailable` | Estimated allocatable RAM without major swapping |
| `Buffers` | Metadata buffers |
| `Cached` | Page cache |
| `SReclaimable` | Reclaimable slab |
| `SwapTotal` | Total swap |
| `SwapFree` | Free swap |
| `Dirty` | Dirty pages awaiting writeback |
| `Writeback` | Pages being written |
| `Slab` | Kernel object caches |

## 4.4 OOM killer basics

Symptoms:

- Processes are killed unexpectedly.
- Kernel logs mention OOM.
- Service restarts without app-level crash logs.

Check:

```bash
journalctl -k --no-pager | grep -i 'out of memory\|oom-killer\|killed process'
```

Questions:

- Which process was killed?
- What was its RSS?
- Was cgroup memory limit involved?
- Was the host under global pressure or only one container?

## 4.5 Container vs host OOM

For cgroup-limited workloads:

```bash
systemd-cgls
cat /sys/fs/cgroup/memory.max 2>/dev/null
cat /sys/fs/cgroup/memory.current 2>/dev/null
```

Container tools:

```bash
docker stats --no-stream
kubectl top pod -A
```

## 4.6 Memory leaks

Signs:

- Resident set grows steadily.
- Restarts temporarily fix the issue.
- Garbage-collected runtimes show heap growth after normal load.

Approach:

- Establish baseline growth rate.
- Compare memory over time.
- Capture heap dumps when supported.
- Correlate growth with traffic or jobs.
- Identify cache misuse or object retention.

## 4.7 Swap thrashing

Symptoms:

- System is very slow.
- `si` and `so` in `vmstat` are high.
- Load average is high with low CPU progress.

Check:

```bash
vmstat 1 10
sar -W 1 5
```

High swap-in or swap-out indicates memory pressure.

## 4.8 Page cache vs application memory

Do not drop caches reflexively.

Consider:

- Is page cache actually the problem?
- Is reclaimable memory available?
- Is slab unusually large?
- Are anonymous pages dominating?

## 4.9 Kernel memory pressure

Check slab usage:

```bash
slabtop -o
cat /proc/slabinfo | head
```

Use cases:

- Network conntrack growth.
- Dentry or inode cache explosion.
- Kernel memory leaks.

## 4.10 `top` and `htop` interpretation

Focus on:

- RES.
- SHR.
- total threads.
- `%MEM`.
- zombie count.
- swap usage.

## 4.11 Overcommit behavior

Relevant sysctls:

```bash
sysctl vm.overcommit_memory
sysctl vm.overcommit_ratio
```

Use with care.

Changing overcommit policy can reduce allocation failures but may worsen OOM outcomes.

## 4.12 Huge pages and THP

Check:

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
grep -i huge /proc/meminfo
```

Transparent Huge Pages can help some workloads and hurt others.

## 4.13 NUMA effects

On NUMA hosts:

```bash
numactl --hardware
numastat
```

Symptoms:

- Free memory exists on one node but not another.
- Latency-sensitive workloads behave inconsistently.

## 4.14 Memory incident checklist

- Check OOM logs.
- Check `MemAvailable`.
- Check swap activity.
- Identify top RSS processes.
- Determine host vs container scope.
- Review recent deploys.
- Review leak patterns over time.
- Apply limits or fix the leak.

---
