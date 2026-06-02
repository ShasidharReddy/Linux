# Kdump, Crash Analysis, and Kernel Debugging Guide


---

## 🎬 Crash Analysis — Animated Workflow

```mermaid
graph TD
    subgraph KDUMP["💥 Kdump Capture Flow"]
        direction TB
        K1["🔴 Kernel Panic!"]:::panic --> K2["⚡ kexec triggers"]:::kexec
        K2 --> K3["🧠 Capture kernel boots"]:::capture
        K3 --> K4["💾 Dump memory to /var/crash"]:::dump
        K4 --> K5["🔄 System reboots"]:::reboot
        K5 --> K6["📁 vmcore available"]:::vmcore
    end

    subgraph ANALYSIS["🔍 Crash Analysis Steps"]
        direction LR
        A1["crash vmlinux vmcore"]:::cmd --> A2["bt (backtrace)"]:::cmd
        A2 --> A3["log (dmesg)"]:::cmd
        A3 --> A4["ps (processes)"]:::cmd
        A4 --> A5["files (open files)"]:::cmd
        A5 --> A6["mod (modules)"]:::cmd
        A6 --> A7["�� Root Cause"]:::found
    end

    subgraph DEBUG["🔧 Debug Toolkit"]
        direction TB
        DB1["💥 kdump/crash"]:::expert_d
        DB2["🔍 ftrace"]:::expert_d
        DB3["📊 SystemTap"]:::expert_d
        DB4["🧪 kprobes"]:::expert_d
        DB5["🔬 perf probe"]:::expert_d
        DB1 --> DB2 --> DB3 --> DB4 --> DB5
    end

    classDef panic fill:#F44336,stroke:#C62828,color:#fff,stroke-width:3px
    classDef kexec fill:#FF9800,stroke:#E65100,color:#fff
    classDef capture fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef dump fill:#e94560,stroke:#b71c1c,color:#fff
    classDef reboot fill:#2196F3,stroke:#1565C0,color:#fff
    classDef vmcore fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef cmd fill:#0f3460,stroke:#16213e,color:#fff
    classDef found fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef expert_d fill:#b80000,stroke:#7f0000,color:#fff,stroke-width:2px
```

---

This guide is now split into focused runbooks so you can move from capture setup to postmortem analysis, live debugging, and production operating practices.

## Overview

- Start with the kdump overview and setup guides to understand capture flow and validate your environment.
- Use the crash, panic, core dump, SysRq, kernel debugging, SystemTap, logging, and memory debugging guides during investigations.
- Finish with production practices for appendices, reference tables, FAQs, and incident playbooks.

## Learning Path

```mermaid
graph LR
A["Kdump Overview"] --> B["Kdump Setup"]
B --> C["Crash Analysis"]
C --> D["Kernel Panic"]
D --> E["Core Dumps"]
E --> F["SysRq"]
F --> G["Kernel Debugging"]
G --> H["SystemTap"]
H --> I["Kernel Logs"]
I --> J["Memory Debugging"]
J --> K["Kernel Troubleshooting"]
K --> L["Production Practices"]
```

## Table of Contents

1. [Kdump Overview](01-kdump-overview.md)
2. [Kdump Setup](02-kdump-setup.md)
3. [Crash Analysis](03-crash-analysis.md)
4. [Kernel Panic Analysis](04-kernel-panic.md)
5. [Core Dumps](05-core-dumps.md)
6. [Magic SysRq Key](06-sysrq.md)
7. [Kernel Debugging Tools](07-kernel-debugging.md)
8. [SystemTap](08-systemtap.md)
9. [dmesg and Kernel Logs](09-dmesg-kernel-logs.md)
10. [Memory Debugging](10-memory-debugging.md)
11. [Kernel Troubleshooting](11-kernel-troubleshooting.md)
12. [Production Practices](12-production-practices.md)
