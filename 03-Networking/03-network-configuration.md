# Network Configuration

Linux provides several ways to configure network interfaces depending on the distribution and age of the system.

## 3.1 Core configuration tasks
Typical interface configuration includes:

- Setting link state
- Assigning IPv4 or IPv6 addresses
- Defining the default gateway
- Adding static routes
- Configuring DNS resolvers
- Enabling DHCP or static addressing

## 3.2 Modern `ip` command suite
The `ip` command from `iproute2` is the standard tool on modern Linux.

### 3.2.1 Show addresses
```bash
ip addr show
ip a
```

### 3.2.2 Show one interface
```bash
ip addr show dev eth0
```

### 3.2.3 Bring interface up or down
```bash
sudo ip link set dev eth0 up
sudo ip link set dev eth0 down
```

### 3.2.4 Add an IPv4 address
```bash
sudo ip addr add 192.168.10.20/24 dev eth0
```

### 3.2.5 Remove an IPv4 address
```bash
sudo ip addr del 192.168.10.20/24 dev eth0
```

### 3.2.6 Add a default route
```bash
sudo ip route add default via 192.168.10.1
```

### 3.2.7 Add a static route
```bash
sudo ip route add 10.20.30.0/24 via 192.168.10.1 dev eth0
```

### 3.2.8 Show routing table
```bash
ip route show
ip r
```

### 3.2.9 Show IPv6 routes
```bash
ip -6 route show
```

### 3.2.10 Show link details
```bash
ip -details link show
```

## 3.3 Understanding `ip addr` output
Example:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.20/24 brd 192.168.10.255 scope global eth0
    inet6 fe80::5054:ff:fe12:3456/64 scope link
```

Meaning:

- `UP` means administratively up.
- `LOWER_UP` usually means link detected.
- `mtu 1500` is the interface MTU.
- `inet` is the IPv4 address.
- `inet6` is the IPv6 address.

## 3.4 `ip link`
Use `ip link` for Layer 2 interface operations.

Examples:

```bash
ip link show
sudo ip link set dev eth0 mtu 9000
sudo ip link set dev eth0 address 00:11:22:33:44:55
```

## 3.5 `ip route`
Routing commands are frequently used in troubleshooting and system bootstrap.

Examples:

```bash
ip route get 8.8.8.8
ip route add 172.16.0.0/16 via 192.168.10.1
ip route del 172.16.0.0/16 via 192.168.10.1
```

## 3.6 `ifconfig` and `route` legacy commands
Older systems may still have `ifconfig` and `route` via `net-tools`.

Examples:

```bash
ifconfig
ifconfig eth0 192.168.10.20 netmask 255.255.255.0 up
route -n
route add default gw 192.168.10.1
```

Why legacy tools are discouraged:

- Not actively developed like `iproute2`
- Limited support for modern features
- Inconsistent behavior across distros

## 3.7 Persistent configuration methods by distribution
Different Linux distributions persist network settings differently.

| Distribution Family | Common Method |
|---|---|
| Ubuntu Server (newer) | Netplan |
| Ubuntu/Debian (older) | `/etc/network/interfaces` |
| RHEL/CentOS 7+ | NetworkManager or legacy scripts |
| RHEL 8/9 | NetworkManager with `nmcli` |
| Desktop distros | NetworkManager |

## 3.8 NetworkManager overview
NetworkManager is a service that manages interfaces, connections, routing, and DNS integration.

Useful commands:

```bash
nmcli general status
nmcli device status
nmcli connection show
```

## 3.9 `nmcli` command examples
### 3.9.1 Show devices
```bash
nmcli device status
```

### 3.9.2 Show connections
```bash
nmcli connection show
```

### 3.9.3 Bring a connection up
```bash
sudo nmcli connection up "System eth0"
```

### 3.9.4 Bring a connection down
```bash
sudo nmcli connection down "System eth0"
```

### 3.9.5 Configure static IPv4
```bash
sudo nmcli connection modify eth0 \
  ipv4.addresses 192.168.10.20/24 \
  ipv4.gateway 192.168.10.1 \
  ipv4.dns "1.1.1.1 8.8.8.8" \
  ipv4.method manual
