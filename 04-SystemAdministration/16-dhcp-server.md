# DHCP Server

---

<a id="dhcp-server"></a>
## 🧷 DHCP Server

### What DHCP provides
Dynamic Host Configuration Protocol (DHCP) assigns network settings to clients automatically.

Typical values provided by DHCP:
- IP address.
- Subnet mask.
- Default gateway.
- DNS servers.
- Search domain.
- Lease duration.
- Optional PXE, NTP, or vendor-specific parameters.

Benefits:
- Centralized IP address management.
- Fewer client-side configuration errors.
- Easier device replacement and lab automation.
- Better control over reservations and options.

### DHCP transaction summary

```mermaid
graph LR
    A["Client boots"] --> B["DHCPDISCOVER"]
    B --> C["DHCPOFFER"]
    C --> D["DHCPREQUEST"]
    D --> E["DHCPACK"]
    E --> F["Client configures network"]
```

### Install ISC DHCP server
Ubuntu and Debian:

```bash
sudo apt update
sudo apt install -y isc-dhcp-server
```

RHEL-family systems:

```bash
sudo dnf install -y dhcp-server
```

Important note:
- ISC DHCP is mature and still common.
- Many new deployments evaluate Kea for future growth.
- If you run ISC DHCP today, keep the configuration simple, documented, and tested.

### Service files and key paths
Debian and Ubuntu:
- Main config: `/etc/dhcp/dhcpd.conf`
- Defaults file: `/etc/default/isc-dhcp-server`
- Lease database: `/var/lib/dhcp/dhcpd.leases`
- Service name: `isc-dhcp-server`

RHEL-family:
- Main config: `/etc/dhcp/dhcpd.conf`
- Interface file: `/etc/sysconfig/dhcpd`
- Lease database: `/var/lib/dhcpd/dhcpd.leases`
- Service name: `dhcpd`

### Decide the correct deployment model
Before configuring DHCP, decide:
- Which VLAN or subnet this server will serve.
- Whether the server sits on the same L2 segment as clients.
- Whether a relay agent such as `ip helper-address` is required.
- Which address range is dynamic.
- Which devices need fixed reservations.
- Which DNS and NTP servers clients should receive.

### Basic global configuration
Example `/etc/dhcp/dhcpd.conf`:

```conf
authoritative;
default-lease-time 600;
max-lease-time 7200;
log-facility local7;

option domain-name "example.internal";
option domain-name-servers 192.168.1.10, 192.168.1.11;
option ntp-servers 192.168.1.12;

ddns-update-style none;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.199;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    option domain-name "example.internal";
    option domain-name-servers 192.168.1.10, 192.168.1.11;
    option ntp-servers 192.168.1.12;
    default-lease-time 600;
    max-lease-time 3600;
}
```

What this does:
- Declares the server authoritative for the served network.
- Provides a dynamic pool from `.100` to `.199`.
- Returns gateway, DNS, and NTP settings to clients.
- Keeps lease time moderate for office or lab use.

### Multiple subnet definitions
If the server serves more than one network through relay agents, add multiple subnet blocks.

Example:

```conf
authoritative;
default-lease-time 900;
max-lease-time 7200;

option domain-name "example.internal";
option domain-name-servers 192.168.1.10, 192.168.1.11;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.199;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
}

subnet 192.168.20.0 netmask 255.255.255.0 {
    range 192.168.20.100 192.168.20.199;
    option routers 192.168.20.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.20.255;
    option domain-name-servers 192.168.1.10, 192.168.1.11;
}
```

### Fixed reservations
Use reservations for printers, appliances, hypervisors, and hosts that benefit from stable addressing.

Example reservation block:

```conf
host web01 {
    hardware ethernet 52:54:00:ab:cd:01;
    fixed-address 192.168.1.20;
    option host-name "web01";
}

host db01 {
    hardware ethernet 52:54:00:ab:cd:02;
    fixed-address 192.168.1.30;
    option host-name "db01";
}

host printer01 {
    hardware ethernet aa:bb:cc:dd:ee:ff;
    fixed-address 192.168.1.50;
    option host-name "printer01";
}
```

Best practices for reservations:
- Keep reserved addresses outside the dynamic pool where possible.
- Document ownership and purpose of each reservation.
- Verify MAC addresses carefully before rollout.
- Use DHCP reservations instead of hand-configured static IPs when centralized control is preferred.

### Example: production-ready office subnet

```conf
authoritative;
default-lease-time 1800;
max-lease-time 7200;
log-facility local7;
ddns-update-style none;

option domain-name "example.internal";
option domain-name-servers 192.168.1.10, 192.168.1.11;
option ntp-servers 192.168.1.12;
option time-offset 0;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.180;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    option domain-name "example.internal";
    option domain-name-servers 192.168.1.10, 192.168.1.11;
    option ntp-servers 192.168.1.12;
    default-lease-time 1800;
    max-lease-time 7200;

    host fileserver01 {
        hardware ethernet 52:54:00:11:22:33;
        fixed-address 192.168.1.21;
        option host-name "fileserver01";
    }

    host app01 {
        hardware ethernet 52:54:00:11:22:34;
        fixed-address 192.168.1.22;
        option host-name "app01";
    }
}
```

### Interface binding
On Debian and Ubuntu, define the interface in `/etc/default/isc-dhcp-server`:

