# Linux Automation & Configuration Management Guide


---

## 🎬 Infrastructure Automation — Animated Workflow

```mermaid
graph TD
    subgraph IaC["🤖 Infrastructure as Code Flow"]
        direction TB
        CODE["📝 Define Infrastructure"]:::code --> VCS["📂 Git Repository"]:::vcs
        VCS --> PLAN["📋 Plan Changes"]:::plan
        PLAN --> REVIEW{"🔍 Review?"}:::review
        REVIEW -->|Approve| APPLY["🚀 Apply"]:::apply
        REVIEW -->|Reject| CODE
        APPLY --> VERIFY["✅ Verify State"]:::verify
    end

    subgraph CONFIG["⚙️ Configuration Management"]
        direction LR
        ANSIBLE["🤖 Ansible"]:::tool --> |Playbook| TARGETS["🖥️ Target Servers"]:::target
        PUPPET["🎭 Puppet"]:::tool --> |Manifest| TARGETS
        CHEF["👨‍🍳 Chef"]:::tool --> |Recipe| TARGETS
        SALT["🧂 Salt"]:::tool --> |State| TARGETS
        TARGETS --> CONFIGURED["✅ Configured"]:::done
    end

    subgraph PIPELINE["🔄 GitOps Workflow"]
        direction LR
        PR["📝 Pull Request"]:::pr --> MERGE["🔀 Merge"]:::merge
        MERGE --> DETECT["🔍 Change Detected"]:::detect
        DETECT --> SYNC["🔄 Sync to Cluster"]:::sync
        SYNC --> HEALTHY["💚 Healthy"]:::healthy
    end

    classDef code fill:#2196F3,stroke:#1565C0,color:#fff
    classDef vcs fill:#0f3460,stroke:#16213e,color:#fff
    classDef plan fill:#FF9800,stroke:#E65100,color:#fff
    classDef review fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:3px
    classDef apply fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:2px
    classDef verify fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef tool fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef target fill:#607D8B,stroke:#37474F,color:#fff
    classDef done fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:2px
    classDef pr fill:#2196F3,stroke:#1565C0,color:#fff
    classDef merge fill:#e94560,stroke:#b71c1c,color:#fff
    classDef detect fill:#FF9800,stroke:#E65100,color:#fff
    classDef sync fill:#00BCD4,stroke:#00838F,color:#fff
    classDef healthy fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
```

---

## Overview

This guide is organized into focused topics covering automation fundamentals, major configuration management and provisioning tools, infrastructure CI/CD, advanced operating patterns, and monitoring-driven remediation on Linux.

## Learning Path

```mermaid
graph LR
    N1["01 Automation Fundamentals"]
    N2["02 Ansible"]
    N3["03 Terraform for Linux"]
    N4["04 Puppet"]
    N5["05 Chef"]
    N6["06 SaltStack"]
    N7["07 Cloud-Init"]
    N8["08 Packer"]
    N9["09 CI/CD for Infrastructure"]
    N10["10 Advanced Patterns"]
    N11["11 Monitoring Automation"]
    N1 --> N2
    N2 --> N3
    N3 --> N4
    N4 --> N5
    N5 --> N6
    N6 --> N7
    N7 --> N8
    N8 --> N9
    N9 --> N10
    N10 --> N11
```

## Table of Contents

- [Automation Fundamentals](01-fundamentals.md)
- [Ansible](02-ansible.md)
- [Ansible Deep Dive](12-ansible-deep-dive.md)
- [Terraform for Linux](03-terraform.md)
- [Puppet](04-puppet.md)
- [Chef](05-chef.md)
- [SaltStack](06-saltstack.md)
- [Cloud-Init](07-cloud-init.md)
- [Packer](08-packer.md)
- [CI/CD for Infrastructure](09-cicd-infrastructure.md)
- [Advanced Patterns](10-advanced-patterns.md)
- [Monitoring Automation](11-monitoring-automation.md)