```

### 3.9.6 Enable DHCP
```bash
sudo nmcli connection modify eth0 ipv4.method auto
```

### 3.9.7 Apply changes
```bash
sudo nmcli connection down eth0
sudo nmcli connection up eth0
```

## 3.10 `nmtui`
`nmtui` is a text-based UI for NetworkManager.

Good use cases:

- Quick console configuration
- Virtual machines without a GUI
- Administrators who prefer menus over CLI syntax

Launch it with:

```bash
sudo nmtui
```

Menu options generally include:

- Edit a connection
- Activate a connection
- Set system hostname

## 3.11 Netplan overview
Netplan is common on Ubuntu Server.

YAML files are typically stored in:

```text
/etc/netplan/
```

Common file names:

- `00-installer-config.yaml`
- `50-cloud-init.yaml`
- `01-netcfg.yaml`

### 3.11.1 Apply configuration
```bash
sudo netplan generate
sudo netplan apply
```

### 3.11.2 Test configuration safely
```bash
sudo netplan try
```

`netplan try` is safer for remote servers because it can roll back if connectivity is lost.

## 3.12 Netplan DHCP example for Ubuntu
```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
      dhcp6: false
```

## 3.13 Netplan static IP example for Ubuntu
```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.10.20/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
        search:
          - example.local
```

## 3.14 Netplan static IPv6 example
```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp6: false
      addresses:
        - 2001:db8:100::20/64
      routes:
        - to: default
          via: 2001:db8:100::1
      nameservers:
        addresses:
          - 2606:4700:4700::1111
          - 2001:4860:4860::8888
```

## 3.15 `/etc/network/interfaces` overview
Older Debian and Ubuntu systems often use this file.

Typical path:

```text
/etc/network/interfaces
```

### 3.15.1 DHCP example
```ini
auto eth0
iface eth0 inet dhcp
```

### 3.15.2 Static IPv4 example
```ini
auto eth0
iface eth0 inet static
    address 192.168.10.20
    netmask 255.255.255.0
    gateway 192.168.10.1
    dns-nameservers 1.1.1.1 8.8.8.8
    dns-search example.local
```

### 3.15.3 Bring interface down and up
```bash
sudo ifdown eth0 && sudo ifup eth0
```

## 3.16 CentOS/RHEL legacy ifcfg scripts
Older RHEL/CentOS systems use files such as:

```text
/etc/sysconfig/network-scripts/ifcfg-eth0
```

### 3.16.1 Static example
```ini
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.10.20
PREFIX=24
GATEWAY=192.168.10.1
DNS1=1.1.1.1
DNS2=8.8.8.8
```

### 3.16.2 DHCP example
```ini
TYPE=Ethernet
BOOTPROTO=dhcp
NAME=eth0
DEVICE=eth0
ONBOOT=yes
```

Restart networking carefully:

```bash
sudo systemctl restart NetworkManager
```

or on older systems:

```bash
sudo systemctl restart network
```

## 3.17 Static vs DHCP
| Aspect | Static | DHCP |
|---|---|---|
| Address stability | Fixed | May change |
| Server use | Common | Sometimes used |
| Client use | Less common | Very common |
| Management | Manual | Centralized |
| Risk | Misconfiguration | Lease dependency |

Use static addressing for:

- Servers
- Gateways
- Load balancers
- Hypervisors
- DNS servers

Use DHCP for:

- User laptops
- Workstations
- Ephemeral test VMs
- Lab environments

## 3.18 Ubuntu examples: static and DHCP
### 3.18.1 Ubuntu static example with Netplan
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens160:
      dhcp4: false
      addresses:
        - 10.10.20.15/24
      routes:
        - to: default
          via: 10.10.20.1
      nameservers:
        addresses:
          - 10.10.1.53
          - 1.1.1.1
        search:
          - corp.example.com
```

### 3.18.2 Ubuntu DHCP example with Netplan
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens160:
      dhcp4: true
      dhcp-identifier: mac
```

## 3.19 CentOS/RHEL examples: static and DHCP
### 3.19.1 RHEL static with `nmcli`
```bash
sudo nmcli connection add type ethernet ifname ens160 con-name ens160 \
  ipv4.method manual \
  ipv4.addresses 10.10.20.15/24 \
  ipv4.gateway 10.10.20.1 \
  ipv4.dns "10.10.1.53 1.1.1.1" \
  autoconnect yes
