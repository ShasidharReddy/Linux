# Question 1: Hypervisor Layer — Bare-Metal Servers

This chapter answers the first production question: **how do you turn raw servers into a safe, repeatable virtualization platform?**

The recommendation here is **Proxmox VE on bare metal**, backed by Linux KVM, with a **3-node minimum cluster**, ZFS mirror for the OS, dedicated management and storage networks, and fencing through IPMI.

## Why the hypervisor layer comes first after network

Physical servers do not become a platform just because Linux boots. The hypervisor layer is the control boundary for:

- VM lifecycle
- CPU and memory allocation
- snapshots and templates
- live migration and HA restart
- shared storage attachment
- cluster membership and quorum

If the hypervisor design is weak, every higher layer inherits the weakness.

## Hypervisor Choice Decision

| Hypervisor | Pros | Cons | Best For |
|-----------|------|------|----------|
| KVM + libvirt | Free, native Linux, flexible, scriptable | CLI-heavy, more DIY clustering | Cost-conscious teams, Linux-first shops |
| Proxmox VE | KVM-based, web UI, built-in clustering, HA, backup integration | Enterprise repo/support costs, opinionated stack | SMBs, internal platforms, pragmatic on-prem production |
| VMware ESXi | Mature ecosystem, vMotion, DRS, enterprise support | License cost, proprietary, vendor lock-in | Enterprises with budget and existing VMware skills |
| oVirt/RHV | KVM-based, enterprise-style features, open-source roots | More moving parts, smaller mindshare today | RHEL-heavy organizations |

## Recommendation

**Choose Proxmox VE.**

### Why Proxmox fits this scenario

1. **Free and KVM-based**: no need to pay for a proprietary hypervisor to get started.
2. **Balanced simplicity**: less DIY than raw libvirt, less licensing burden than VMware.
3. **Built-in clustering**: corosync membership, HA groups, and migration are already integrated.
4. **Operationally friendly**: web UI for routine admin, CLI and API for automation.
5. **Storage flexibility**: local ZFS, NFS, iSCSI, Ceph, and LVM all supported.

### When to choose something else

- Choose **raw KVM + libvirt** if the organization is deeply Linux-automation centric and wants minimal platform abstraction.
- Choose **VMware** if vSphere is already standard and the budget is approved.
- Choose **oVirt/RHV** if the environment is tightly aligned with Red Hat tooling and support.

## Pre-requisites Before First Node

### Physical

- rack-mounted servers with cable labels on both ends
- dual power feeds where available
- BIOS and BMC firmware updated to approved baseline
- IPMI/iLO/iDRAC configured on OOB network
- boot media and remote console tested

### Network

- management VLAN configured on the switch
- switch trunk design agreed for hypervisor NICs
- OOB management VLAN available for BMC/IPMI
- storage VLAN available before NFS/iSCSI onboarding

### Core services

- forward and reverse DNS entries for each node
- NTP source reachable from management network
- IPAM sheet reserved for host, storage, migration, and VM bridge ranges
- password vault entry for BMC, switch, and initial hypervisor credentials

### Naming convention example

| Node | FQDN | MGMT IP | OOB IP |
|------|------|---------|--------|
| pve01 | pve01.infra.example.com | 10.10.10.11 | 10.10.99.11 |
| pve02 | pve02.infra.example.com | 10.10.10.12 | 10.10.99.12 |
| pve03 | pve03.infra.example.com | 10.10.10.13 | 10.10.99.13 |

## Node Count & Clustering

### Minimum node count

Use **3 nodes minimum**.

Why:

- corosync quorum works cleanly with an odd count
- split-brain risk is lower than with 2 nodes
- one node can fail and the cluster still keeps quorum
- HA placement has somewhere to move workloads

### Sizing guidance per node

| Resource | Suggested Production Starting Point | Why |
|----------|-------------------------------------|-----|
| CPU | 2x CPU sockets or modern 24-56 core platform | VM density and headroom |
| RAM | 256 GB to 512 GB | consolidation without excessive ballooning |
| Local OS disks | 2x NVMe in ZFS mirror | resilient hypervisor boot volume |
| Local cache / scratch | optional extra NVMe | ISO, cache, temporary local workloads |
| NICs | 2x 10GbE minimum, ideally 4 ports total | separation of VM/mgmt and storage |
| BMC | 1 OOB port | recovery path |

