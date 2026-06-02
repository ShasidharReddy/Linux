# Bonding and VLANs

Bonding, teaming, VLAN tagging, and bridges are common in servers, hypervisors, and virtualization stacks.

## 8.1 Why bonding?

Bonding combines multiple physical NICs into one logical interface for:

- Redundancy
- Higher throughput in some scenarios
- Cleaner management

## 8.2 Bonding modes overview

| Mode | Name | Description |
|---|---|---|
| 0 | balance-rr | Round-robin |
| 1 | active-backup | One active, one standby |
| 2 | balance-xor | XOR policy |
| 3 | broadcast | Sends on all slaves |
| 4 | 802.3ad | LACP |
| 5 | balance-tlb | Adaptive transmit load balancing |
| 6 | balance-alb | Adaptive load balancing |

## 8.3 Choosing a bond mode

General guidance:

- Use `active-backup` for simple redundancy.
- Use `802.3ad` when switches support LACP.
- Use other modes only with clear design intent.

## 8.4 Inspect bonding status

```bash
cat /proc/net/bonding/bond0
```

## 8.5 `active-backup` conceptual behavior

- One NIC carries traffic.
- Another waits in standby.
- Failover occurs if the active link fails.

Good for:

- Simplicity
- High compatibility
- Minimal switch configuration

## 8.6 LACP and 802.3ad

LACP dynamically negotiates link aggregation between host and switch.

Requirements:

- Switch supports LACP
- Correct switch-side port-channel config
- Matching bond mode on Linux

Benefits:

- Redundancy
- Better aggregate throughput across multiple flows

Note:

A single TCP flow usually does not exceed one member’s bandwidth due to hashing.

## 8.7 NetworkManager bond example

Create bond:

```bash
sudo nmcli connection add type bond ifname bond0 mode active-backup con-name bond0
sudo nmcli connection add type ethernet ifname ens1 master bond0
sudo nmcli connection add type ethernet ifname ens2 master bond0
sudo nmcli connection modify bond0 ipv4.addresses 10.10.20.10/24 ipv4.gateway 10.10.20.1 ipv4.method manual
sudo nmcli connection up bond0
```

## 8.8 Netplan bond example

```yaml
network:
  version: 2
  bonds:
    bond0:
      interfaces:
        - ens1
        - ens2
      parameters:
        mode: active-backup
        primary: ens1
      addresses:
        - 10.10.20.10/24
      routes:
        - to: default
          via: 10.10.20.1
      nameservers:
        addresses:
          - 1.1.1.1
```

## 8.9 VLAN basics

A VLAN logically separates Layer 2 networks using 802.1Q tags.

Benefits:

- Isolation
- Better segmentation
- Reduced broadcast domain size
- Multi-tenant designs

## 8.10 Access vs trunk ports

| Port Type | Behavior |
|---|---|
| Access | Carries one untagged VLAN |
| Trunk | Carries multiple tagged VLANs |

Linux can tag frames on trunk-connected interfaces.

## 8.11 Create VLAN interface with `ip`

```bash
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip addr add 192.168.10.20/24 dev eth0.10
sudo ip link set dev eth0.10 up
```

## 8.12 Show VLANs

```bash
ip -d link show type vlan
```

## 8.13 NetworkManager VLAN example

```bash
sudo nmcli connection add type vlan con-name vlan10 dev eth0 id 10 ifname eth0.10 \
  ipv4.addresses 192.168.10.20/24 ipv4.gateway 192.168.10.1 ipv4.method manual
```

## 8.14 Netplan VLAN example

```yaml
network:
  version: 2
  ethernets:
    eth0: {}
  vlans:
    vlan10:
      id: 10
      link: eth0
      addresses:
        - 192.168.10.20/24
      routes:
        - to: default
          via: 192.168.10.1
```

## 8.15 Bridge interfaces

A bridge connects interfaces at Layer 2.

Common use cases:

- KVM virtualization
- Container networking
- Software switching

## 8.16 Create bridge with `ip`

```bash
sudo ip link add name br0 type bridge
sudo ip link set dev br0 up
sudo ip link set dev eth0 master br0
```

Assign IP to the bridge instead of the slave interface when using bridge-host routing.

## 8.17 Bridge example with NetworkManager

```bash
sudo nmcli connection add type bridge ifname br0 con-name br0
sudo nmcli connection add type bridge-slave ifname ens3 master br0
sudo nmcli connection modify br0 ipv4.addresses 10.10.20.50/24 ipv4.gateway 10.10.20.1 ipv4.method manual
sudo nmcli connection up br0
```

## 8.18 Bond plus VLAN designs

Common pattern:

- Physical NICs bonded into `bond0`
- VLAN subinterfaces on top of `bond0`
- Optionally bridged for VMs

Example objects:

- `bond0`
- `bond0.10`
- `bond0.20`
- `br-mgmt`

## 8.19 Bridge plus VLAN awareness

Modern Linux bridges can be VLAN-aware.

Useful in:

- Virtualization hosts
- Container platforms
- Multi-network VM environments

## 8.20 Typical hypervisor design

Example:

- `bond0` for physical redundancy
- `bond0.100` management network
- `bond0.200` storage network
- `bond0.300` tenant network trunk
- `br0` as VM bridge

## 8.21 Validate bonding and VLANs

Commands:

```bash
ip link
ip addr
cat /proc/net/bonding/bond0
bridge link
bridge vlan show
```

## 8.22 Common mistakes

- Switch not configured for LACP
- Wrong native VLAN
- IP assigned to slave instead of bridge
- VLAN tag mismatch
- MTU inconsistency across bonded or tagged path
- Using unsupported bond mode with switch config

## 8.23 Troubleshooting bond issues

Check:

- Member links up?
- Bond mode correct?
- Switch side configured?
- MAC flapping on switch?
- `cat /proc/net/bonding/bond0`

## 8.24 Troubleshooting VLAN issues

Check:

- VLAN exists on switch trunk?
- Correct VLAN ID?
- Interface up?
- Gateway in same VLAN?
- Firewall blocking?

## 8.25 Summary

Bonding and VLANs are foundational for resilient and segmented Linux infrastructure. Validate both host and switch-side settings together.

---

## 12.9 Connect a Linux host to multiple VLANs on one NIC

Objective:

- Use a trunk port from the switch
- Terminate VLANs 10 and 20 on the server

Commands:

```bash
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip addr add 10.10.10.50/24 dev eth0.10
sudo ip addr add 10.10.20.50/24 dev eth0.20
sudo ip link set eth0 up
sudo ip link set eth0.10 up
sudo ip link set eth0.20 up
```

Validation:

```bash
ip -d link show eth0.10
ip -d link show eth0.20
bridge vlan show
ping -c 3 10.10.10.1
ping -c 3 10.10.20.1
```

---

## E.14 Virtualization and Linux networking

Hypervisors often use:

- Linux bridges
- Bonded NICs
- VLAN trunks
- Tap devices
- Virtual switches

Always distinguish:

- Host management traffic
- Guest traffic
- Storage traffic
- Migration traffic
