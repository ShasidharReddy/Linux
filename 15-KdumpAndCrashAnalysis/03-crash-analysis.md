# Crash Dump Analysis

This guide covers working with vmcores, symbols, and the crash utility for postmortem analysis.

## 3.1 Overview

Collecting a dump is only half the job.

You then need to analyze it in a disciplined way.

The standard Linux tool for postmortem kernel analysis is `crash`.

It combines gdb-style symbol access with Linux kernel-specific commands.

## 3.2 Required inputs

You generally need:

- `vmcore`
- matching `vmlinux` with symbols
- sometimes relevant kernel modules with debug symbols
- system details such as kernel version and hardware context

The `vmlinux` file must match the crashed kernel build.

A mismatch can produce misleading stacks and corrupted symbol resolution.

## 3.3 Installing crash utility

Ubuntu/Debian:

```bash
sudo apt install -y crash
```

RHEL/CentOS:

```bash
sudo dnf install -y crash
```

SUSE:

```bash
sudo zypper install -y crash
```

## 3.4 Locating `vmlinux`

Depending on distro, symbol-rich images may be found in:

- `/usr/lib/debug/boot/vmlinux-*`
- `/usr/lib/debug/lib/modules/<kernel>/vmlinux`
- debuginfo package locations
- manually archived build outputs

## 3.5 Opening a vmcore

Basic syntax:

```bash
crash /usr/lib/debug/lib/modules/$(uname -r)/vmlinux /var/crash/2024-01-01-12:00/vmcore
```

Or with explicit versioned path:

```bash
crash /usr/lib/debug/boot/vmlinux-5.14.0-427.el9.x86_64 /var/crash/127.0.0.1-2024-01-01-12:00/vmcore
```

## 3.6 Initial crash session checklist

When `crash` opens, gather:

- system summary
- panic message
- backtrace of crashing task
- full log buffer
- task list
- module list
- suspect memory state

Good first commands are:

```text
sys
log
bt
ps
mod
```

## 3.7 Essential `crash` command: `sys`

`sys` prints general system information.

It typically includes:

- hostname
- kernel release
- machine type
- panic string
- dumpfile info
- CPU count
- uptime

Example:

```text
crash> sys
      KERNEL: /usr/lib/debug/lib/modules/5.14.0/vmlinux
    DUMPFILE: /var/crash/vmcore
        CPUS: 16
        DATE: Tue Jan  2 08:20:31 UTC 2024
      UPTIME: 12 days, 03:12:11
LOAD AVERAGE: 2.11, 1.98, 2.03
       TASKS: 1043
    NODENAME: db-node-01
     RELEASE: 5.14.0-427.el9.x86_64
     VERSION: #1 SMP PREEMPT_DYNAMIC
     MACHINE: x86_64  (2900 Mhz)
      MEMORY: 64 GB
       PANIC: "Kernel panic - not syncing: Fatal exception"
```

## 3.8 Essential `crash` command: `log`

`log` displays kernel log buffer captured in the dump.

This is often the fastest way to spot:

- panic string
- oops text
- watchdog messages
- RCU stall warnings
- filesystem corruption messages
- machine check reports

Example:

```text
crash> log
```

Use it early.

## 3.9 Essential `crash` command: `bt`

`bt` prints backtrace for the current context.

This is crucial for identifying where panic occurred.

Example:

```text
crash> bt
```

Useful variations:

```text
crash> bt -a
crash> bt -f
crash> bt <pid>
```

Meaning:

- `-a` backtrace all tasks or CPUs depending on context/version
- `-f` show full stack data
- `<pid>` inspect a specific task

## 3.10 Reading a backtrace

A good backtrace review asks:

- what function faulted?
- what function called it?
- what lock or subsystem was active?
- was it process context, interrupt context, or softirq?
- is the fault in core kernel or a module?
- is stack corrupted?

## 3.11 Essential `crash` command: `ps`

`ps` shows tasks in the system.

Example:

```text
crash> ps
```

Useful variants:

