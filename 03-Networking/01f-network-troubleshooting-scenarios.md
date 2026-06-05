# Network Troubleshooting Scenarios

← Back to [01-fundamentals.md](./01-fundamentals.md)

Quick references, glossary, end-to-end scenarios, packet stories, and production checklists.

---

## 15. Quick Reference Tables
<a id="section-15"></a>

### 15.1 Common well-known ports

| Service | Port | Protocol | Typical purpose |
|---|---:|---|---|
| FTP data | 20 | TCP | Legacy file transfer data channel |
| FTP control | 21 | TCP | Legacy file transfer control channel |
| SSH | 22 | TCP | Remote shell and secure copy |
| SMTP | 25 | TCP | Mail transfer |
| DNS | 53 | UDP/TCP | Name resolution |
| DHCP server | 67 | UDP | Address assignment server |
| DHCP client | 68 | UDP | Address assignment client |
| HTTP | 80 | TCP | Unencrypted web traffic |
| POP3 | 110 | TCP | Legacy mail retrieval |
| NTP | 123 | UDP | Time synchronization |
| IMAP | 143 | TCP | Mail retrieval and sync |
| SNMP | 161 | UDP | Network management |
| HTTPS | 443 | TCP | Encrypted web traffic |
| SMB | 445 | TCP | Windows file sharing |
| LDAPS | 636 | TCP | Encrypted LDAP |
| NFS | 2049 | TCP/UDP | Network file system |
| OpenVPN | 1194 | UDP | VPN transport |
| WireGuard | 51820 | UDP | VPN transport |

### 15.2 Useful Linux networking commands

| Goal | Command |
|---|---|
| Show addresses | `ip addr` |
| Show routes | `ip route` |
| Show links | `ip link` |
| Show neighbors | `ip neigh` |
| Show TCP/UDP sockets | `ss -tulpen` |
| Test ICMP reachability | `ping` |
| Trace a path | `traceroute` or `tracepath` |
| Query DNS | `dig` or `resolvectl query` |
| Capture packets | `tcpdump` |
| Inspect NIC settings | `ethtool` |


### 15.2.1 Sample command outputs

```bash
$ ip a
# Expected output:
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
#     inet 192.168.1.20/24 brd 192.168.1.255 scope global dynamic eth0
#     inet6 fe80::5054:ff:fe12:3456/64 scope link
```

```bash
$ ip route
# Expected output:
# default via 192.168.1.1 dev eth0 proto dhcp metric 100
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.20 metric 100
```

```bash
$ ss -tulpn
# Expected output:
# Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
# tcp    LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1220,fd=3))
# udp    UNCONN  0       0       127.0.0.53:53       0.0.0.0:*          users:(("systemd-resolve",pid=812,fd=13))
```

```bash
$ dig example.com
# Sample output:
# ;; ANSWER SECTION:
# example.com.        300     IN      A       93.184.216.34
```

```bash
$ ping -c 2 8.8.8.8
# Sample output:
# 64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.2 ms
# 64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.0 ms
# --- 8.8.8.8 ping statistics ---
# 2 packets transmitted, 2 received, 0% packet loss
```

```bash
$ traceroute example.com
# Sample output:
#  1  192.168.1.1  1.021 ms  0.924 ms  0.887 ms
#  2  10.20.0.1    4.311 ms  4.267 ms  4.120 ms
#  3  93.184.216.34  18.552 ms  18.311 ms  18.204 ms
```

### 15.3 PDU quick reference

- Application, Presentation, Session: Data
- Transport: Segment or datagram
- Network: Packet
- Data Link: Frame
- Physical: Bits

### 15.4 Common failure symptoms

- No carrier: Layer 1 or 2 physical issue.
- ARP incomplete: local Layer 2 reachability issue.
- No route to host: Layer 3 routing issue.
- SYN retransmissions: transport path or filtering issue.
- TLS alert: certificate or protocol mismatch issue.
- HTTP 5xx: application issue.

---

## Section 16
## 16. Glossary and Mental Models
<a id="section-16"></a>

### 16.1 Glossary

