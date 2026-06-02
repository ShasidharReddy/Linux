# Advanced Linux Internals


---

## 🎬 Kernel Internals — Animated Workflow

```mermaid
graph TD
    subgraph SYSCALL["🔌 System Call Flow"]
        direction TB
        APP_I["👤 User App"]:::user --> GLIBC["📚 glibc wrapper"]:::lib
        GLIBC --> TRAP["⚡ INT 0x80 / SYSCALL"]:::trap
        TRAP --> ENTRY["🚪 sys_call_table"]:::kernel
        ENTRY --> HANDLER["🔧 Kernel Handler"]:::kernel
        HANDLER --> RESULT["📤 Return to User"]:::user
    end

    subgraph PROCESS["🔄 Process States"]
        direction LR
        NEW["🆕 Created"]:::new --> READY["📋 Ready"]:::ready
        READY --> RUNNING["🟢 Running"]:::running
        RUNNING --> BLOCKED["🔴 Blocked (I/O)"]:::blocked
        BLOCKED --> READY
        RUNNING --> ZOMBIE["💀 Zombie"]:::zombie
        ZOMBIE --> REAPED["🗑️ Reaped"]:::done
        RUNNING --> STOPPED["⏸️ Stopped"]:::stopped
        STOPPED --> READY
    end

    subgraph MEMORY["🧠 Memory Management"]
        direction TB
        VA["📍 Virtual Address"]:::virtual --> MMU["🔧 MMU"]:::mmu
        MMU --> TLB{"⚡ TLB Hit?"}:::tlb
        TLB -->|Yes| PA["💾 Physical Address"]:::physical
        TLB -->|No| PT["📋 Page Table Walk"]:::pagetable
        PT --> PA
        PT --> PF["❌ Page Fault"]:::fault
        PF --> SWAP["💾 Swap In"]:::swap
        SWAP --> PA
    end

    classDef user fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef lib fill:#2196F3,stroke:#1565C0,color:#fff
    classDef trap fill:#F44336,stroke:#C62828,color:#fff,stroke-width:2px
    classDef kernel fill:#1a1a2e,stroke:#e94560,color:#fff,stroke-width:2px
    classDef new fill:#2196F3,stroke:#1565C0,color:#fff
    classDef ready fill:#FF9800,stroke:#E65100,color:#fff
    classDef running fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef blocked fill:#F44336,stroke:#C62828,color:#fff
    classDef zombie fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef done fill:#607D8B,stroke:#37474F,color:#fff
    classDef stopped fill:#795548,stroke:#4E342E,color:#fff
    classDef virtual fill:#2196F3,stroke:#1565C0,color:#fff
    classDef mmu fill:#FF9800,stroke:#E65100,color:#fff
    classDef tlb fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:2px
    classDef physical fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef pagetable fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef fault fill:#F44336,stroke:#C62828,color:#fff
    classDef swap fill:#e94560,stroke:#b71c1c,color:#fff
```

---

This guide is now split into focused deep-dive files so you can study a subsystem in isolation or follow the full internals path from kernel architecture to tracing.

## Overview

- Start with kernel architecture, process internals, and memory management to build a mental model.
- Use the subsystem guides for filesystems, I/O, networking, eBPF, namespaces, cgroups, syscalls, IPC, and drivers.
- Finish with tracing and debugging for practical observability, command references, walkthroughs, glossary material, and recap notes.

## Learning Path

```mermaid
graph LR
A["Kernel Architecture"] --> B["Process Internals"]
B --> C["Memory Management"]
C --> D["File System Internals"]
D --> E["I/O Subsystem"]
E --> F["Network Stack"]
F --> G["eBPF"]
G --> H["Namespaces and Cgroups"]
H --> I["System Calls"]
I --> J["IPC"]
J --> K["Device Drivers"]
K --> L["Tracing and Debugging"]
```

## Table of Contents

1. [Kernel Architecture](01-kernel-architecture.md)
2. [Process Internals](02-process-internals.md)
3. [Memory Management](03-memory-management.md)
4. [Filesystem Internals](04-filesystem-internals.md)
5. [I/O Subsystem](05-io-subsystem.md)
6. [Network Stack](06-network-stack.md)
7. [eBPF](07-ebpf.md)
8. [Namespaces and Cgroups](08-namespaces-cgroups.md)
9. [System Calls](09-syscalls.md)
10. [IPC](10-ipc.md)
11. [Device Drivers](11-device-drivers.md)
12. [Tracing and Debugging](12-tracing-debugging.md)
