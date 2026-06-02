# dmesg and Kernel Logs

This guide covers kernel log levels, filtering, persistence, and log-driven investigations.

## 9.1 Overview

`dmesg` reads the kernel ring buffer.

Kernel logs are often the fastest first source when diagnosing:

- driver issues
- boot failures
- hardware errors
- watchdog events
- OOM events
- panic precursors

## 9.2 Basic usage

```bash
dmesg
```

To make timestamps human-readable on supported versions:

```bash
dmesg -T
```

## 9.3 Filtering output

Examples:

```bash
dmesg | grep -i error
dmesg | grep -i -E 'panic|oops|BUG|watchdog'
dmesg -T | tail -100
```

## 9.4 Follow live kernel messages

```bash
dmesg -w
```

This is useful during reproduction tests.

## 9.5 Log levels

Linux log levels in descending severity:

| Level | Meaning |
|---|---|
| emerg | system unusable |
| alert | action must be taken immediately |
| crit | critical condition |
| err | error condition |
| warning | warning condition |
| notice | significant but normal |
| info | informational |
| debug | debug-level message |

## 9.6 `printk` log level prefixes

Kernel messages may carry numeric prefixes or level macros corresponding to these severities.

The console log level determines what is printed directly to console.

## 9.7 Example filtering by level

On some systems, `dmesg --level=` can be used.

Example:

```bash
dmesg --level=err,warn
```

## 9.8 Persistent kernel logs with journald

On systemd systems, kernel logs are often persisted by journald.

Useful commands:

```bash
journalctl -k --no-pager
journalctl -k -b --no-pager
journalctl -k -b -1 --no-pager
```

These allow viewing current or previous boot kernel logs.

## 9.9 Why previous boot logs matter

After a panic and reboot, the current `dmesg` may no longer contain the failing boot's messages.

Use previous boot journal or `vmcore-dmesg.txt` from kdump.

## 9.10 Common kernel messages and interpretations

| Message pattern | Typical meaning |
|---|---|
| `BUG:` | serious kernel bug or assertion |
| `Oops:` | kernel exception/oops |
| `watchdog:` | soft or hard lockup issue |
| `INFO: task ... blocked` | hung task |
| `rcu:` | RCU stall or warning |
| `Out of memory:` | OOM killer action or fatal shortage |
| `I/O error` | device or storage problem |
| `Call Trace:` | stack trace follows |

## 9.11 Example panic-related dmesg scan

```bash
journalctl -k -b -1 --no-pager | grep -i -E 'panic|BUG|Oops|watchdog|rcu|Call Trace'
```

## 9.12 Understanding timestamps

Kernel logs may show:

- monotonic timestamps since boot
- wall clock via journal formatting

When building incident timelines, convert carefully.

## 9.13 Boot-time diagnostics

`dmesg` is especially important for:

- driver initialization failures
- ACPI or firmware warnings
- PCIe enumeration problems
- filesystem mount errors
- memory map issues
- Secure Boot and lockdown messages

## 9.14 Hardware-related messages

Examples to watch:

- MCE errors
- EDAC memory errors
- AER PCIe errors
- disk timeout or reset messages
- NIC reset or DMA faults

These often precede instability.

## 9.15 OOM messages

Typical OOM log sections show:

- memory usage summary
- process list and scores
- killed task
- cgroup or system memory context

OOM is not always a kernel bug, but it can explain severe service disruptions.

## 9.16 Hung task messages

A typical message:

```text
INFO: task myapp:1234 blocked for more than 120 seconds.
```

This indicates a task stuck in uninterruptible sleep or similar blocked state.

Investigate storage, locks, and driver paths.

## 9.17 RCU stall messages

RCU stall logs often indicate:

- CPU not passing through quiescent state
- interrupt storm
- long non-preemptible section
- soft lockup or scheduler starvation

These are important precursors to panics.

## 9.18 Using `journalctl` for focused views

Examples:

```bash
journalctl -k --since "2024-01-02 08:00:00" --until "2024-01-02 08:30:00" --no-pager
journalctl -k -p err..alert --no-pager
```

## 9.19 Capturing logs before reboot

If the system is unstable but responsive:

- dump `dmesg`
- trigger SysRq task traces if useful
- preserve journal entries
- only then decide whether to reboot or force panic for kdump

## 9.20 Log flooding caution

Excessive debug logging can:

- obscure relevant messages
- increase overhead
- rotate logs quickly
- affect timing-sensitive problems

Use focused logging and save outputs quickly.

## 9.21 `vmcore-dmesg.txt`

Many kdump setups extract log buffer into `vmcore-dmesg.txt`.

This is convenient because you can inspect panic messages without opening the full dump immediately.

Still, do not stop there if the issue is complex.

## 9.22 Message prioritization heuristic

When scanning logs, prioritize:

1. earliest anomaly before panic
2. storage/network/controller errors
3. allocator warnings
4. lockup messages
5. taint or module warnings
6. final panic trace

## 9.23 Example workflow with logs

```bash
journalctl -k -b -1 --no-pager > previous_boot_kernel.log
grep -n -i -E 'panic|BUG|Oops|watchdog|rcu|blocked|error' previous_boot_kernel.log
```

## 9.24 Interpreting taint from logs

Logs often include a taint string like `Tainted: G OE`.

That means the kernel ran in a non-pristine state.

This can point to out-of-tree or proprietary modules and affect root cause direction.

## 9.25 Practical advice

Never review only the last 20 lines.

Many incidents start long before the visible crash.

Read enough history to find the first irregular event.

---
