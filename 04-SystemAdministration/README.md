# Linux System Administration Guide


---

## 🎬 SysAdmin Daily Workflow — Animated

```mermaid
graph TD
    subgraph SERVICE["🔄 Service Lifecycle"]
        direction LR
        S1["📝 Unit File"]:::config --> S2["systemctl enable"]:::enable
        S2 --> S3["systemctl start"]:::start
        S3 --> S4{"Running?"}:::check
        S4 -->|Yes| S5["✅ Active"]:::ok
        S4 -->|No| S6["📋 journalctl -u"]:::debug
        S6 --> S3
    end

    subgraph STORAGE["💾 Storage Management"]
        direction TB
        D1["🔲 Physical Disk"]:::disk --> D2["📦 Partition (fdisk)"]:::part
        D2 --> D3["🧱 Physical Volume (pvcreate)"]:::pv
        D3 --> D4["📚 Volume Group (vgcreate)"]:::vg
        D4 --> D5["📁 Logical Volume (lvcreate)"]:::lv
        D5 --> D6["🗂️ Filesystem (mkfs)"]:::fs
        D6 --> D7["📌 Mount Point"]:::mount
    end

    subgraph USERS["👥 User Management"]
        direction LR
        U1["useradd"]:::cmd --> U2["passwd"]:::cmd
        U2 --> U3["usermod -aG"]:::cmd
        U3 --> U4["visudo"]:::cmd
        U4 --> U5["✅ User Ready"]:::ok
    end

    subgraph BACKUP["💾 Backup Strategy"]
        direction LR
        BK1["📊 Full Backup"]:::full --> BK2["📈 Incremental"]:::inc
        BK2 --> BK3["📈 Incremental"]:::inc
        BK3 --> BK4["📈 Incremental"]:::inc
        BK4 --> BK5["📊 Full Backup"]:::full
    end

    classDef config fill:#2196F3,stroke:#1565C0,color:#fff
    classDef enable fill:#FF9800,stroke:#E65100,color:#fff
    classDef start fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef check fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:2px
    classDef ok fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef debug fill:#F44336,stroke:#C62828,color:#fff
    classDef disk fill:#607D8B,stroke:#37474F,color:#fff
    classDef part fill:#795548,stroke:#4E342E,color:#fff
    classDef pv fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef vg fill:#e94560,stroke:#b71c1c,color:#fff
    classDef lv fill:#2196F3,stroke:#1565C0,color:#fff
    classDef fs fill:#00BCD4,stroke:#00838F,color:#fff
    classDef mount fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef cmd fill:#0f3460,stroke:#16213e,color:#fff
    classDef full fill:#e94560,stroke:#b71c1c,color:#fff,stroke-width:2px
    classDef inc fill:#FF9800,stroke:#E65100,color:#fff
```

---

This guide has been split into smaller, topic-focused files. Reference material from the original monolithic guide has been merged into the relevant topic files.

## Table of Contents

1. [Package Management](./01-package-management.md)
2. [Service Management with systemd](./02-systemd.md)
3. [Process Management](./03-process-management.md)
4. [Disk and Storage Management](./04-disk-storage.md)
5. [Logical Volume Management (LVM)](./05-lvm.md)
6. [Software RAID with mdadm](./06-raid.md)
7. [Filesystems](./07-filesystems.md)
8. [Log Management](./08-log-management.md)
9. [Scheduled Tasks](./09-scheduled-tasks.md)
10. [System Monitoring](./10-monitoring.md)
11. [Backup and Recovery](./11-backup-recovery.md)
12. [Kernel Management](./12-kernel-management.md)
13. [NFS Setup](./13-nfs-setup.md)
14. [DNS Server (BIND9)](./14-dns-server.md)
15. [System Hardening](./15-system-hardening.md)
16. [Cloud Migration](./16-cloud-migration.md)
17. [DHCP Server](./16-dhcp-server.md)
18. [Patching and Vulnerabilities](./17-patching-and-vulnerabilities.md)
19. [Time Synchronization (NTP/Chrony)](./17-time-synchronization.md)

## Notes

- Bootloader and GRUB content is included in [Kernel Management](./12-kernel-management.md).
- The current source guide also contained DHCP and Chrony/NTP sections, so they were preserved as additional topic files.