```text
crash> ps -m
crash> ps -G
crash> ps | grep D
```

This helps identify:

- blocked tasks
- runaway processes
- kernel threads
- panicing task PID
- tasks stuck in uninterruptible sleep

## 3.12 Essential `crash` command: `vm`

`vm` shows memory usage and virtual memory state.

Example:

```text
crash> vm
```

This helps investigate:

- memory pressure
- OOM scenarios
- page statistics
- swap state
- huge page use

## 3.13 Essential `crash` command: `files`

`files` lists open file information for a task.

Example:

```text
crash> files 1234
```

Use this when a process-related kernel bug may depend on specific files, sockets, or devices.

## 3.14 Essential `crash` command: `net`

`net` displays network subsystem state.

Example:

```text
crash> net
```

This can reveal:

- socket state
- interfaces
- routing data depending on version/support
- connections related to a stuck workload

## 3.15 Essential `crash` command: `mod`

`mod` lists loaded modules.

Example:

```text
crash> mod
```

Use this to identify:

- third-party modules
- suspect drivers
- module load addresses
- whether a crash address lands inside a module region

## 3.16 Essential `crash` command: `struct`

`struct` decodes kernel structures using symbols.

Example:

```text
crash> struct task_struct ffff888012345000
```

Or inspect a field:

```text
crash> struct task_struct.pid,comm,state ffff888012345000
```

This is one of the most powerful commands for deep kernel debugging.

## 3.17 Essential `crash` command: `dis`

`dis` disassembles code around an address or symbol.

Example:

```text
crash> dis panic
crash> dis -l do_page_fault
crash> dis ffffffffa0123456
```

This is useful when a faulting RIP points inside a function and you need instruction context.

## 3.18 Essential `crash` command: `rd`

`rd` reads raw memory.

Example:

```text
crash> rd ffff888012345000 16
```

This is lower-level and mainly useful when interpreting data structures, pointers, or corrupted memory.

## 3.19 Other high-value commands

| Command | Use |
|---|---|
| `help` | command help |
| `set` | session settings |
| `kmem` | slab/page allocator inspection |
| `mount` | mounted filesystems |
| `dev` | devices |
| `irq` | interrupt info |
| `runq` | run queue state |
| `pte` | page table entries |
| `search` | memory search |
| `foreach` | iterate over tasks or objects |

## 3.20 Typical first 10 minutes in `crash`

1. run `sys`
2. run `log`
3. run `bt`
4. identify panicing task or CPU
5. run `ps`
6. inspect modules with `mod`
7. inspect suspect structure with `struct`
8. disassemble fault location with `dis -l`
9. correlate with source and symbols
10. write down a hypothesis before exploring edge cases

## 3.21 Finding root cause from a crash dump

A disciplined root cause process usually looks like this:

1. Confirm panic type.
2. Identify faulting instruction and call chain.
3. Determine subsystem involved.
4. Determine whether the fault is deterministic or secondary fallout.
5. Check logs for earlier warnings before the final panic.
6. Look for taint flags or third-party modules.
7. Correlate addresses with source lines.
8. Check whether memory corruption is likely.
9. Compare with known bugs or fixed commits.
10. Document evidence and confidence level.

## 3.22 Panic vs consequence

The top of the stack is not always the root cause.

For example:

- watchdog panic may occur after long lockup caused elsewhere
- panic in slab allocator may stem from earlier use-after-free
- filesystem fault may be secondary to memory corruption

Always scan logs and other task stacks for earlier indicators.

## 3.23 Understanding taint flags

Kernel taint flags indicate non-standard conditions such as:

- proprietary modules loaded
- forced module load
- machine check occurred
- kernel warning happened
- externally built modules present

In `crash` or logs, note taint status because it influences supportability and suspicion.

## 3.24 Example analysis: NULL pointer dereference

Suppose the log shows:

```text
BUG: kernel NULL pointer dereference, address: 0000000000000018
RIP: 0010:my_driver_handle_event+0x2a/0x80 [my_driver]
```

