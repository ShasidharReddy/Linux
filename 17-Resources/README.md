# Linux Resources and Bookmarks Guide


---

## 🎬 Learning Resources Map — Animated Guide

```mermaid
graph TD
    subgraph PATH["📚 Resource Categories"]
        direction TB
        R1["🌐 Online Platforms"]:::online --> R2["📖 Documentation"]:::docs
        R2 --> R3["🔧 Practice Labs"]:::labs
        R3 --> R4["📜 Certifications"]:::certs
        R4 --> R5["🏆 Mastery"]:::mastery
    end

    subgraph TOOLS_R["🔧 Essential Bookmarks"]
        direction LR
        T1["cidr.xyz"]:::tool_r --> T2["uptime.is"]:::tool_r
        T2 --> T3["crontab.guru"]:::tool_r
        T3 --> T4["explainshell"]:::tool_r
        T4 --> T5["shellcheck"]:::tool_r
        T5 --> T6["regex101"]:::tool_r
    end

    subgraph COMMUNITY["👥 Community"]
        direction LR
        C1["📋 Stack Overflow"]:::community_r
        C2["💬 Reddit r/linux"]:::community_r
        C3["📧 Mailing Lists"]:::community_r
        C4["🐙 GitHub"]:::community_r
    end

    classDef online fill:#2196F3,stroke:#1565C0,color:#fff
    classDef docs fill:#FF9800,stroke:#E65100,color:#fff
    classDef labs fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef certs fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef mastery fill:#e94560,stroke:#b71c1c,color:#fff,stroke-width:3px
    classDef tool_r fill:#0f3460,stroke:#16213e,color:#fff
    classDef community_r fill:#607D8B,stroke:#37474F,color:#fff
```

---

This guide is now split into focused reference collections so you can jump straight to networking, Linux documentation, security, performance, container, package, learning, and quick-reference material.

## Overview

- Use the networking, security, performance, container, and package guides when you need authoritative references fast.
- Use the SRE, Linux docs, and learning guides for deeper study, incident handling, and day-two operations.
- Use the cheat sheet and RFC guides for rapid lookup during troubleshooting, design, and interviews.

## Learning Path

```mermaid
graph LR
A["Networking"] --> B["SRE and Uptime"]
B --> C["Linux Docs"]
C --> D["Security"]
D --> E["Performance"]
E --> F["Containers and K8s"]
F --> G["Package Repos"]
G --> H["Learning"]
H --> I["Cheat Sheets"]
I --> J["Essential RFCs"]
```

## Table of Contents

1. [Networking References](01-networking-refs.md)
2. [SRE and Uptime](02-sre-uptime.md)
3. [Linux Documentation](03-linux-docs.md)
4. [Security References](04-security-refs.md)
5. [Performance References](05-performance-refs.md)
6. [Container and Kubernetes References](06-container-k8s.md)
7. [Package Repositories](07-package-repos.md)
8. [Learning and Practice](08-learning.md)
9. [Cheat Sheets and Quick References](09-cheatsheets.md)
10. [Essential RFCs](10-essential-rfcs.md)
