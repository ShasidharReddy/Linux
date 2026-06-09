# 15. Architecture and HLD Reference Diagrams

- **Purpose:** Provide reusable high level Mermaid diagrams for common bare metal infrastructure designs.
- **Style:** Production-oriented, concise bullets, diagrams, and planning notes.
- **Audience:** Architects, platform engineers, SREs, operations teams, and documentation owners.
- **Use this guide when:** Preparing HLDs, design reviews, migration plans, and executive summaries.
> **Disclaimer:** Third-party logos and screenshots are used for educational purposes only.

- Use these diagrams as reference templates, then adapt labels, rack IDs, VLANs, and host counts to your environment.
- Keep naming consistent with CMDB, IPAM, and change records.
- Each diagram is intentionally generic so it renders cleanly in Markdown and Mermaid.

### 15.1 Complete Datacenter Rack Layout

- Shows a single 42U rack with PDUs, patch panels, switches, and servers mapped by position.
- Use it for rack elevation planning, capacity checks, and documenting management versus production cabling.

```mermaid
flowchart TD
    U42[U42 Patch panel]
    U41[U41 Patch panel]
    U40[U40 Spine uplink panel]
    U39[U39 ToR switch A]
    U38[U38 ToR switch B]
    U37[U37 Management switch]
    U36[U36 KVM and console]
    U35[U35 Server 01]
    U34[U34 Server 02]
    U33[U33 Server 03]
    U32[U32 Server 04]
    U31[U31 Storage node 01]
    U30[U30 Storage node 02]
    U29[U29 GPU node 01]
    U28[U28 GPU node 02]
    U06[U06 UPS monitor]
    U05[U05 PDU A]
    U04[U04 PDU B]
    U39 --> U35
    U39 --> U31
    U38 --> U34
    U38 --> U32
    U37 --> U35
    U37 --> U34
    U37 --> U31
    U37 --> U29
    U05 --> U35
    U04 --> U35
```

### 15.2 Network Architecture Spine Leaf Topology

- Shows the modern leaf and spine model with routed underlay and server attachment at the leaf layer.
- Use it for new datacenter builds, rack expansion plans, and bandwidth discussions.

```mermaid
flowchart TD
    Internet[Internet edge] --> FW[Firewall]
    FW --> Core[Core or border routers]
    Core --> Spine1[Spine 1]
    Core --> Spine2[Spine 2]
    Spine1 --> Leaf1[Leaf 1]
    Spine1 --> Leaf2[Leaf 2]
    Spine1 --> Leaf3[Leaf 3]
    Spine1 --> Leaf4[Leaf 4]
    Spine2 --> Leaf1
    Spine2 --> Leaf2
    Spine2 --> Leaf3
    Spine2 --> Leaf4
    Leaf1 --> Rack1[Servers rack 1]
    Leaf2 --> Rack2[Servers rack 2]
    Leaf3 --> Rack3[Servers rack 3]
    Leaf4 --> Rack4[Servers rack 4]
```

### 15.3 Three Tier Application Architecture

- Shows a classic application stack with HA load balancers, app nodes, database replication, cache, and storage.
- Use it for internal business apps, web platforms, and migration planning from legacy estates.

```mermaid
flowchart LR
    Users[Users] --> VIP[VIP]
    VIP --> LB1[HAProxy 1]
    VIP --> LB2[HAProxy 2]
    LB1 --> APP1[App 1]
    LB1 --> APP2[App 2]
    LB2 --> APP3[App 3]
    APP1 --> Cache[Redis Sentinel]
    APP2 --> Cache
    APP3 --> Cache
    APP1 --> DB1[DB primary]
    APP2 --> DB1
    APP3 --> DB1
    DB1 --> DB2[DB replica]
    APP1 --> NFS[NFS or SAN]
    Monitor[Monitoring overlay] --> LB1
    Monitor --> APP1
    Monitor --> DB1
```

### 15.4 Kubernetes Bare Metal Cluster

- Shows a highly available Kubernetes platform on physical nodes with ingress, storage, and service advertisement.
- Use it when documenting platform services and control plane dependencies.

```mermaid
flowchart TD
    Users[Users] --> IngressVIP[Ingress VIP]
    Admin[Admins] --> APIVIP[API VIP]
    APIVIP --> HAP[HAProxy]
    HAP --> CP1[Control plane 1]
    HAP --> CP2[Control plane 2]
    HAP --> CP3[Control plane 3]
    CP1 --> Workers[Worker pool]
    CP2 --> Workers
    CP3 --> Workers
    Workers --> Ingress[Ingress controller]
    Workers --> MetalLB[MetalLB]
    Workers --> Store[Longhorn or Ceph]
    CP1 --> ETCD[etcd]
    CP2 --> ETCD
    CP3 --> ETCD
```

### 15.5 High Availability Active Passive Failover

- Shows two nodes managed by Pacemaker and Corosync with DRBD replication and a floating VIP.
- Use it for stateful legacy applications or simple highly available services.

```mermaid
flowchart LR
    Clients[Clients] --> VIP[Service VIP]
    VIP --> N1[Node 1 active]
    VIP --> N2[Node 2 standby]
    N1 --- Cluster[Pacemaker and Corosync]
    N2 --- Cluster
    N1 --> DRBD1[DRBD primary]
    N2 --> DRBD2[DRBD secondary]
    DRBD1 --> Sync[Replication link]
    Sync --> DRBD2
    Fence[IPMI fence] --> N1
    Fence --> N2
```

