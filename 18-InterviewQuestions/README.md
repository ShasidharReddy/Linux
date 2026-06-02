# Linux Interview Questions Guide


---

## 🎬 Interview Prep Strategy — Animated Workflow

```mermaid
graph TD
    subgraph PREP["📝 Interview Preparation Path"]
        direction TB
        S1["🟢 Basic Questions"]:::basic_q --> S2["🟡 Intermediate Questions"]:::mid_q
        S2 --> S3["🔴 Advanced Questions"]:::adv_q
        S3 --> S4["🎯 Scenario-Based"]:::scenario
        S4 --> S5["🔧 DevOps/SRE Questions"]:::devops_q
        S5 --> S6["💡 Architecture & Design"]:::arch
        S6 --> S7["⚡ One-Liners & Quick Ref"]:::quick
        S7 --> S8["🏆 Interview Ready!"]:::ready
    end

    subgraph TOPICS["🧠 Key Topics to Master"]
        direction LR
        T1["🐧 Linux Basics"]:::topic --> T2["🐚 Shell/Scripting"]:::topic
        T2 --> T3["🌐 Networking"]:::topic
        T3 --> T4["🔒 Security"]:::topic
        T4 --> T5["🐳 Containers"]:::topic
        T5 --> T6["☸️ Kubernetes"]:::topic
        T6 --> T7["📊 Monitoring"]:::topic
    end

    subgraph TIPS["💡 Pro Tips"]
        direction TB
        TIP1["🗣️ Think out loud"]:::tip
        TIP2["📋 Structured approach"]:::tip
        TIP3["🔍 Ask clarifying questions"]:::tip
        TIP4["💻 Practice hands-on"]:::tip
    end

    classDef basic_q fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef mid_q fill:#FF9800,stroke:#E65100,color:#fff
    classDef adv_q fill:#e94560,stroke:#b71c1c,color:#fff
    classDef scenario fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef devops_q fill:#00BCD4,stroke:#00838F,color:#fff
    classDef arch fill:#795548,stroke:#4E342E,color:#fff
    classDef quick fill:#2196F3,stroke:#1565C0,color:#fff
    classDef ready fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef topic fill:#0f3460,stroke:#16213e,color:#fff
    classDef tip fill:#FF9800,stroke:#E65100,color:#fff
```

---

This guide is now split into focused interview-prep files so you can study by level, practice production scenarios, and keep quick-reference material close at hand.

## Overview

- Start with the basic, intermediate, and advanced question sets to build depth in order.
- Use the scenario and DevOps/SRE guides for real-world troubleshooting and platform-oriented interview practice.
- Use the one-liner, architecture, and quick-reference guides for fast revision before interviews.

## Learning Path

```mermaid
graph LR
A["Basic Questions"] --> B["Intermediate Questions"]
B --> C["Advanced Questions"]
C --> D["Scenario Questions"]
D --> E["DevOps and SRE"]
E --> F["One-Liners"]
F --> G["Architecture and Design"]
G --> H["Quick Reference"]
```

## Table of Contents

1. [Basic Questions](01-basic-questions.md)
2. [Intermediate Questions](02-intermediate-questions.md)
3. [Advanced Questions](03-advanced-questions.md)
4. [Scenario Questions](04-scenario-questions.md)
5. [DevOps and SRE Questions](05-devops-sre-questions.md)
6. [One-Liners](06-one-liners.md)
7. [Architecture and Design](07-architecture-design.md)
8. [Quick Reference](08-quick-reference.md)
