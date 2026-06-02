# Magic SysRq Key

This guide covers SysRq usage for emergency recovery, debugging, and controlled diagnostics.

## 6.1 What is SysRq?

Magic SysRq is a low-level Linux emergency command interface.

It can trigger special kernel actions even when the system is partly hung.

It is especially useful for:

- safe emergency reboot sequences
- forcing sync
- remounting read-only
- collecting state
- triggering panic for kdump tests

## 6.2 Enabling SysRq

Check current value:

```bash
cat /proc/sys/kernel/sysrq
```

Enable all functions temporarily:

```bash
echo 1 | sudo tee /proc/sys/kernel/sysrq
```

Disable:

```bash
echo 0 | sudo tee /proc/sys/kernel/sysrq
```

## 6.3 Persistent SysRq configuration

Example sysctl file:

```conf
# /etc/sysctl.d/99-sysrq.conf
kernel.sysrq = 1
```

Apply:

```bash
sudo sysctl --system
```

## 6.4 Numeric SysRq mask values

Some kernels use bitmasks allowing only selected functions.

Example values vary by kernel documentation.

A common operational pattern is:

- `0` disabled
- `1` enabled all functions
- controlled mask for limited features in hardened environments

## 6.5 Triggering SysRq via `/proc/sysrq-trigger`

Example:

```bash
echo t | sudo tee /proc/sysrq-trigger
```

This triggers the `t` SysRq action, which dumps task backtraces to kernel log.

## 6.6 Important SysRq commands

| Key | Meaning |
|---|---|
| `b` | immediate reboot without sync |
| `c` | trigger crash/panic |
| `e` | send SIGTERM to all processes except init |
| `f` | invoke OOM killer |
| `h` | help |
| `i` | send SIGKILL to all processes except init |
| `l` | dump backtrace for all active CPUs |
| `m` | dump memory info |
| `o` | power off |
| `p` | dump registers/current flags |
| `r` | switch keyboard from raw to XLATE |
| `s` | sync disks |
| `t` | dump task lists/backtraces |
| `u` | remount filesystems read-only |
| `w` | dump blocked tasks |

Kernel support can vary somewhat by version and architecture.

## 6.7 The REISUB sequence

`REISUB` is a mnemonic for safer emergency reboot.

It commonly means:

- `R` unraw keyboard
- `E` terminate processes
- `I` kill processes
- `S` sync disks
- `U` remount read-only
- `B` reboot

This is better than hard power cycling when the system is partially responsive.

## 6.8 REISUB explained step by step

### R

Take keyboard out of raw mode.

### E

Try graceful termination first.

### I

Kill remaining processes if needed.

### S

Flush buffered writes to disk.

### U

Remount filesystems read-only to reduce corruption risk.

### B

Reboot immediately.

## 6.9 Using SysRq for debugging

Useful debug actions include:

- `t` dump current tasks
- `w` dump blocked tasks
- `m` dump memory info
- `l` dump stacks on all CPUs
- `p` dump registers

These can be invaluable before forcing a reboot.

## 6.10 Example: gather evidence from a hung system

```bash
echo w | sudo tee /proc/sysrq-trigger
echo t | sudo tee /proc/sysrq-trigger
echo m | sudo tee /proc/sysrq-trigger
journalctl -k --since "5 min ago" --no-pager
```

If the system still responds enough to log these, you may capture crucial evidence.

## 6.11 Example: controlled panic for kdump test

```bash
echo 1 | sudo tee /proc/sys/kernel/sysrq
echo c | sudo tee /proc/sysrq-trigger
```

Use only on test systems or during maintenance.

## 6.12 Example: safer emergency reboot

```bash
echo s | sudo tee /proc/sysrq-trigger
echo u | sudo tee /proc/sysrq-trigger
echo b | sudo tee /proc/sysrq-trigger
```

This is not as graceful as a normal reboot but may be safer than a hard reset.

## 6.13 Security considerations

SysRq can be very powerful.

Enabling it broadly may not fit locked-down environments.

Consider:

- who can write to `/proc/sysrq-trigger`
- whether console access is restricted
- whether only limited SysRq masks are appropriate

## 6.14 Operational caution

`b` and `o` do not perform a normal shutdown sequence.

They can still cause data loss.

Prefer `s` and `u` first if possible.

## 6.15 SysRq in virtual consoles

Traditionally, SysRq key combos can be sent from a physical keyboard.

On remote or headless systems, `/proc/sysrq-trigger` is often more practical.

---