```bash
INTERFACESv4="eth0"
INTERFACESv6=""
```

On RHEL-family systems, define it in `/etc/sysconfig/dhcpd`:

```bash
DHCPDARGS=eth0
```

If you do not bind to the correct interface, the service may start but not answer clients as expected.

### Validate configuration before starting
Run a syntax check:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

Start and enable the service:

```bash
sudo systemctl enable --now isc-dhcp-server
sudo systemctl enable --now dhcpd
```

Use the service name that exists on your system.

Review logs after startup:

```bash
sudo journalctl -u isc-dhcp-server -u dhcpd --since -30m
```

### Firewall and network requirements
Open UDP 67 on the server.

RHEL-family with firewalld:

```bash
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --reload
```

Requirements beyond the server host:
- Clients must be on the same broadcast domain or use a DHCP relay.
- Routers must forward DHCP requests correctly.
- No rogue DHCP server should be present on the segment.

### DHCP relay notes
If the DHCP server is not on the same VLAN as clients, configure the network device to relay DHCP.

Common pattern on routers and L3 switches:
- Cisco-like syntax often uses `ip helper-address <dhcp-server-ip>`.
- Linux relays can use `dhcrelay` where appropriate.

Relay design tips:
- Keep relay targets explicit.
- Confirm relay source interface belongs to the intended subnet.
- Monitor for duplicate or misrouted offers.

### Client testing workflow
From a Linux client using NetworkManager:

```bash
sudo nmcli con down "Wired connection 1"
sudo nmcli con up "Wired connection 1"
ip addr show
ip route
resolvectl status
```

From a traditional client using `dhclient`:

```bash
sudo dhclient -r eth0
sudo dhclient -v eth0
```

What to confirm:
- The client receives an address in the expected range.
- The default route is correct.
- DNS servers match policy.
- Reserved hosts receive their fixed addresses.

### Lease file inspection
Lease state is stored on disk and is useful during troubleshooting.

Examples:

```bash
sudo tail -50 /var/lib/dhcp/dhcpd.leases
sudo grep -n "192.168.1.120" /var/lib/dhcp/dhcpd.leases
```

Use the lease file to confirm:
- A lease was offered and acknowledged.
- A MAC address matches a reservation.
- Lease times are reasonable.
- Old stale entries are not confusing your investigation.

### Optional DHCP options worth knowing

| Option | Purpose | Example |
|---|---|---|
| `option routers` | Default gateway | `option routers 192.168.1.1;` |
| `option domain-name-servers` | DNS servers | `option domain-name-servers 192.168.1.10;` |
| `option domain-name` | Search domain | `option domain-name "example.internal";` |
| `option ntp-servers` | Time servers | `option ntp-servers 192.168.1.12;` |
| `next-server` | PXE/TFTP host | `next-server 192.168.1.60;` |
| `filename` | PXE boot file | `filename "pxelinux.0";` |

PXE example snippet:

```conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.199;
    option routers 192.168.1.1;
    next-server 192.168.1.60;
    filename "pxelinux.0";
}
```

### Common DHCP troubleshooting

#### Service starts but no clients receive addresses
Check:
- Correct interface binding.
- Firewall rules.
- Relay configuration if clients are on another subnet.
- Another DHCP server on the same segment causing confusion.

Useful commands:

```bash
sudo ss -uapn | grep :67
sudo tcpdump -ni eth0 port 67 or port 68
sudo journalctl -u isc-dhcp-server -u dhcpd --since -30m
```

#### `No subnet declaration for <interface>`
This means the server is listening on an interface whose network is not declared in `dhcpd.conf`.

Fix options:
- Add the corresponding subnet block.
- Bind the service only to the intended interface.

#### Reservations are ignored
Check:
- MAC address accuracy.
- Whether the client already holds a previous lease.
- Whether the reserved address sits inside an overlapping dynamic pool.
- Whether the client identifies with the expected interface.

#### Clients get IP but not DNS or gateway
Check:
- `option routers` and `option domain-name-servers` lines.
- Whether subnet-specific options override global defaults.
- Client-side caching or NetworkManager state.

#### Intermittent lease failures
Investigate:
- VLAN misconfiguration.
- Relay packet loss.
- Duplicate DHCP servers.
- Exhausted pool range.
- Server process restarts or lease database corruption.

### DHCP security and operational practices
- Never run an unauthorized DHCP server on a production segment.
- Keep address pools clearly separated from reserved and infrastructure ranges.
- Back up `dhcpd.conf` and, where appropriate, lease state.
- Use switch protections such as DHCP snooping where supported.
- Log all reservation changes through change control.
- Keep DNS and DHCP data consistent.
- Monitor for pool exhaustion.

### DHCP operational checklist
- Validate config with `dhcpd -t` before restart.
- Confirm the correct interface or relay path.
- Test a dynamic client and a reserved client.
- Confirm DNS and gateway options from the client side.
- Monitor lease pool utilization.
- Remove obsolete reservations.
- Document every served subnet.

### DHCP quick reference

```bash
# Install
sudo apt install isc-dhcp-server
sudo dnf install dhcp-server

# Validate
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# Service control
sudo systemctl enable --now isc-dhcp-server
sudo systemctl enable --now dhcpd

# Troubleshooting
sudo journalctl -u isc-dhcp-server -u dhcpd --since -30m
sudo tcpdump -ni eth0 port 67 or port 68
```