### 15.6 High Availability Active Active with Load Balancing

- Shows multiple active application nodes behind a shared load balancer with multi node storage and Galera.
- Use it for horizontally scalable web and API platforms.

```mermaid
flowchart TD
    Users[Users] --> LB[Load balancer farm]
    LB --> APP1[App node 1]
    LB --> APP2[App node 2]
    LB --> APP3[App node 3]
    APP1 --> Session[Session persistence]
    APP2 --> Session
    APP3 --> Session
    APP1 --> Galera[Galera cluster]
    APP2 --> Galera
    APP3 --> Galera
    APP1 --> Shared[GlusterFS or Ceph]
    APP2 --> Shared
    APP3 --> Shared
```

### 15.7 Disaster Recovery Flow

- Shows the movement of data and decision making from the primary site to the DR site.
- Use it to explain RTO, RPO, and failover activation points to stakeholders.

```mermaid
flowchart LR
    Primary[Primary site] --> Backup[Backup pipeline]
    Backup --> Vault[Backup repository]
    Vault --> DR[DR site]
    Primary --> Decision[Failover decision]
    Decision --> RPO[RPO target]
    Decision --> RTO[RTO target]
    Decision --> DNS[DNS failover]
    DNS --> Users[Users redirected]
```

### 15.8 PXE Boot Provisioning Flow

- Shows the end to end bare metal provisioning process from firmware through unattended OS installation.
- Use it for server factory, rack expansion, and rebuild documentation.

```mermaid
flowchart LR
    Boot[BIOS or UEFI] --> DHCP[DHCP]
    DHCP --> TFTP[TFTP]
    TFTP --> PXE[PXE bootloader]
    PXE --> Kickstart[Kickstart or Preseed]
    Kickstart --> Install[OS install]
    Install --> Config[Post install config]
    Cobbler[Cobbler or Foreman] --> DHCP
    Cobbler --> Kickstart
```

### 15.9 Monitoring Stack Architecture

- Shows metrics, logs, and alert routing across hosts, devices, and operator channels.
- Use it when designing observability and on call workflows.

```mermaid
flowchart LR
    NodeExp[Node exporters] --> Prom[Prometheus]
    SNMP[SNMP devices] --> Prom
    AppExp[App exporters] --> Prom
    Prom --> Graf[Grafana]
    Prom --> Alert[Alertmanager]
    Alert --> Slack[Slack]
    Alert --> Email[Email]
    Alert --> Pager[PagerDuty]
    Logs[Filebeat] --> Logstash[Logstash]
    Logstash --> Elastic[Elasticsearch]
    Elastic --> Kibana[Kibana]
```

### 15.10 Security Zones and Network Segmentation

- Shows common network zones and the firewall control points between them.
- Use it for security reviews, remote access planning, and segmentation policy design.

```mermaid
flowchart LR
    Internet[Internet] --> FW[Firewall]
    FW --> DMZ[DMZ]
    FW --> App[Application zone]
    FW --> DB[Database zone]
    FW --> Mgmt[Management zone]
    VPN[VPN users] --> Bastion[Bastion host]
    Bastion --> Mgmt
    DMZ --> App
    App --> DB
```

### 15.11 Backup Architecture

- Shows local, remote, and offsite backup tiers plus retention concepts.
- Use it for backup platform planning and audit evidence.

```mermaid
flowchart TD
    Prod[Production data] --> Local[Local disk backup]
    Local --> Remote[Remote backup server]
    Remote --> Offsite[Offsite tape or S3]
    Local --> Short[Daily retention]
    Remote --> Medium[Weekly retention]
    Offsite --> Long[Monthly and yearly retention]
```

### 15.12 Full Infrastructure HLD

- Shows the end to end executive summary for a typical production bare metal platform.
- Use it in overview decks, architecture records, and environment handoff documents.

```mermaid
flowchart TD
    Internet[Internet] --> Edge[Firewall and edge router]
    Edge --> LB[Load balancer VIP]
    LB --> App[Application tier]
    App --> DB[Database tier]
    App --> Cache[Cache tier]
    App --> Storage[Storage tier]
    Mgmt[Management plane] --> BMC[BMC and IPMI]
    Mgmt --> Ansible[Ansible]
    Mgmt --> Monitor[Monitoring]
    Monitor --> App
    Monitor --> DB
    Network[Network overlay] --> Edge
    Network --> Storage
```

## Usage notes

- Keep Mermaid node labels short and consistent to avoid render issues.
- Split overly dense diagrams into separate views for management, data, and security planes.
- Store the source Markdown in version control and regenerate exports from the same source.
- Pair every HLD with an IP plan, VLAN map, and service inventory.

## Official references

- [Mermaid documentation](https://mermaid.js.org/)
- [Kubernetes architecture concepts](https://kubernetes.io/docs/concepts/architecture/)
- [Red Hat HA documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_high_availability_clusters/index)
- [Prometheus architecture overview](https://prometheus.io/docs/introduction/overview/)
- [Keepalived documentation](https://keepalived.readthedocs.io/)
- [HAProxy documentation](https://www.haproxy.org/)
