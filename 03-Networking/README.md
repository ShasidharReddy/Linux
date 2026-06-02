# Linux Networking Guide

> A production-quality reference from basic concepts to advanced Linux networking operations.
>
> Audience: administrators, DevOps engineers, SREs, platform engineers, students, and anyone building or troubleshooting Linux networks.


---

## 🎬 Network Packet Journey — Animated Workflow

```mermaid
graph TD
    subgraph OSI["📶 OSI Model — Data Flow"]
        direction TB
        O7["7️⃣ Application (HTTP/SSH)"]:::app --> O6["6️⃣ Presentation (SSL/TLS)"]:::pres
        O6 --> O5["5️⃣ Session"]:::sess
        O5 --> O4["4️⃣ Transport (TCP/UDP)"]:::trans
        O4 --> O3["3️⃣ Network (IP)"]:::net
        O3 --> O2["2️⃣ Data Link (Ethernet)"]:::link
        O2 --> O1["1️⃣ Physical (Cable/WiFi)"]:::phys
    end

    subgraph PACKET["📦 Packet Journey"]
        direction LR
        S["🖥️ Source"]:::src --> FW{"🔥 Firewall"}:::fw
        FW -->|Allow| R1["🔀 Router 1"]:::router
        FW -->|Deny| DROP["🚫 Dropped"]:::drop
        R1 --> R2["🔀 Router 2"]:::router
        R2 --> DNS{"🌐 DNS Resolve"}:::dns
        DNS --> D["🖥️ Destination"]:::dest
    end

    subgraph TROUBLESHOOT["🔧 Network Debug Flow"]
        direction LR
        T1["ping"]:::tool --> T2["traceroute"]:::tool
        T2 --> T3["netstat/ss"]:::tool
        T3 --> T4["tcpdump"]:::tool
        T4 --> T5["wireshark"]:::tool
        T5 --> T6["✅ Issue Found"]:::found
    end

    classDef app fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef pres fill:#8BC34A,stroke:#558B2F,color:#fff
    classDef sess fill:#CDDC39,stroke:#9E9D24,color:#333
    classDef trans fill:#FF9800,stroke:#E65100,color:#fff
    classDef net fill:#e94560,stroke:#b71c1c,color:#fff
    classDef link fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef phys fill:#607D8B,stroke:#37474F,color:#fff
    classDef src fill:#2196F3,stroke:#1565C0,color:#fff
    classDef fw fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:3px
    classDef router fill:#0f3460,stroke:#16213e,color:#fff
    classDef drop fill:#F44336,stroke:#C62828,color:#fff
    classDef dns fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef dest fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef tool fill:#0f3460,stroke:#16213e,color:#fff
    classDef found fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
```

---

## Overview

This guide has been split into focused topic files so you can study progressively or jump straight to the area you need. Labs, checklists, practical scenarios, and quick-reference material were redistributed into the most relevant files.

## Learning Path

```mermaid
flowchart TD
    A[01 Fundamentals] --> B[02 Network Hardware]
    A --> C[03 Network Configuration]
    A --> D[04 DNS]
    A --> E[05 Firewall]
    A --> F[06 SSH]
    F --> G[07 Tunneling and VPN]
    C --> H[08 Troubleshooting]
    D --> H
    E --> H
    G --> H
    C --> I[09 Network Services]
    B --> J[10 Bonding and VLANs]
    I --> K[11 Advanced Networking]
    G --> K
    I --> L[12 Load Balancing]
```

## Table of Contents

1. [01 Fundamentals](./01-fundamentals.md) — OSI model, TCP/IP, IP addressing, subnetting, CIDR, ports, protocols, and core concepts.
2. [02 Network Hardware](./02-network-hardware.md) — switches, routers, hubs, firewalls, load balancers, and infrastructure troubleshooting.
3. [03 Network Configuration](./03-network-configuration.md) — `ip addr`, `nmcli`, Netplan, routing, DHCP, static addressing, and persistence methods.
4. [04 DNS](./04-dns.md) — resolver behavior, `resolv.conf`, `dig`, `nslookup`, BIND, DNS scenarios, and DNS checklists.
5. [05 Firewall](./05-firewall.md) — `iptables`, `nftables`, `firewalld`, `ufw`, NAT, firewall operations, and safety checklists.
6. [06 SSH](./06-ssh.md) — `ssh`, `sshd_config`, keys, bastions, `ProxyJump`, file transfer, and SSH hardening.
7. [07 Tunneling and VPN](./07-tunneling-and-vpn.md) — SSH tunnels, TUN/TAP, GRE/VXLAN concepts, OpenVPN, WireGuard, and tunnel scenarios.
8. [08 Network Troubleshooting](./08-network-troubleshooting.md) — `ping`, `traceroute`, `tcpdump`, `nmap`, `ss`, incident workflows, and operator notes.
9. [09 Network Services](./09-network-services.md) — HTTP, FTP, NFS, Samba, DHCP, reverse proxies, and service deployment checklists.
10. [10 Bonding and VLANs](./10-bonding-and-vlans.md) — bond modes, LACP, VLAN tagging, bridges, and virtualization connectivity.
11. [11 Advanced Networking](./11-advanced-networking.md) — namespaces, veth, NAT, forwarding, policy routing, traffic shaping, and cloud/container notes.
12. [12 Load Balancing](./12-load-balancing.md) — HAProxy, Nginx load balancing, Keepalived, health checks, and HA patterns.

## What This Guide Covers

- Network models and addressing
- Linux interface and route configuration
- DNS resolver behavior and server setup
- Firewall technologies and packet flow
- SSH usage and secure remote access
- Troubleshooting tools and structured workflows
- Common network services
- Bonding, VLANs, and bridging
- Namespaces, NAT, forwarding, and traffic shaping
- VPNs with OpenVPN and WireGuard
- Load balancing with HAProxy, Nginx, and Keepalived
