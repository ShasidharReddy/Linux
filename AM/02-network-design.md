# Question 2: Network Design

A production platform fails fast when the network is flat, undocumented, or single-homed. This chapter designs the network for **security, HA, automation, observability, and simplicity**.

The north star is:

- isolate traffic by purpose
- make every host dual-homed
- keep storage and management predictable
- automate switch and host configuration after initial bootstrap
- verify with real failure tests, not assumptions

## VLAN Segmentation Strategy

| VLAN ID | Name | Purpose | CIDR | Security Level |
|---------|------|---------|------|---------------|
| 10 | MGMT | Hypervisor management, admin tooling | 10.10.10.0/24 | High |
| 20 | STORAGE | NFS and iSCSI traffic | 10.10.20.0/24 | High |
| 30 | VM-PROD | Production VM east-west and north-south | 10.10.30.0/24 | Medium |
| 40 | VM-DEV | Development and staging VMs | 10.10.40.0/24 | Low |
| 50 | DMZ | Public-facing reverse proxies and edge services | 10.10.50.0/24 | Internet-facing |
| 60 | K8S | Kubernetes node and infra network | 10.10.60.0/24 | Medium |
| 99 | OOB | Out-of-band IPMI/iLO/iDRAC | 10.10.99.0/24 | Highest |

## VLAN Topology and Firewall Zones

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    INET[Internet] --> FW[Hardware Firewall]
    FW --> DMZ[VLAN 50 DMZ]
    FW --> MGMT[VLAN 10 MGMT]
    FW --> PROD[VLAN 30 VM PROD]
    FW --> DEV[VLAN 40 VM DEV]
    FW --> K8S[VLAN 60 K8S]
    FW --> STOR[VLAN 20 STORAGE]
    OOB[VLAN 99 OOB] --> BMC[IPMI iLO iDRAC]
~~~

## Why this segmentation works

### Management stays private

Admin interfaces are in VLAN 10 and never directly exposed to the internet. This makes bastion access, logging, and control-plane monitoring easier to govern.

### Storage stays isolated

Storage is chatty and sensitive to latency. Putting NFS and iSCSI on VLAN 20 reduces contention and shrinks the blast radius of tenant traffic.

### DMZ is thin by design

DMZ should host reverse proxies, load balancers, and tightly controlled edge services, not databases or broad management access.

### OOB remains independent

VLAN 99 is the rescue lane. It should not depend on the in-band production path.

## Complete network topology

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    ISP[ISP or Upstream Router] --> FW[Firewall Appliance]
    FW --> SWA[Switch A]
    FW --> SWB[Switch B]
    SWA --- SWB
    SWA --> P1A[pve01 link A]
    SWB --> P1B[pve01 link B]
    SWA --> P2A[pve02 link A]
    SWB --> P2B[pve02 link B]
    SWA --> P3A[pve03 link A]
    SWB --> P3B[pve03 link B]
    SWA --> STOA[Storage ctrl A]
    SWB --> STOB[Storage ctrl B]
    OOBSW[OOB Switch] --> B1[pve01 IPMI]
    OOBSW --> B2[pve02 IPMI]
    OOBSW --> B3[pve03 IPMI]
~~~

## Host-level redundancy

### Recommendation

Use **bond mode 4 (802.3ad LACP)** for production-facing host uplinks.

### Why LACP

- active-active use of both links
- aggregated bandwidth
- cleaner failover during single cable or single port failure
- integrates well with dual-switch MLAG or stack designs

### Minimum NIC layout per hypervisor

| NIC Set | Usage | Recommendation |
|---------|-------|----------------|
| eno1 + eno2 | MGMT + VM trunks | Bonded LACP |
| eno3 + eno4 | STORAGE | Separate bonded LACP or active/backup depending on storage design |
| BMC | OOB | Dedicated access port on VLAN 99 |