## 3-Node Proxmox Cluster Topology

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    SW1[Switch A] --- SW2[Switch B]
    FW[Firewall MGMT Routing] --- SW1
    FW --- SW2
    OOB[OOB Switch VLAN 99] --- BMC1[pve01 IPMI]
    OOB --- BMC2[pve02 IPMI]
    OOB --- BMC3[pve03 IPMI]
    P1[pve01 PVE + KVM] --- SW1
    P1 --- SW2
    P2[pve02 PVE + KVM] --- SW1
    P2 --- SW2
    P3[pve03 PVE + KVM] --- SW1
    P3 --- SW2
    ST[Shared Storage NFS iSCSI] --- SW1
    ST --- SW2
~~~

## Cluster Quorum and HA Logic

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[3 Nodes Online] --> B[Corosync Quorum Healthy]
    B --> C[HA Manager Can Place VMs]
    C --> D[One Node Fails]
    D --> E[2 Nodes Remain]
    E --> F[Quorum Still Healthy]
    F --> G[Restart or Migrate Protected Workloads]
~~~

## Installation Workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Update BIOS and BMC] --> B[Boot Proxmox ISO]
    B --> C[Install to ZFS Mirror]
    C --> D[Configure MGMT IP and DNS]
    D --> E[Patch First Node]
    E --> F[pvecm create cluster]
    F --> G[Install Remaining Nodes]
    G --> H[pvecm add on Node 2 and 3]
    H --> I[Attach Shared Storage]
    I --> J[Create HA Groups and Fencing]
    J --> K[Run Verification Checklist]
~~~

## Step-by-Step Installation

### 1. BIOS and firmware settings

Set these before the OS installer:

- Intel VT-x / AMD-V enabled
- VT-d / IOMMU enabled if PCI passthrough may be required
- SR-IOV enabled if supported and needed
- boot mode set to UEFI
- hyper-threading according to security policy
- redundant power policy enabled
- NUMA exposed if large VMs will be used
- BMC/IPMI NIC placed in OOB VLAN 99

### 2. Install Proxmox VE

Download the ISO, boot via remote media or physical console, and install to a mirrored local volume.

Example choices during install:

- filesystem: **ZFS RAID1**
- hostname: `pve01.infra.example.com`
- management interface: first bonded path or single temporary port
- DNS: internal resolvers in MGMT VLAN

### 3. Example network config after install

`/etc/network/interfaces`

~~~bash
auto lo
iface lo inet loopback

auto eno1
iface eno1 inet manual

auto eno2
iface eno2 inet manual

auto bond0
iface bond0 inet manual
    bond-slaves eno1 eno2
    bond-miimon 100
    bond-mode 802.3ad
    bond-xmit-hash-policy layer3+4

auto vmbr0
iface vmbr0 inet static
    address 10.10.10.11/24
    gateway 10.10.10.1
    bridge-ports bond0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 10 20 30 40 50 60
~~~

### 4. Patch and validate the first node

~~~bash
apt update
apt full-upgrade -y
hostnamectl
ip -br addr
chronyc sources -v
nslookup pve01.infra.example.com
~~~

### 5. Create the cluster on node 1

~~~bash
pvecm create am-prod-cluster
pvecm status
~~~

### 6. Join nodes 2 and 3

On `pve02` and `pve03`:

~~~bash
apt update && apt full-upgrade -y
pvecm add 10.10.10.11
pvecm status
~~~

### 7. Validate membership from any node

~~~bash
pvecm nodes
corosync-cfgtool -s
systemctl status corosync pve-cluster
~~~

## HA and fencing design

HA without fencing is not safe enough. If a node is only *suspected* down but is still alive and writing, split-brain can corrupt data or create duplicate workload ownership.

### Fencing recommendation

Use **IPMI-based fencing** via the BMC for each node.

Conceptual design:

- the cluster decides a node is failed
- HA manager triggers a fence action through BMC
- power state is confirmed
- protected workloads restart elsewhere

### HA failover sequence

~~~mermaid
%%{init:{"theme":"neutral"}}%%
sequenceDiagram
    participant Monitor as Cluster Monitor
    participant Failed as Failed Node
    participant BMC as IPMI BMC
    participant HA as HA Manager
    participant Peer as Surviving Node
    Monitor->>Failed: heartbeat timeout detected
    Monitor->>BMC: fence failed node
    BMC->>Failed: power off
    Monitor->>HA: fence confirmed
    HA->>Peer: start protected VM
    Peer->>HA: VM running
~~~

## Design decisions during install

### Filesystem choice: ZFS mirror vs ext4

