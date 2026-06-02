# SystemTap

This guide covers SystemTap installation, probe types, scripts, and practical tracing patterns.

## 8.1 What is SystemTap?

SystemTap is a Linux tracing and instrumentation framework.

It allows you to write scripts that probe kernel and user-space behavior.

Historically, it has been a powerful option for deep diagnostics, though eBPF is increasingly preferred in some environments.

## 8.2 Typical use cases

- tracing syscalls
- timing kernel functions
- observing lock behavior
- counting events
- debugging performance anomalies
- investigating rare code paths

## 8.3 Installation

RHEL-family example:

```bash
sudo dnf install -y systemtap systemtap-runtime kernel-devel kernel-debuginfo kernel-debuginfo-common
```

Ubuntu/Debian package availability varies by release.

Example conceptually:

```bash
sudo apt install -y systemtap systemtap-runtime linux-image-$(uname -r)-dbgsym
```

SUSE package names also vary.

The key dependency is matching kernel headers and debug information.

## 8.4 Permission model

SystemTap often requires elevated privileges or group membership because it instruments the kernel.

Use strict operational controls in production.

## 8.5 Basic script structure

A simple SystemTap script has probe blocks.

Example:

```stap
probe begin {
    printf("starting\n")
}

probe end {
    printf("ending\n")
}
```

## 8.6 Running a script

```bash
sudo stap script.stp
```

## 8.7 Common probe types

| Probe | Meaning |
|---|---|
| `syscall.*` | system call activity |
| `kernel.function("foo")` | function entry |
| `kernel.function("foo").return` | function return |
| `timer.s(N)` | periodic timer |
| `process("/path/bin").function("foo")` | user-space function |

## 8.8 Example: syscall tracing

```stap
probe syscall.open {
    printf("pid=%d file=%s\n", pid(), filename)
}
```

Run:

```bash
sudo stap open_trace.stp
```

## 8.9 Example: kernel function entry probe

```stap
probe kernel.function("do_sys_open") {
    printf("open by %s pid=%d\n", execname(), pid())
}
```

## 8.10 Example: function latency measurement

```stap
global start

probe kernel.function("vfs_read") {
    start[tid()] = gettimeofday_us()
}

probe kernel.function("vfs_read").return {
    if (start[tid()]) {
        printf("vfs_read took %d us\n", gettimeofday_us() - start[tid()])
        delete start[tid()]
    }
}
```

## 8.11 Example: periodic statistics

```stap
global counts

probe syscall.read {
    counts[execname()]++
}

probe timer.s(5) {
    foreach (name in counts)
        printf("%s %d\n", name, counts[name])
}
```

## 8.12 Debugging I/O stalls with SystemTap

Conceptual strategy:

- probe block-layer functions
- record timestamps on request issue
- measure completion latency
- group by device or process

This can help distinguish kernel scheduling delay from device delay.

## 8.13 Debugging lock contention with SystemTap

Possible approach:

- probe suspected lock acquire path
- record wait and hold durations
- sample top offenders

Be cautious with overhead.

## 8.14 Example: monitoring scheduler events

```stap
probe scheduler.process_fork {
    printf("fork parent=%d child=%d\n", parent_pid(), child_pid())
}
```

## 8.15 Example: monitoring kernel function returns

```stap
probe kernel.function("tcp_sendmsg").return {
    printf("tcp_sendmsg returned %d\n", $return)
}
```

## 8.16 Pros and cons of SystemTap

| Pros | Cons |
|---|---|
| expressive scripting | setup can be heavy |
| deep kernel visibility | depends on debuginfo |
| supports complex analysis | more intrusive than some eBPF tools |
| powerful in labs | less favored than eBPF in many modern fleets |

## 8.17 When to choose SystemTap

Choose it when:

- your environment already uses it
- you need advanced scripts that already exist in Stap form
- eBPF tooling is unavailable or insufficient
- you are working in a lab with matching debuginfo ready

## 8.18 When not to choose it

Avoid or limit it when:

- production safety margin is small
- matching debuginfo is unavailable
- simple tracing can be done with ftrace or bpftrace
- the team lacks experience maintaining Stap scripts

## 8.19 Troubleshooting SystemTap setup

Common problems:

- missing debuginfo
- version mismatch between running kernel and symbols
- compiler/toolchain missing
- probe resolution failure
- permission issues

## 8.20 Simple workflow for a new script

1. identify exact question
2. choose narrow probes
3. test in lab
4. estimate overhead
5. run for short time in production if approved
6. collect and summarize output

---