A likely workflow:

1. run `log` and capture full oops
2. run `mod` and confirm `my_driver`
3. use `bt` on current task
4. use `dis -l my_driver_handle_event`
5. inspect source line with matching debuginfo
6. inspect structure pointer arguments if recoverable
7. determine whether object was NULL or freed

## 3.25 Example analysis: soft lockup leading to panic

Suppose the log shows repeated watchdog lines.

Workflow:

1. run `log` and identify stuck CPU
2. run `bt -a`
3. inspect stack on stuck CPU
4. inspect lock ownership if possible
5. check if stack loops in spinlock or interrupt-disabled region
6. check for driver or filesystem hot path

## 3.26 Example analysis: slab corruption panic

Symptoms may include:

- `list_del corruption`
- `slub` assertions
- invalid freelist pointer
- page allocation metadata damage

Workflow:

1. capture exact corruption message from `log`
2. inspect involved cache with `kmem`
3. inspect nearby objects or suspect structures
4. check recent kernel warnings before panic
5. consider use-after-free or overflow patterns
6. enable KASAN or slub debugging in reproduction environment

## 3.27 Example analysis: page fault in filesystem path

Possible symptoms:

- panic in `xfs_*` or `ext4_*` function
- page fault during I/O completion
- metadata corruption messages in log

Workflow:

1. verify whether filesystem corruption preceded panic
2. inspect stack for transaction or reclaim context
3. correlate with block device errors
4. check if memory corruption could have corrupted filesystem structures in RAM
5. examine loaded storage controller modules

## 3.28 Example analysis: networking crash

Possible clues:

- panic in `skb_*`, `napi_*`, NIC driver, GRO, or TCP path
- heavy network load on affected node
- log lines around DMA or reset failures

Workflow:

1. inspect `log`
2. inspect `bt`
3. inspect `net`
4. inspect NIC module via `mod`
5. inspect ring or device structure with `struct` if symbols are available

## 3.29 Using source correlation tools outside `crash`

After identifying a function and offset, use:

```bash
addr2line -e vmlinux 0xffffffff81012345
objdump -dS vmlinux | less
nm -n vmlinux | grep symbol_name
```

These help map addresses and instructions back to source.

## 3.30 `crash` session example

```text
$ crash vmlinux vmcore
crash> sys
crash> log
crash> bt
crash> mod
crash> ps | grep myservice
crash> struct task_struct ffff8881034e8000
crash> dis -l my_driver_handle_event
crash> quit
```

## 3.31 Working with modules

For module-related crashes, ensure module debug symbols match too.

Otherwise, offsets may be hard to interpret.

Useful steps:

- note module load base from `mod`
- calculate symbol offset if necessary
- obtain debuginfo for that module version

## 3.32 Looking at all task backtraces

`bt -a` is often invaluable.

You may discover:

- system-wide lock contention
- many tasks blocked on same mutex
- deadlocked worker threads
- I/O congestion patterns
- RCU grace period blockers

## 3.33 Investigating deadlocks

Signs of deadlock may include:

- many tasks in `D` state
- lockdep warning before panic
- watchdog panic on stuck CPU
- two or more threads holding opposite locks

Use combined evidence from:

- `log`
- `ps`
- `bt -a`
- lock-related structs if accessible

## 3.34 Inspecting a task structure

Example:

```text
crash> ps | grep myservice
  PID    PPID  CPU       TASK        ST  %MEM     VSZ    RSS  COMM
 1234    1     6  ffff8881034e8000   IN   0.1  100000  15000 myservice
crash> struct task_struct.pid,comm,state,stack ffff8881034e8000
```

This can help tie a task to stack, state, and scheduling context.

## 3.35 Inspecting files for a task

Example:

```text
crash> files 1234
```

Questions to ask:

- Is the task blocked on a device file?
- Is a socket involved?
- Is the file descriptor table corrupted?

## 3.36 Memory analysis with `kmem`

`kmem` can show page allocator and slab state.

Example:

