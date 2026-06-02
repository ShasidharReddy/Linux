# Core Dumps

This guide covers userspace core dump generation, storage, collection, and GDB analysis workflows.

## 5.1 Overview

Application core dumps are different from kernel dumps.

They capture a user-space process memory image when the process crashes.

These are analyzed with `gdb` or similar tools.

Use core dumps when the problem is in an application, shared library, or runtime rather than the kernel.

## 5.2 Core dump prerequisites

The kernel must permit dump generation.

The process or service environment must not suppress it.

Common requirements:

- `ulimit -c` not zero
- valid `core_pattern`
- writable target path or coredump handler
- service manager not restricting dumps

## 5.3 Checking current core size limit

```bash
ulimit -c
```

Typical output:

- `0` means disabled
- `unlimited` means enabled without size cap

## 5.4 Temporarily enabling core dumps

```bash
ulimit -c unlimited
```

This affects the current shell and child processes.

## 5.5 Persistent core limit settings

Options include:

- `/etc/security/limits.conf`
- systemd service limits
- shell startup files for specific contexts

Example:

```conf
* soft core unlimited
* hard core unlimited
```

## 5.6 `core_pattern`

The kernel uses `/proc/sys/kernel/core_pattern` to determine how core dumps are named or handled.

Show current value:

```bash
cat /proc/sys/kernel/core_pattern
```

Example simple pattern:

```text
core.%e.%p.%t
```

## 5.7 Setting core dump path and naming

Example:

```bash
echo '/var/coredumps/core.%e.%p.%t' | sudo tee /proc/sys/kernel/core_pattern
```

This stores core files under `/var/coredumps` with executable name, PID, and timestamp.

## 5.8 Useful `core_pattern` tokens

| Token | Meaning |
|---|---|
| `%e` | executable name |
| `%p` | PID |
| `%t` | UNIX timestamp |
| `%h` | hostname |
| `%u` | uid |
| `%g` | gid |
| `%s` | signal number |

## 5.9 Persistent sysctl configuration

Example file:

```conf
# /etc/sysctl.d/99-coredump.conf
kernel.core_pattern=/var/coredumps/core.%e.%p.%t
```

Apply with:

```bash
sudo sysctl --system
```

## 5.10 systemd-coredump overview

Many modern distributions route core dumps through `systemd-coredump`.

This means `core_pattern` may point to a pipe handler instead of a plain file path.

Example value:

```text
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %e
```

In this model, dumps are indexed and managed by systemd tooling.

## 5.11 Using `coredumpctl`

List dumps:

```bash
coredumpctl list
```

Show metadata for latest dump:

```bash
coredumpctl info
```

Debug latest dump directly in gdb:

```bash
coredumpctl gdb
```

Debug dump for specific executable:

```bash
coredumpctl gdb /usr/bin/myapp
```

## 5.12 Example `coredumpctl list` output

```text
TIME                            PID  UID  GID SIG COREFILE EXE
Tue 2024-01-02 10:00:00 UTC    4242 1000 1000 11 present  /usr/local/bin/myapp
```

## 5.13 GDB basics for core analysis

Open a binary with its core:

```bash
gdb /usr/local/bin/myapp /var/coredumps/core.myapp.4242.1704189600
```

If using systemd-coredump, `coredumpctl gdb` is often easier.

## 5.14 Essential GDB commands

| Command | Purpose |
|---|---|
| `bt` | backtrace |
| `thread apply all bt` | backtrace all threads |
| `info threads` | list threads |
| `info registers` | CPU register state |
| `frame N` | switch stack frame |
| `print var` | inspect variable |
| `x/16gx addr` | inspect memory |
| `disassemble` | show instructions |

## 5.15 Example GDB session

```text
(gdb) bt
(gdb) info threads
(gdb) thread apply all bt
(gdb) frame 2
(gdb) info locals
(gdb) print my_ptr
(gdb) info registers
```

## 5.16 What to inspect in an application core

- crashing thread backtrace
- all thread backtraces
- signal that triggered crash
- local variables in faulting frame
- heap pointers and object lifetimes
- shared library versions
- whether optimized-out variables limit analysis

## 5.17 Compiling for usable cores

For in-house binaries, build with debug symbols.

Example:

```bash
gcc -g -O0 -o app app.c
```

In production, you may use optimized builds plus split debug symbol packages.

## 5.18 Stripped binaries and separate debuginfo

A stripped production binary is common.

That is fine if separate debuginfo is available.

Without it, backtraces may be far less useful.

## 5.19 Generating a core dump on demand with `gcore`

`gcore` can dump a running process without crashing it.

Example:

```bash
gcore -o core.snapshot 4242
```

This is useful for investigating a hung or misbehaving process.

## 5.20 Signals that generate core dumps

Common signals include:

- `SIGSEGV`
- `SIGABRT`
- `SIGILL`
- `SIGFPE`
- `SIGBUS`
- sometimes `SIGQUIT` depending on handler behavior

## 5.21 Example forced core with abort

A process calling `abort()` typically generates `SIGABRT` and a core dump if enabled.

This is useful for assertion failures in applications.

## 5.22 Core dump security considerations

Core dumps may contain:

- credentials
- secrets in memory
- customer data
- encryption keys
- tokens

Protect storage, retention, access, and transfer paths accordingly.

## 5.23 systemd service configuration example

A service may need:

```ini
[Service]
LimitCORE=infinity
```

Then reload and restart the service.

## 5.24 Debugging containerized applications

Core dump behavior in containers depends on:

- runtime configuration
- ulimits
- namespace setup
- writable storage path
- host `core_pattern`

Container core dump collection often needs special design.

## 5.25 Distinguishing application vs kernel failure

Use this simple rule:

- application exits with signal and core file: user-space crash
- system panics or reboots: likely kernel or hardware-level failure

Sometimes both occur in sequence, so correlate timestamps carefully.

---