```

### 3.19.2 RHEL DHCP with `nmcli`
```bash
sudo nmcli connection add type ethernet ifname ens160 con-name ens160 \
  ipv4.method auto \
  autoconnect yes
```

## 3.20 Configuring multiple IPs on one interface
```bash
sudo ip addr add 192.168.10.21/24 dev eth0
sudo ip addr add 192.168.10.22/24 dev eth0
```

Common uses:

- Virtual hosting
- Migration cutovers
- Testing
- Legacy app coexistence

## 3.21 Temporary vs persistent changes
Commands like `ip addr add` are temporary.

They usually disappear after reboot or service restart.

Persistent configuration should be set through:

- Netplan
- NetworkManager
- `/etc/network/interfaces`
- Distribution-specific configuration files

## 3.22 Hostname configuration
Show hostname:

```bash
hostnamectl status
```

Set hostname:

```bash
sudo hostnamectl set-hostname web01.example.com
```

Ensure DNS and reverse DNS align with hostname standards in production environments.

## 3.23 Interface statistics and errors
```bash
ip -s link show dev eth0
```

Look for:

- RX errors
- TX errors
- Dropped packets
- Overruns
- Carrier errors

## 3.24 MAC address changes
Example:

```bash
sudo ip link set dev eth0 down
sudo ip link set dev eth0 address 02:11:22:33:44:55
sudo ip link set dev eth0 up
```

Use carefully. Some networks enforce port security or DHCP reservations.

## 3.25 MTU changes
Temporary MTU change:

```bash
sudo ip link set dev eth0 mtu 9000
```

Persistent MTU should be configured using the distro’s network management system.

Example Netplan:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      mtu: 9000
      dhcp4: true
```

## 3.26 Routing metrics
Metrics determine route preference when multiple routes exist.

Example:

```bash
sudo ip route add default via 192.168.10.1 metric 100
sudo ip route add default via 192.168.20.1 metric 200
```

The lower metric is preferred.

## 3.27 Policy routing overview
Advanced routing may use multiple routing tables and rules.

Examples:

- Source-based routing
- Multi-homed servers
- VPN split routing

Useful commands:

```bash
ip rule show
ip route show table main
ip route show table all
```

## 3.28 Static route examples
### 3.28.1 Temporary route
```bash
sudo ip route add 10.50.0.0/16 via 192.168.10.1 dev eth0
```

### 3.28.2 Host route
```bash
sudo ip route add 203.0.113.50/32 via 192.168.10.254
```

### 3.28.3 Blackhole route
```bash
sudo ip route add blackhole 198.51.100.0/24
```

## 3.29 DHCP client tools
Common tools:

- `dhclient`
- `systemd-networkd`
- NetworkManager internal DHCP handling

Examples:

```bash
sudo dhclient -v eth0
sudo dhclient -r eth0
```

## 3.30 Name resolution and networking services
When network settings change, verify these layers too:

- DNS resolver settings
- Routing table correctness
- Firewall rules
- Service bind address
- SELinux or AppArmor policy if relevant

## 3.31 Safe remote change workflow
When changing remote server networking:

1. Open a persistent SSH session.
2. Open a second backup SSH session.
3. Use `tmux` or `screen`.
4. Schedule rollback if possible.
5. Prefer `netplan try` or staged `nmcli` changes.
6. Validate before logging out.

## 3.32 Validation commands after configuration
```bash
ip addr show
ip route show
ping -c 3 <gateway>
ping -c 3 8.8.8.8
dig example.com
curl -I https://example.com
```

## 3.33 Common configuration mistakes
- Wrong prefix length
- Wrong gateway
- Duplicate IP
- Missing DNS
- Interface name mismatch
- MTU mismatch
- Route added to wrong interface
- Persistent config differs from runtime config

## 3.34 Quick comparison table
| Tool | Best For | Notes |
|---|---|---|
| `ip` | Runtime changes and inspection | Preferred modern tool |
| `ifconfig` | Legacy systems | Deprecated in many environments |
| `nmcli` | Persistent config on NM systems | Script-friendly |
| `nmtui` | Interactive text UI | Fast for console use |
| Netplan | Ubuntu Server | YAML-based |
| `/etc/network/interfaces` | Older Debian/Ubuntu | Legacy but still encountered |

