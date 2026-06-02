# Linux Performance Tuning Guide


---

## 🎬 Performance Analysis — Animated Workflow

```mermaid
graph TD
    subgraph METHODOLOGY["🔍 USE Method (Utilization, Saturation, Errors)"]
        direction TB
        START_P["🚨 Performance Issue"]:::alert --> CPU["🔲 CPU Check"]:::resource
        CPU --> MEM["🧠 Memory Check"]:::resource
        MEM --> DISK["💾 Disk I/O Check"]:::resource
        DISK --> NET_P["🌐 Network Check"]:::resource
        NET_P --> FOUND{"🔍 Bottleneck?"}:::decision
        FOUND -->|Yes| FIX["🔧 Tune & Fix"]:::fix
        FOUND -->|No| DEEPER["🔬 Deep Analysis"]:::deep
    end

    subgraph TOOLS["🧰 Performance Toolkit"]
        direction LR
        T1["top/htop"]:::basic_t --> T2["vmstat/iostat"]:::basic_t
        T2 --> T3["sar/dstat"]:::mid_t
        T3 --> T4["perf record"]:::adv_t
        T4 --> T5["🔥 Flamegraph"]:::adv_t
        T5 --> T6["eBPF/bcc"]:::expert_t
    end

    subgraph TUNING["⚡ Tuning Areas"]
        direction TB
        TN1["🔲 CPU: nice, taskset, cgroups"]:::tune
        TN2["🧠 Memory: vm.swappiness, huge pages"]:::tune
        TN3["💾 Disk: I/O scheduler, readahead"]:::tune
        TN4["🌐 Network: TCP buffers, congestion"]:::tune
        TN1 --> BENCH["📊 Benchmark"]:::bench
        TN2 --> BENCH
        TN3 --> BENCH
        TN4 --> BENCH
        BENCH --> COMPARE["📈 Compare Results"]:::compare
    end

    classDef alert fill:#F44336,stroke:#C62828,color:#fff,stroke-width:3px
    classDef resource fill:#2196F3,stroke:#1565C0,color:#fff
    classDef decision fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:2px
    classDef fix fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:2px
    classDef deep fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef basic_t fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef mid_t fill:#FF9800,stroke:#E65100,color:#fff
    classDef adv_t fill:#e94560,stroke:#b71c1c,color:#fff
    classDef expert_t fill:#b80000,stroke:#7f0000,color:#fff,stroke-width:2px
    classDef tune fill:#0f3460,stroke:#16213e,color:#fff
    classDef bench fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:2px
    classDef compare fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
```

---

## Overview

A production-grade guide for Linux performance analysis and tuning. It is designed for operators, SREs, platform engineers, and developers, and it emphasizes measurement before tuning with practical, production-safe habits.

## Learning Path

```mermaid
graph TD
    N1["01 Performance Methodology"]
    N2["02 CPU Performance"]
    N3["03 Memory Performance"]
    N4["04 Disk I/O Performance"]
    N5["05 Network Performance"]
    N6["06 System Profiling"]
    N7["07 Kernel Tuning"]
    N8["08 Application Performance"]
    N9["09 Benchmarking"]
    N10["10 Troubleshooting Scenarios"]
    N1 --> N2
    N2 --> N3
    N3 --> N4
    N4 --> N5
    N5 --> N6
    N6 --> N7
    N7 --> N8
    N8 --> N9
    N9 --> N10
```

## Table of Contents

- [Performance Methodology](01-methodology.md)
- [CPU Performance](02-cpu.md)
- [Memory Performance](03-memory.md)
- [Disk I/O Performance](04-disk-io.md)
- [Network Performance](05-network.md)
- [System Profiling](06-profiling.md)
- [Kernel Tuning](07-kernel-tuning.md)
- [Application Performance](08-application.md)
- [Benchmarking](09-benchmarking.md)
- [Troubleshooting Scenarios](10-troubleshooting.md)