## NIC bonding topology

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    PVE[Hypervisor Host]
    PVE --> BOND0[bond0 LACP MGMT + VM VLAN trunk]
    PVE --> BOND1[bond1 LACP STORAGE VLAN]
    BOND0 --> SWA[Switch A]
    BOND0 --> SWB[Switch B]
    BOND1 --> SWA
    BOND1 --> SWB
~~~

## Example Proxmox host networking

`/etc/network/interfaces`

~~~bash
auto lo
iface lo inet loopback

auto eno1
iface eno1 inet manual

auto eno2
iface eno2 inet manual

auto eno3
iface eno3 inet manual

auto eno4
iface eno4 inet manual

auto bond0
iface bond0 inet manual
    bond-slaves eno1 eno2
    bond-miimon 100
    bond-mode 802.3ad
    bond-xmit-hash-policy layer3+4

auto bond1
iface bond1 inet manual
    bond-slaves eno3 eno4
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
    bridge-vids 10 30 40 50 60

auto vmbr1
iface vmbr1 inet static
    address 10.10.20.11/24
    mtu 9000
    bridge-ports bond1
    bridge-stp off
    bridge-fd 0
~~~

## Switch redundancy model

### Preferred design

Use either:

- a **stacked pair** of switches, or
- an **MLAG pair** if your platform supports it

Each server connects to **both** switches. That way a switch failure does not isolate the host.

### Redundant switch topology

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    SWA[Switch A] --- PEER[Peer Link or Stack Backplane]
    PEER --- SWB[Switch B]
    SWA --> HOST1A[Host port A]
    SWB --> HOST1B[Host port B]
    SWA --> HOST2A[Storage port A]
    SWB --> HOST2B[Storage port B]
    SWA --> FW1[Firewall uplink A]
    SWB --> FW2[Firewall uplink B]
~~~

## Switch configuration concepts

### Trunk ports to hypervisors

- allow VLANs 10,20,30,40,50,60
- use LACP port channel
- disable access VLAN ambiguity
- set MTU 9000 on storage path if platform supports per-interface MTU

### Access ports for OOB

- assign VLAN 99 only
- disable trunking
- port security where appropriate

### Inter-switch links

- trunk required VLANs only
- LACP if supported
- RSTP or vendor equivalent enabled

## Cisco IOS example

~~~bash
vlan 10
 name MGMT
vlan 20
 name STORAGE
vlan 30
 name VM-PROD
vlan 40
 name VM-DEV
vlan 50
 name DMZ
vlan 60
 name K8S
vlan 99
 name OOB

interface Port-channel10
 description PVE01_TRUNK
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
 spanning-tree portfast trunk
 mtu 9216

interface range TenGigabitEthernet1/0/1-2
 description PVE01_MEMBER
 channel-group 10 mode active

interface GigabitEthernet1/0/48
 description PVE01_IPMI
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
~~~

## Cumulus Linux example

~~~bash
net add bond bond10 bond slaves swp1 swp2
net add bond bond10 clag id 10
net add interface bond10 bridge vids 10,20,30,40,50,60
net add interface bond10 bridge pvid 1
net add interface swp48 bridge access 99
net add bridge bridge mtu 9216
net pending
net commit
~~~

## Generic switch checklist

- LACP active on both ends
- VLAN list restricted to required networks
- LLDP enabled for discovery
- RSTP enabled
- jumbo frames consistent on storage path end-to-end
- logging exported to syslog collector

## IP addressing plan

| Network | Gateway | Static Range | DHCP Range | Notes |
|---------|---------|--------------|------------|-------|
| MGMT 10.10.10.0/24 | 10.10.10.1 | 10.10.10.10-10.10.10.99 | none | static only |
| STORAGE 10.10.20.0/24 | none or routed via storage policy | 10.10.20.10-10.10.20.99 | none | static only, no internet |
| VM-PROD 10.10.30.0/24 | 10.10.30.1 | 10.10.30.10-10.10.30.99 | 10.10.30.150-10.10.30.220 | use DHCP only for selected services |
| VM-DEV 10.10.40.0/24 | 10.10.40.1 | 10.10.40.10-10.10.40.99 | 10.10.40.150-10.10.40.220 | lower trust |
| DMZ 10.10.50.0/24 | 10.10.50.1 | 10.10.50.10-10.10.50.80 | none | public NAT or 1:1 mapping |
| K8S 10.10.60.0/24 | 10.10.60.1 | 10.10.60.10-10.10.60.120 | none | node IPs static |
| OOB 10.10.99.0/24 | 10.10.99.1 | 10.10.99.10-10.10.99.80 | none | static only |

