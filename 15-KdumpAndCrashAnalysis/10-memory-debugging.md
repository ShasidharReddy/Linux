# Memory Debugging

This guide covers KASAN, UBSAN, kmemleak, allocator diagnostics, and memory-corruption analysis.

## 10.1 Overview

Memory bugs in the kernel are among the hardest issues to diagnose.

Examples include:

- use-after-free
- out-of-bounds access
- double free
- uninitialized reads
- allocator metadata corruption
- leaks

Linux provides several tools to help detect or narrow these problems.

## 10.2 KASAN

KASAN is the Kernel Address Sanitizer.

It detects invalid memory access bugs such as:

- heap out-of-bounds
- stack out-of-bounds in supported modes
- use-after-free
- use-after-scope in certain contexts

It is extremely valuable in reproducing memory corruption bugs in a lab.

## 10.3 KASAN trade-offs

KASAN has substantial memory and performance overhead.

It is typically used in test or debug kernels rather than production.

## 10.4 Interpreting KASAN reports

A KASAN report usually includes:

- bug class
- faulting access type and size
- stack trace for invalid access
- stack trace of allocation
- stack trace of free if applicable
- shadow memory info

This often makes root cause much clearer than a later generic panic.

## 10.5 UBSAN

UBSAN is the Undefined Behavior Sanitizer.

It detects undefined behavior such as:

- signed integer overflow in selected cases
- shift out of bounds
- invalid enum values
- misaligned access
- type violations depending on configuration

Useful especially when code depends on assumptions that fail only under rare inputs.

## 10.6 kmemleak

`kmemleak` detects kernel memory leaks by scanning for unreachable allocations.

It is useful when memory growth suggests leaked kernel objects.

## 10.7 Using kmemleak

If enabled in kernel config, interface typically exists at:

```text
/sys/kernel/debug/kmemleak
```

Example:

```bash
echo scan | sudo tee /sys/kernel/debug/kmemleak
cat /sys/kernel/debug/kmemleak
```

## 10.8 kmemcheck

`kmemcheck` is an older memory checking facility.

It is less common in modern practice, but historically helped detect use of uninitialized memory.

Current environments are more likely to rely on KASAN and related sanitizers.

## 10.9 `slub_debug`

SLUB allocator debugging options help detect heap corruption.

Boot parameter example:

```bash
slub_debug=FZPU
```

Common flags can enable:

- consistency checks
- red zones
- poisoning
- user tracking in some configurations

## 10.10 Why `slub_debug` helps

It can turn silent corruption into earlier, more localized failures.

Earlier failure points are often easier to debug than late random panics.

## 10.11 Memory poisoning

Poisoning fills freed or allocated objects with patterns.

This helps catch use-after-free or uninitialized use by making bugs more obvious during testing.

## 10.12 Page allocator debugging

Additional debugging features can help catch page-level corruption or allocation misuse.

These vary by kernel config and debug options.

## 10.13 Detecting use-after-free

Strong tools for this include:

- KASAN
- SLUB poisoning/red zones
- targeted reproducer plus `crash` analysis
- code audit of allocation/free paths

## 10.14 Detecting out-of-bounds writes

Useful tools:

- KASAN
- SLUB red zones
- hardware watchpoints in lab via KGDB or VM tools
- focused tracing around object lifecycle

## 10.15 Detecting leaks

Useful tools:

- kmemleak
- slab growth observation through `slabtop`
- `crash` `kmem` analysis on vmcore
- workload-specific allocation counters

## 10.16 `slabtop`

On a running system:

```bash
slabtop
```

This helps identify caches growing unusually large.

It is not a proof of leak, but it is a good symptom finder.

## 10.17 Typical memory corruption clues in logs

- `list_del corruption`
- `BUG kmalloc-...`
- `general protection fault` in unrelated code
- allocator freelist errors
- `bad page state`
- `page allocation failure` under normal load

## 10.18 Reproduction strategy for memory bugs

1. reproduce on debug kernel if possible
2. enable KASAN or `slub_debug`
3. isolate workload that triggers corruption
4. reduce concurrency only if it still reproduces
5. capture earliest report, not just final panic

## 10.19 Production-safe strategies

In production, heavy sanitizers may be impractical.

Safer options include:

- kdump for postmortem capture
- targeted ftrace or eBPF counters
- dynamic debug in suspect subsystem
- mirroring traffic/workload in lab reproduction

## 10.20 Memory debugging checklist

- exact symptom class identified
- allocator or object type known
- earliest warning captured
- object lifecycle understood
- debug build or sanitizer plan defined
- reproduction minimized and repeatable if possible

---
