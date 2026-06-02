# CPU Issues

This guide covers CPU troubleshooting plus cross-resource performance degradation triage.

## 5.1 High CPU categories

- High user CPU.
- High system CPU.
- High iowait.
- High steal time in VMs.
- High interrupt or softirq load.
- High context switch overhead.

## 5.2 First commands

```bash
uptime
top -H
ps -eo pid,ppid,user,ni,pri,stat,%cpu,comm --sort=-%cpu | head -20
mpstat -P ALL 1 5
pidstat -u -r -d -h 1 5
```

## 5.3 User vs system CPU

- High user CPU usually means application work.
- High system CPU suggests kernel work.
- Common system CPU causes include networking, filesystem, syscall-heavy loops, or interrupt load.

## 5.4 Runaway processes

Check:

```bash
top -H -p <pid>
ps -Lp <pid> -o pid,tid,psr,pcpu,stat,comm --sort=-pcpu | head -20
strace -p <pid>
```

Look for:

- Busy loops.
- Retry storms.
- Excessive logging.
- Stuck lock contention.

## 5.5 CPU steal time

In VMs, check `st` in `top` or `mpstat`.

If steal time is high:

- The hypervisor is oversubscribed.
- The guest is not receiving scheduled CPU time.
- Moving the VM or resizing may be required.

## 5.6 Interrupt storms

Check interrupt rates:

```bash
watch -n1 'cat /proc/interrupts'
```

Common causes:

- Faulty NIC behavior.
- High packet rates.
- Storage interrupt bursts.
- Misbehaving drivers.

## 5.7 Softirq overload

Check:

```bash
cat /proc/softirqs
mpstat -P ALL 1 5
```

Network-heavy systems may show high softirq CPU.

## 5.8 Context switch overhead

Check:

```bash
vmstat 1 5
pidstat -w 1 5
```

High `cs` values may indicate:

- Too many threads.
- Lock contention.
- Scheduler churn.
- Excessive short-lived processes.

## 5.9 CPU frequency and throttling

Check:

```bash
lscpu
cpupower frequency-info 2>/dev/null
journalctl -k | grep -i -E 'throttle|thermal|mce|machine check'
```

## 5.10 Load average vs CPU usage

High load does not always mean high CPU.

Load includes:

- runnable tasks.
- uninterruptible sleep tasks.

If load is high and CPU is low:

- suspect I/O wait.
- suspect stuck NFS.
- suspect blocked tasks.

## 5.11 CPU issue checklist

- Measure user, system, iowait, steal.
- Identify top processes and threads.
- Check interrupts and softirqs.
- Check context switches.
- Check throttling or thermal events.
- Correlate with deploys and traffic.

---

## Performance Degradation

## 11.1 Use the USE method

USE stands for:

- Utilization.
- Saturation.
- Errors.

Apply it to each resource:

- CPU.
- Memory.
- Disk.
- Network.

## 11.2 Performance triage commands

```bash
uptime
vmstat 1 5
iostat -xz 1 5
mpstat -P ALL 1 5
free -h
sar -n DEV 1 5
pidstat -dur 1 5
```

## 11.3 CPU bottleneck indicators

- High run queue.
- High user or system CPU.
- Low idle.
- Throttling.
- High steal.

## 11.4 Memory bottleneck indicators

- OOM events.
- Swap activity.
- Low `MemAvailable`.
- Reclaim storms.

## 11.5 Disk bottleneck indicators

- High `await`.
- High `%util`.
- Large queue depth.
- D-state tasks.

## 11.6 Network bottleneck indicators

- Drops.
- retransmits.
- low throughput.
- high latency.
- conntrack saturation.

## 11.7 Distinguish cause from victim

A slow database can cause web workers to pile up.

Those web workers may consume memory and threads.

The database may be the cause.

The web tier may be the victim.

## 11.8 Compare to baseline

Always ask:

- What is normal for this host?
- What changed compared to yesterday?
- Is the issue absolute or relative to historical baseline?

## 11.9 Workload awareness

Some hosts are bursty by design.

Some workloads are batch oriented.

Do not confuse expected peaks with regressions.

## 11.10 Performance checklist

- Check CPU.
- Check memory.
- Check disk latency.
- Check network errors.
- Check application dependency latency.
- Compare before and after change.
- Measure after each mitigation.

---