## DNS Design

### Recommendation

Run **internal DNS** on dedicated management VMs or existing hardened DNS appliances.

Options:

| Option | Pros | Cons | Good fit |
|--------|------|------|----------|
| BIND | mature, flexible, standard | more configuration detail | production internal DNS |
| dnsmasq | simple, lightweight | fewer advanced features | small environments and labs |
| AD-integrated DNS | ties into Windows identity | Windows dependency | mixed enterprise shops |

### Forward zone example

`/etc/bind/db.infra.example.com`

~~~bash
$TTL 300
@   IN SOA ns1.infra.example.com. dnsadmin.infra.example.com. (
        2025060101 3600 900 604800 300 )
    IN NS ns1.infra.example.com.
ns1 IN A 10.10.10.53
pve01 IN A 10.10.10.11
pve02 IN A 10.10.10.12
pve03 IN A 10.10.10.13
k8s-api IN A 10.10.60.10
~~~

### Reverse zone example

`/etc/bind/db.10.10.10`

~~~bash
$TTL 300
@   IN SOA ns1.infra.example.com. dnsadmin.infra.example.com. (
        2025060101 3600 900 604800 300 )
    IN NS ns1.infra.example.com.
11  IN PTR pve01.infra.example.com.
12  IN PTR pve02.infra.example.com.
13  IN PTR pve03.infra.example.com.
53  IN PTR ns1.infra.example.com.
~~~

## Service discovery notes

- use DNS names, not hard-coded IPs, for storage, APIs, and internal services
- create records for VIPs, not individual node IPs, when clients should target a load-balanced endpoint
- keep low TTL for failover-sensitive records

## Automation vs Manual

| Area | Recommended Method | Why | Manual Exceptions |
|------|--------------------|-----|------------------|
| Switch config | Ansible `network_cli` | version control, audit trail, repeatability | first boot console setup |
| Host network files | Ansible templates | consistent bond/bridge definitions | emergency break-glass edits |
| IPAM | Git or CMDB-backed source | change history | none |
| Cabling | physical labels and diagrams | reality cannot be automated | all physical work |

### Example Ansible for switch VLANs

~~~yaml
---
- name: Configure VLANs and trunks
  hosts: switches
  connection: network_cli
  gather_facts: false
  tasks:
    - name: Create VLANs
      ios_vlan:
        vlan_id: "{{ item.id }}"
        name: "{{ item.name }}"
        state: present
      loop:
        - { id: 10, name: MGMT }
        - { id: 20, name: STORAGE }
        - { id: 30, name: VM_PROD }
        - { id: 40, name: VM_DEV }
        - { id: 50, name: DMZ }
        - { id: 60, name: K8S }
        - { id: 99, name: OOB }
~~~

## Verification checklist

### VLAN reachability

~~~bash
ping -c 3 10.10.10.12
ping -c 3 10.10.20.12
arping -I vmbr0 10.10.10.1
~~~

### Bond failover

1. run continuous ping from a management host
2. unplug one member link
3. verify traffic continues without prolonged loss
4. confirm bond state still shows one active aggregator

~~~bash
cat /proc/net/bonding/bond0
ip -br link
~~~

### MTU validation for storage

~~~bash
ping -M do -s 8972 10.10.20.12 -c 3
ip link show vmbr1
~~~

### Throughput test

~~~bash
iperf3 -s
iperf3 -c 10.10.20.12 -P 4 -t 30
~~~

## Common trade-offs

### Separate physical fabrics vs shared converged fabric

