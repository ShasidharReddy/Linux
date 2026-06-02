# Linux Security Guide


---

## 🎬 Security Hardening — Animated Workflow

```mermaid
graph TD
    subgraph HARDENING["🔒 System Hardening Flow"]
        direction TB
        H1["🔍 Audit Current State"]:::audit --> H2["🔐 SSH Hardening"]:::secure
        H2 --> H3["🔥 Firewall Rules"]:::secure
        H3 --> H4["🛡️ SELinux/AppArmor"]:::secure
        H4 --> H5["🔑 Key-Based Auth"]:::secure
        H5 --> H6["📋 Audit Logging"]:::secure
        H6 --> H7["✅ Hardened"]:::done
    end

    subgraph INCIDENT["🚨 Incident Response"]
        direction LR
        I1["🔔 Alert"]:::alert --> I2["🔍 Identify"]:::identify
        I2 --> I3["🛑 Contain"]:::contain
        I3 --> I4["🔧 Eradicate"]:::fix
        I4 --> I5["🔄 Recover"]:::recover
        I5 --> I6["📝 Lessons Learned"]:::learn
    end

    subgraph ENCRYPTION["🔐 Encryption Layers"]
        direction TB
        E1["💾 LUKS Disk Encryption"]:::encrypt
        E2["🔒 TLS/SSL Transport"]:::encrypt
        E3["🔑 GPG File Encryption"]:::encrypt
        E4["🗝️ SSH Key Pairs"]:::encrypt
        E1 --> E2 --> E3 --> E4
    end

    classDef audit fill:#FF9800,stroke:#E65100,color:#fff
    classDef secure fill:#2196F3,stroke:#1565C0,color:#fff
    classDef done fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef alert fill:#F44336,stroke:#C62828,color:#fff,stroke-width:3px
    classDef identify fill:#FF9800,stroke:#E65100,color:#fff
    classDef contain fill:#e94560,stroke:#b71c1c,color:#fff
    classDef fix fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef recover fill:#2196F3,stroke:#1565C0,color:#fff
    classDef learn fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef encrypt fill:#1a1a2e,stroke:#e94560,color:#fff,stroke-width:2px
```

---

A production-focused Linux security guide covering foundational controls, hardening, auditing, incident response, and advanced defensive techniques.

> Audience: Linux administrators, DevOps engineers, security engineers, SREs, students, and operators who need practical hardening guidance from basic to advanced.

> Scope: This guide is distribution-aware but intentionally generic. Validate package names, service names, and paths on your platform before rollout.

## Table of Contents

1. [Security Fundamentals](./01-security-fundamentals.md)
2. [User Security](./02-user-security.md)
3. [Filesystem Security](./03-filesystem-security.md)
4. [SELinux](./04-selinux.md)
5. [AppArmor](./05-apparmor.md)
6. [Firewall Security](./06-firewall-security.md)
7. [SSH Hardening](./07-ssh-hardening.md)
8. [Encryption](./08-encryption.md)
9. [Auditing](./09-auditing.md)
10. [Intrusion Detection](./10-intrusion-detection.md)
11. [Network Security](./11-network-security.md)
12. [Container Security](./12-container-security.md)
13. [Incident Response](./13-incident-response.md)

## Appendix Material

- Appendix A provides concise operational checklists you can adapt for build reviews, quarterly hardening audits, and pre-production approvals.
- Use this command reference as a quick lookup during hardening reviews and incident response.
- Use these questions for self-study, team training, or internal review sessions.
- Checklist, command reference, and review-question appendix material has been folded into the relevant chapter.