| Choice | Why choose it | Trade-off |
|--------|----------------|-----------|
| ZFS mirror | checksums, snapshots, self-healing on mirror, operational visibility | more RAM usage, more concepts to learn |
| ext4 on mdraid | familiar, lightweight, simple | fewer built-in storage features |

**Recommendation:** use **ZFS mirror for the hypervisor OS**. It gives safer local storage behavior and easier rollback for host-level changes.

### Linux bridge vs Open vSwitch

| Choice | Pros | Cons |
|--------|------|------|
| Linux bridge | simple, native, well understood, enough for most Proxmox clusters | fewer advanced virtual networking features |
| OVS | richer virtual networking features, good in advanced overlays | more complexity for limited near-term gain |

**Recommendation:** start with **Linux bridge** unless you already need advanced OVS-specific features.

### Local vs shared storage

| Storage Type | Best Use | Risk |
|--------------|----------|------|
| Local | ISOs, backups, scratch workloads, non-HA test VMs | live migration harder, host-local failure impact |
| Shared | production VMs, templates, migration, HA | requires network and storage design discipline |

## Post-install commands worth standardizing

### Proxmox repositories

~~~bash
sed -i 's/^deb/# deb/' /etc/apt/sources.list.d/pve-enterprise.list
cat >/etc/apt/sources.list.d/pve-no-subscription.list <<'EOR'
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
EOR
apt update
~~~

### Time and DNS

~~~bash
apt install -y chrony dnsutils
chronyc tracking
resolvectl status || cat /etc/resolv.conf
~~~

### Basic packages

~~~bash
apt install -y qemu-guest-agent jq vim tmux curl wget prometheus-node-exporter
systemctl enable --now qemu-guest-agent prometheus-node-exporter
~~~

## Example HA group and VM policy

~~~bash
ha-manager groupadd prod-ha --nodes pve01:1,pve02:1,pve03:1
ha-manager add vm:200 --group prod-ha --state started
ha-manager status
~~~

## Example IPMI fencing checks

~~~bash
apt install -y ipmitool fence-agents
ipmitool -I lanplus -H 10.10.99.11 -U admin -a power status
ipmitool -I lanplus -H 10.10.99.12 -U admin -a chassis identify 10
~~~

## Verification Checklist Before Moving to the Next Layer

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Cluster quorum | `pvecm status` | quorum present, expected votes |
| Node membership | `pvecm nodes` | all nodes online |
| Corosync state | `corosync-cfgtool -s` | ring healthy |
| Bridges | `bridge vlan show` | expected VLANs visible |
| DNS | `getent hosts pve01.infra.example.com` | forward lookup works |
| Reverse DNS | `dig -x 10.10.10.11 +short` | PTR record correct |
| NTP | `chronyc sources` | synced source present |
| OOB console | BMC web or console login | working remote recovery path |
| Metrics | `curl -s localhost:9100/metrics | head` | node exporter data |

## Failure tests to run early

1. **Single NIC pull**: verify bond survives.
2. **Single switch path loss**: verify host remains reachable.
3. **Single node reboot**: verify quorum remains healthy.
4. **HA restart test**: move a non-critical test VM or let it restart on another host.
5. **Fence simulation**: validate BMC credentials and power action behavior during a maintenance window.

## Advanced: Ansible Playbook for Proxmox Post-Install Setup

~~~yaml
---
- name: Post-install Proxmox baseline
  hosts: proxmox
  become: true
  vars:
    common_packages:
      - chrony
      - vim
      - tmux
      - jq
      - curl
      - wget
      - qemu-guest-agent
      - prometheus-node-exporter
  tasks:
    - name: Disable enterprise repository when not subscribed
      lineinfile:
        path: /etc/apt/sources.list.d/pve-enterprise.list
        regexp: '^deb '
        line: '# deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise'
      ignore_errors: true

    - name: Enable no-subscription repository
      copy:
        dest: /etc/apt/sources.list.d/pve-no-subscription.list
        content: |
          deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

    - name: Install baseline packages
      apt:
        update_cache: true
        name: "{{ common_packages }}"
        state: present

    - name: Ensure services are enabled
      service:
        name: "{{ item }}"
        enabled: true
        state: started
      loop:
        - chrony
        - prometheus-node-exporter

    - name: Harden SSH root login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin prohibit-password'
      notify: restart ssh

    - name: Set sysctl values
      copy:
        dest: /etc/sysctl.d/99-am-baseline.conf
        content: |
          net.ipv4.conf.all.rp_filter=1
          net.ipv4.conf.default.rp_filter=1
          vm.swappiness=10
      notify: reload sysctl

  handlers:
    - name: restart ssh
      service:
        name: ssh
        state: restarted

    - name: reload sysctl
      command: sysctl --system