- **ACK**: A TCP acknowledgment indicating the next byte expected from the peer.
- **ARP**: Address Resolution Protocol, used to map IPv4 addresses to MAC addresses on a local link.
- **CIDR**: Classless Inter-Domain Routing, notation such as `/24` that tells you the prefix length.
- **Datagram**: A self-contained message unit used by connectionless transports such as UDP.
- **Default gateway**: The router a host uses for destinations outside the local subnet.
- **DNS**: Domain Name System, which maps names to resource records.
- **Encapsulation**: Wrapping higher-layer data with lower-layer headers.
- **Ethernet frame**: A Layer 2 unit carrying MAC addresses and an upper-layer payload.
- **FIN**: A TCP flag meaning the sender has finished sending data in one direction.
- **FQDN**: Fully Qualified Domain Name.
- **Hop**: One routed step between source and destination.
- **IP packet**: A Layer 3 unit carrying source and destination IP addressing metadata.
- **MAC address**: A Layer 2 hardware identifier used on the local segment.
- **MTU**: Maximum Transmission Unit, the largest payload that fits on a link without fragmentation.
- **NAT**: Network Address Translation between address domains.
- **PDU**: Protocol Data Unit, the name of the data object at a given layer.
- **Port**: A transport-layer identifier for a service endpoint on a host.
- **Retransmission**: Sending a TCP segment again because the original was assumed lost.
- **Segment**: A TCP transport-layer data unit.
- **SNI**: Server Name Indication, a TLS extension carrying the intended hostname.
- **SYN**: A TCP flag used to synchronize sequence numbers and start a connection.
- **TTL**: Time To Live, reduced by each routed hop to prevent loops.
- **Window size**: TCP flow-control advertisement indicating how much data the receiver can currently accept.

### 16.2 Mental model summaries

- OSI layers are a troubleshooting map.
- TCP/IP is the practical implementation map.
- MAC addresses matter only on the current local link.
- IP addresses matter across routed networks.
- Ports identify applications, not machines by themselves.
- Routers move packets between networks by longest-prefix match.
- Switches move frames inside a local Layer 2 domain by MAC learning.
- DNS tells you where to go by name.
- ARP tells you how to reach the next hop on the current Ethernet segment.
- TCP gives you a reliable ordered byte stream on top of an unreliable IP network.

## Appendix A — End-to-End Troubleshooting Scenarios

### Appendix A.1 Website does not resolve

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.2 Website resolves but does not connect

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.3 TCP connects but TLS fails

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.4 TLS works but HTTP returns errors

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.5 Only one VLAN cannot reach the service

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.6 Works on IPv4 but not IPv6

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.7 Works locally but not through VPN

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.8 Intermittent packet loss under load

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.9 Connection stalls after initial data

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.10 One-way traffic after NAT

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.11 Random slowness during peak hours

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.12 Duplicate IP address symptoms

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.13 Asymmetric routing suspicion

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.14 Many CLOSE_WAIT sockets on server

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.15 Huge TIME_WAIT on client host

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.16 DNS answers differ by location

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.17 Only HTTPS fails while HTTP works

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.18 SSH works but large file transfer stalls

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.19 Ping works but app still fails

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.
- Use packet capture to prove whether packets leave, arrive, and return.
- Correlate captures with server logs and firewall counters.
- Document the root cause so the same symptom is easier next time.

### Appendix A.20 Cloud security group versus host firewall mismatch

- Start by identifying the lowest confirmed healthy layer.
- Capture the exact user symptom and timestamp.
- Compare behavior from a healthy source and an unhealthy source.
- Check DNS, route selection, and neighbor resolution before changing application settings.

## Appendix C — Packet Journey Mini-Stories

### Appendix C.1 Browser to HTTPS website

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.2 Laptop to SSH server

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.3 Host to local default gateway

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.4 Resolver to authoritative DNS server

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.5 Client to database over private subnet

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.6 VoIP phone sending UDP media

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.7 Kubernetes pod talking to a service VIP

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.8 VPN client reaching an internal API

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.9 Load balancer forwarding to backend node

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

### Appendix C.10 Server sending syslog to central collector

- Name resolution or service discovery identifies the target.
- The source host selects a route and next hop.
- Neighbor resolution provides the needed local Layer 2 destination.
- Transport builds the right socket conversation or datagram.
- IP forwards hop by hop until the destination network is reached.
- The destination stack decapsulates and hands data to the application.
- The response follows the reverse logic, though not always the exact same physical path.

## Appendix D — Checklist Before Changing Production Networking

- [ ] Confirm maintenance window and blast radius.
- [ ] Record current IP addresses, routes, neighbor entries, and listeners.
- [ ] Capture a known-good packet trace if possible.
- [ ] Know the rollback plan and console access path.
- [ ] Check DNS TTLs before moving public services.
- [ ] Validate host firewall and upstream policy alignment.
- [ ] Avoid overlapping subnets in VPN or hybrid designs.
- [ ] Verify MTU when overlays, tunnels, or MPLS paths are involved.
- [ ] Monitor retransmissions, drops, and application error rates during the change.
- [ ] Update diagrams and docs after the change succeeds.