| Model | Pros | Cons |
|-------|------|------|
| Separate storage and VM fabrics | cleaner isolation, easier troubleshooting | more ports and cost |
| Shared converged fabric | fewer cables and switches | more contention and policy complexity |

### Layer 2 stretch vs routed boundaries

| Model | Pros | Cons |
|-------|------|------|
| Big L2 domains | simple migration patterns | harder fault isolation, larger blast radius |
| Routed per-VLAN boundaries | clearer control and failure containment | more planning required |

**Recommendation:** routed VLAN boundaries with firewall policy between them.

## Exit criteria

You are ready for [03-firewall-and-security.md](./03-firewall-and-security.md) and [04-shared-storage.md](./04-shared-storage.md) when:

- every hypervisor is dual-homed
- management, storage, and tenant VLANs are documented and tested
- DNS forward and reverse records resolve correctly
- MTU and bandwidth baselines are recorded
- switch config is stored in version control or export backups exist


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | Segmented VLAN model | simple fault isolation and policy boundaries | flat L2 network rejected |
| ADR-02 | Dual-switch MLAG/stack | removes single-switch failure domain | single core switch rejected |
| ADR-03 | LACP on host uplinks | bandwidth aggregation and clean failover | active/backup only for all traffic rejected |
| ADR-04 | Separate OOB network | guaranteed rescue path | sharing OOB with in-band rejected |

### Integration points

- Supplies the transport for [01-hypervisor-layer.md](./01-hypervisor-layer.md), [04-shared-storage.md](./04-shared-storage.md), and [09-kubernetes-deployment.md](./09-kubernetes-deployment.md).
- Firewall zoning in [03-firewall-and-security.md](./03-firewall-and-security.md) should map one-to-one with VLAN intent.

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

- Add QoS or traffic shaping only after measuring real contention; storage and control-plane traffic get priority first.
- For multi-site DR, keep site-local VLAN numbering but use unique subnets per site to simplify routing and failover.
- Compliance impact: PCI requires tighter cardholder environment segmentation; SOC 2 needs documented change control and log retention; HIPAA needs secure admin paths and auditability.
- Alert on interface errors, LACP member loss, MAC flaps, spanning-tree anomalies, BGP/OSPF neighbor loss if routed, and switch CPU/memory pressure.
- Auto-remediation can shut an errdisabled port or move host bonds to active/backup during a maintenance event, but keep human approval for core changes.

## Building & Deployment Runbook

1. Build the VLAN/IP spreadsheet and reserve addresses for gateways, VIPs, hypervisors, storage, and load balancers.
2. Bootstrap both switches with management IPs and save a clean baseline config.
3. Configure stack/MLAG peer links, then create VLANs and routed interfaces where needed.
4. Create host-facing port-channels and trunk the required VLANs.
5. Configure host bonds and bridges, then verify from the host:
   ~~~bash
   ip -br link
   cat /proc/net/bonding/bond0
   bridge vlan show
   ~~~
6. Validate switch state:
   ~~~bash
   show interface trunk
   show etherchannel summary
   show lacp neighbor
   ~~~
7. Run `ping`, `traceroute`, `iperf3`, and jumbo-frame tests before allowing storage or production VMs.
8. Validation gate: unplug one member of each bond, confirm no traffic loss beyond expected convergence, then document measured bandwidth.

### Common mistakes to avoid during build

- Using the same VLAN for storage and management because “it is only a lab right now.”
- Forgetting reverse DNS, which later breaks tooling and troubleshooting.
- Buying optics after hardware arrives and delaying implementation.
- Designing no free ports for expansion, tap/SPAN, or DR links.


## Cross-references

- Security policy: [03-firewall-and-security.md](./03-firewall-and-security.md)
- Storage network tuning: [04-shared-storage.md](./04-shared-storage.md)
- Related repo material: [../Physical-Setup/03-network-architecture.md](../Physical-Setup/03-network-architecture.md)
- Related VM networking concepts: [../Virtual-Setup/01-virtualization-fundamentals.md](../Virtual-Setup/01-virtualization-fundamentals.md)