~~~

## Operational notes

- put each node into maintenance mode before BIOS or firmware work
- keep one cluster node empty enough to absorb a failed peer
- document PCI passthrough dependencies before upgrades
- never combine major cluster, network, and storage changes in the same window

## Common mistakes

- building only 2 nodes and calling it HA
- skipping OOB management until after a lockout
- using a flat bridge for all traffic without VLAN planning
- putting storage traffic on the same busy path as tenant workloads
- enabling HA before verifying fencing
- overcommitting memory heavily before monitoring ballooning and swap pressure

## Exit criteria for this chapter

You are ready for [02-network-design.md](./02-network-design.md) and [04-shared-storage.md](./04-shared-storage.md) when:

- all 3 nodes are in cluster and healthy
- management, storage, and VM bridge design is documented
- BMC works for every host
- baseline packages and monitoring agents are installed
- failover behavior has been tested with at least one non-critical VM


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### Architecture decision record summary

| ADR | Decision | Reason | Alternative Rejected |
|-----|----------|--------|----------------------|
| ADR-01 | Proxmox VE over VMware | lower licensing cost, API + HA included | VMware cost and licensing complexity |
| ADR-02 | 3-node minimum cluster | quorum and maintenance safety | 2-node cluster risks split-brain/design compromises |
| ADR-03 | Local mirrored OS disks + shared VM storage | simple rebuilds and movable workloads | booting hypervisors from SAN adds coupling |
| ADR-04 | IPMI fencing mandatory | clean HA decisions under host failure | no fencing leads to unsafe restart behavior |

### Integration points

- Depends on [02-network-design.md](./02-network-design.md) for VLAN trunks, bonding, and OOB reachability.
- Depends on [04-shared-storage.md](./04-shared-storage.md) for HA-ready NFS/iSCSI.
- Feeds [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) by exposing templates, storage, and API automation.

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

- Enable CPU pinning and NUMA-aware placement only for latency-sensitive VMs; keep default scheduling for general workloads.
- Use BIOS profiles for virtualization plus deterministic power settings; disable unused onboard devices.
- For DR, keep a warm spare cluster or at least replicated backups plus tested restore timings.
- Compliance impact: SOC 2 needs access reviews and log retention, PCI requires tighter segmentation and MFA, HIPAA needs audit trail and encryption controls.
- Alert when cluster free RAM <20%, CPU ready/steal rises, quorum changes, or a node drifts from the approved firmware baseline.
- Auto-remediation can evacuate non-critical VMs from a degraded node, but only after fencing and storage paths are trusted.

## Building & Deployment Runbook

1. Rack nodes, connect redundant power, and cable data plus BMC ports.
2. Update firmware from the approved bundle and record versions:
   ~~~bash
   ipmitool mc info
   pveversion -v
   ~~~
3. Install Proxmox on each node with mirrored OS disks and static management IPs.
4. Create the cluster on the first node and join the rest:
   ~~~bash
   pvecm create am-pve-cluster
   pvecm add 10.10.10.11
   pvecm status
   ~~~
5. Configure bridges, NTP, DNS, and local ZFS; verify `pvesh get /cluster/status` returns all nodes online.
6. Add shared storage and confirm `pvesm status` is identical on every node.
7. Configure HA groups only after BMC fencing checks pass.
8. Validation gate: live-migrate a non-critical VM, power-cycle one test host via BMC, and confirm the cluster stays healthy.

### Common mistakes to avoid during build

- Mixing server generations in the first cluster without validating live migration CPU compatibility.
- Buying too little RAM and then relying on aggressive ballooning as a design strategy.
- Skipping warranty SLAs for nodes that host many tenants.
- Forgetting spare optics, rails, or SSDs and stretching the go-live window.


## Cross-references

- Network details: [02-network-design.md](./02-network-design.md)
- Storage onboarding: [04-shared-storage.md](./04-shared-storage.md)
- VM lifecycle after the cluster exists: [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md)
- Related repo background: [../Virtual-Setup/01-virtualization-fundamentals.md](../Virtual-Setup/01-virtualization-fundamentals.md)
- Related physical patterns: [../Physical-Setup/06-advanced-production-setup.md](../Physical-Setup/06-advanced-production-setup.md)