## 3.35 Summary
Use `iproute2` for day-to-day inspection and temporary changes. Use the distribution-native persistence mechanism for durable configuration.

---

# Command Reference for Configuration

## A.1 Interface and address commands

```bash
ip addr
ip -br addr
ip link
ip -s link
nmcli device status
ifconfig
```

## A.2 Routing commands

```bash
ip route
ip route get 8.8.8.8
ip rule
route -n
traceroute example.com
```

---

# Lab Exercises

## C.1 Exercise 1: Inspect interface state

Goal:

- Learn to inspect addresses, link state, and routes.

Commands:

```bash
ip -br addr
ip -br link
ip route
```

Questions:

- Which interface has the default route?
- Which interfaces are up?
- Which address is on the LAN?

## C.2 Exercise 2: Add a temporary static route

Commands:

```bash
sudo ip route add 10.50.0.0/16 via 192.168.10.1
ip route show
```

Then remove it:

```bash
sudo ip route del 10.50.0.0/16 via 192.168.10.1
```

---

## 3.9 Connect a Linux host to multiple VLANs on one NIC
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

# Change and Persistence Checklists

## D.6 Change management checklist

- [ ] Keep console access for remote changes.
- [ ] Use rollback-friendly methods.
- [ ] Make one networking change at a time.
- [ ] Document before-and-after state.
- [ ] Test locally and remotely.

---

## E.4 Linux network file locations quick list

| Purpose | Common Path |
|---|---|
| Resolver config | `/etc/resolv.conf` |
| Static host entries | `/etc/hosts` |
| NSS order | `/etc/nsswitch.conf` |
| SSH server config | `/etc/ssh/sshd_config` |
| SSH client config | `~/.ssh/config` |
| Netplan configs | `/etc/netplan/` |
| Debian legacy interface config | `/etc/network/interfaces` |
| RHEL legacy interface scripts | `/etc/sysconfig/network-scripts/` |
| nftables main config | `/etc/nftables.conf` |
| sysctl settings | `/etc/sysctl.conf` or `/etc/sysctl.d/` |
| WireGuard configs | `/etc/wireguard/` |
| HAProxy config | `/etc/haproxy/haproxy.cfg` |

## E.5 Useful service management commands

```bash
systemctl status NetworkManager
systemctl status systemd-networkd
systemctl status systemd-resolved
systemctl status sshd
systemctl status firewalld
systemctl status nftables
```

## E.6 NetworkManager troubleshooting tips

- Check if the connection profile exists.
- Confirm the right profile is active on the right interface.
- Verify whether DNS is managed by NetworkManager or another resolver layer.
- Inspect logs with `journalctl -u NetworkManager`.

## E.7 Netplan troubleshooting tips

- Validate YAML indentation carefully.
- Use `netplan try` when remote.
- Know whether renderer is `networkd` or `NetworkManager`.
- Confirm cloud-init is not overwriting files.

---

## E.17 Change safety checklist for remote servers

Before network changes:

- Save current config.
- Keep a second session open.
- Know console access method.
- Identify rollback command.
- Validate syntax first.
- Apply change during supportable window.

After network changes:

- Verify route table.
- Verify DNS.
- Verify app connectivity.
- Verify monitoring still reaches the host.

---

## E.26 Naming conventions for clarity

Good naming examples:

- `bond0`
- `bond0.100`
- `br-mgmt`
- `br-storage`
- `wg0`
- `tun0`
- `vlan10`

Consistent names reduce operational mistakes.

## E.27 Choosing the right persistence method

If a system uses NetworkManager, prefer NetworkManager.

If a system uses Netplan, prefer Netplan.

Avoid mixing:

- manual `ip` changes as long-term config
- hand edits to generated files
- multiple daemons competing for DNS or interface control

---

## Q.3 Admin task to file mapping

| Admin Task | Common File |
|---|---|
| Set static DNS | `/etc/resolv.conf` or NetworkManager/Netplan source |
| Pin local name | `/etc/hosts` |
| Change resolver order | `/etc/nsswitch.conf` |
| Harden SSH | `/etc/ssh/sshd_config` |
| Configure WireGuard | `/etc/wireguard/wg0.conf` |
| Configure HAProxy | `/etc/haproxy/haproxy.cfg` |
| Set sysctl forwarding | `/etc/sysctl.conf` |
