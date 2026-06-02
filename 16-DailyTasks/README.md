# Linux Day-to-Day Tasks Guide


---

## 🎬 Daily SysAdmin Routine — Animated Workflow

```mermaid
graph LR
    subgraph MORNING["☀️ Morning Checks"]
        direction TB
        M1["☕ Start"]:::start_d --> M2["📊 Check disk: df -h"]:::check
        M2 --> M3["🧠 Check memory: free -h"]:::check
        M3 --> M4["🔲 Check CPU: uptime"]:::check
        M4 --> M5["⚙️ Check services: systemctl"]:::check
        M5 --> M6["📋 Check logs: journalctl"]:::check
        M6 --> M7["✅ Systems OK"]:::ok_d
    end

    subgraph DEPLOY_FLOW["🚀 Deployment Checklist"]
        direction LR
        DF1["📦 Pull latest"]:::deploy --> DF2["🧪 Run tests"]:::deploy
        DF2 --> DF3["💾 Backup current"]:::deploy
        DF3 --> DF4["🚀 Deploy new"]:::deploy
        DF4 --> DF5{"✅ Health check?"}:::health
        DF5 -->|Pass| DF6["🟢 Done"]:::ok_d
        DF5 -->|Fail| DF7["🔄 Rollback"]:::rollback_d
    end

    subgraph BACKUP_DAILY["💾 Backup Routine"]
        direction LR
        BD1["📊 Config files"]:::backup --> BD2["🗄️ Databases"]:::backup
        BD2 --> BD3["📁 User data"]:::backup
        BD3 --> BD4["☁️ Off-site copy"]:::backup
        BD4 --> BD5["✅ Verified"]:::ok_d
    end

    classDef start_d fill:#FF9800,stroke:#E65100,color:#fff
    classDef check fill:#2196F3,stroke:#1565C0,color:#fff
    classDef ok_d fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef deploy fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef health fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:2px
    classDef rollback_d fill:#F44336,stroke:#C62828,color:#fff
    classDef backup fill:#0f3460,stroke:#16213e,color:#fff
```

---

This guide is now split into focused operational runbooks so you can jump directly to a daily task area, validation workflow, or recovery scenario.

## Overview

- Start with server setup and health checks for baseline readiness.
- Use the task guides for logs, users, disks, services, backups, security, networking, deployments, Docker, Kubernetes, cron, and quick performance fixes.
- Use the disaster recovery guide for urgent scenarios plus reusable templates, decision trees, scripts, and operational appendices.

## Learning Path

```mermaid
graph LR
A["Server Setup"] --> B["Health Checks"]
B --> C["Logs"]
C --> D["Users"]
D --> E["Disk"]
E --> F["Services"]
F --> G["Backup and Restore"]
G --> H["Security"]
H --> I["Network"]
I --> J["App Deployment"]
J --> K["Docker"]
K --> L["Kubernetes"]
L --> M["Cron"]
M --> N["Performance"]
N --> O["Disaster Recovery"]
```

## Table of Contents

1. [Server Setup](01-server-setup.md)
2. [Health Checks](02-health-checks.md)
3. [Log Management](03-log-management.md)
4. [User Management](04-user-management.md)
5. [Disk Management](05-disk-management.md)
6. [Service Management](06-service-management.md)
7. [Backup and Restore](07-backup-restore.md)
8. [Security Tasks](08-security-tasks.md)
9. [Network Tasks](09-network-tasks.md)
10. [Application Deployment](10-app-deployment.md)
11. [Docker Day-to-Day](11-docker-daily.md)
12. [Kubernetes Day-to-Day](12-kubernetes-daily.md)
13. [Cron Management](13-cron-management.md)
14. [Performance Fixes](14-performance-fixes.md)
15. [Disaster Recovery](15-disaster-recovery.md)