```text
crash> kmem -i
crash> kmem -s
```

Potential use cases:

- slab growth anomalies
- memory exhaustion
- corrupted caches
- page ownership investigations depending on version/features

## 3.37 Reading registers from oops text

The log may contain register dumps.

These often matter more than people think.

For example:

- `RIP` shows fault instruction pointer
- `RSP` stack pointer
- `RAX`, `RBX`, etc. may hold structure pointers
- `CR2` on x86 often indicates faulting address for page faults

## 3.38 Using `dis -l` around RIP

If the oops says:

```text
RIP: 0010:my_func+0x2a/0x90
```

Run:

```text
crash> dis -l my_func
```

Then identify the instruction at offset `+0x2a`.

This often reveals whether the fault was:

- dereferencing pointer field
- copying to bad address
- using NULL structure member
- executing in corrupted code path

## 3.39 Common evidence bundle to save

During investigation, capture:

- `sys`
- `log`
- `bt`
- `bt -a`
- `ps`
- `mod`
- any key `struct` dumps
- disassembly around fault point
- kernel version and config

This bundle helps escalation to vendors or maintainers.

## 3.40 Reproducibility and confidence levels

Not every dump yields absolute proof.

Useful confidence labels:

| Level | Meaning |
|---|---|
| confirmed | direct evidence of defect location |
| strong suspicion | evidence strongly points to component |
| possible | multiple hypotheses remain |
| inconclusive | dump too damaged or incomplete |

## 3.41 Common pitfalls in dump analysis

- using wrong `vmlinux`
- ignoring earlier warnings in `log`
- focusing only on panic thread
- misreading secondary crash as root cause
- forgetting third-party module taint
- analyzing filtered dump that omitted needed data

## 3.42 Real-world style example 1: driver use-after-free suspicion

Observed symptoms:

- panic in network driver Tx cleanup
- invalid pointer dereference
- prior warnings around reset path

Analysis path:

1. `log` shows warning before panic during interface flap.
2. `bt` shows crash in cleanup function.
3. `mod` confirms out-of-tree driver version.
4. `dis -l` shows dereference of freed ring buffer pointer.
5. internal driver structure inspection shows inconsistent ownership.
6. conclusion: likely race between device reset and cleanup path.

Recommended next steps:

- compare against upstream fixes
- reproduce with debug kernel
- enable KASAN in lab if possible

## 3.43 Real-world style example 2: ext4 panic after storage errors

Observed symptoms:

- panic inside ext4 journal path
- prior block layer I/O errors
- controller timeout messages before panic

Analysis path:

1. `log` reveals many disk timeout events before ext4 failure.
2. `bt` shows filesystem transaction code in panic path.
3. module list shows storage HBA driver at known problematic version.
4. conclusion: filesystem panic is likely consequence of storage subsystem failure.

## 3.44 Real-world style example 3: watchdog panic due to CPU soft lockup

Observed symptoms:

- `watchdog: BUG: soft lockup`
- later panic configured on lockup

Analysis path:

1. `bt -a` reveals one CPU spinning in `spin_lock` path.
2. several tasks blocked behind I/O completion.
3. suspect module holds lock in interrupt-disabled region.
4. conclusion: root cause likely lock inversion or long critical section in driver path.

## 3.45 Practical incident template

Use a template like this:

| Field | Example |
|---|---|
| node | db-node-01 |
| kernel | 5.14.0-427.el9.x86_64 |
| panic time | 2024-01-02 08:20 UTC |
| panic string | Fatal exception |
| suspected subsystem | network driver |
| faulting function | `my_driver_handle_event+0x2a` |
| root cause confidence | strong suspicion |
| supporting evidence | log, bt, disassembly, taint |
| remediation | upgrade driver, reproduce in lab |

## 3.46 Recommended learning habit

Practice analysis on known-good test dumps.

Analysts improve quickly when they repeatedly inspect:

- panic strings
- backtraces
- task states
- common structures
- allocator state

Muscle memory with `crash` matters during real incidents.

---
