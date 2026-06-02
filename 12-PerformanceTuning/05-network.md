# Network Performance

[Back to guide index](README.md)

Network performance is about:

- bandwidth
- latency
- packet rate
- drops
- retransmits
- CPU overhead
- queueing

## 5.1 Bandwidth vs latency

Bandwidth is capacity.

Latency is delay.

A system may have excellent bandwidth but poor p99 latency.

## 5.2 Network metrics

- bits/sec
- packets/sec
- RTT
- retransmits
- drops
- errors
- backlog occupancy
- connection rate
- established connections

## 5.3 Core commands

### 5.3.1 `ip` and `ss`

```bash
ip -s link
ss -s
ss -ti
ss -lnt
```

### 5.3.2 `sar`

```bash
sar -n DEV 1 5
sar -n TCP,ETCP 1 5
```

### 5.3.3 `ethtool`

```bash
ethtool eth0
ethtool -k eth0
ethtool -S eth0
ethtool -g eth0
```

### 5.3.4 `nstat`

```bash
nstat -az
```

## 5.4 TCP essentials

Key concepts:

- congestion window
- receive window
- send buffer
- RTT
- slow start
- retransmission timeout
- SACK

## 5.5 Bandwidth-delay product

Formula:

`BDP = bandwidth * RTT`

If buffers are too small for BDP, throughput is capped.

## 5.6 Congestion control

Common algorithms:

- `cubic`
- `bbr`
- `reno`

Check:

```bash
sysctl net.ipv4.tcp_congestion_control
sysctl net.ipv4.tcp_available_congestion_control
```

Set example:

```bash
sysctl -w net.ipv4.tcp_congestion_control=bbr
```

## 5.7 Socket buffers

Relevant settings:

- `net.core.rmem_max`
- `net.core.wmem_max`
- `net.ipv4.tcp_rmem`
- `net.ipv4.tcp_wmem`
- `net.core.netdev_max_backlog`

Too small:

- poor throughput on high RTT paths

Too large:

- risk of bufferbloat

## 5.8 Listen backlogs

Relevant items:

- application backlog parameter
- `net.core.somaxconn`
- `net.ipv4.tcp_max_syn_backlog`

## 5.9 MTU tuning

Default often:

- 1500

Jumbo example:

- 9000

Check:

```bash
ip link show dev eth0
```

Set:

```bash
ip link set dev eth0 mtu 9000
```

Validate end to end.

## 5.10 NIC offloads

Examples:

- checksum offload
- TSO
- GSO
- GRO
- LRO

Check:

```bash
ethtool -k eth0
```

## 5.11 RSS, RPS, RFS, XPS

### 5.11.1 RSS

Hardware receive queue scaling.

### 5.11.2 RPS

Software receive packet steering.

### 5.11.3 RFS

Tries to steer packets toward CPUs running the consuming app.

### 5.11.4 XPS

Transmit queue steering.

These are important for multicore throughput and latency.

## 5.12 Interrupt coalescing

Trade-off:

- fewer interrupts
- more batching
- potentially more latency

Check and set:

```bash
ethtool -c eth0
ethtool -C eth0 rx-usecs 25
```

## 5.13 Ring buffers

Check:

```bash
ethtool -g eth0
```

Set example:

```bash
ethtool -G eth0 rx 4096 tx 4096
```

Larger rings may reduce drops.

They may also increase latency.

## 5.14 `iperf3`

Server:

```bash
iperf3 -s
```

Client:

```bash
iperf3 -c server.example.com -P 4 -t 30
```

Reverse:

```bash
iperf3 -c server.example.com -R -t 30
```

UDP:

```bash
iperf3 -c server.example.com -u -b 1G -t 30
```

## 5.15 Packet tools

Useful tools:

- `ping`
- `mtr`
- `traceroute`
- `tcpdump`
- `dropwatch`
- `bpftrace`

## 5.16 Retransmit diagnosis

Use:

```bash
ss -ti
sar -n TCP,ETCP 1 5
nstat -az | egrep 'Retrans|InErrs|OutSegs'
```

Possible causes:

- packet loss
- congestion
- overloaded receiver
- NIC issues
- path issues

## 5.17 Conntrack

Check:

```bash
sysctl net.netfilter.nf_conntrack_max
cat /proc/sys/net/netfilter/nf_conntrack_count
```

High conntrack pressure can hurt performance.

## 5.18 Container networking

Extra layers may include:

- bridge
- veth
- overlay network
- NAT
- sidecar proxy
- service mesh

Each layer adds work and queueing.

## 5.19 Network bottleneck patterns

### Pattern A

High bandwidth but one hot CPU.

Likely causes:

- small packets
- poor RSS balance
- disabled offloads

### Pattern B

High retransmits.

Likely causes:

- packet loss
- congestion
- receiver bottleneck

### Pattern C

Connection failures under burst.

Likely causes:

- backlog exhaustion
- FD exhaustion
- SYN queue saturation

### Pattern D

Good averages and bad p99.

Likely causes:

- burst queueing
- interrupt imbalance
- bufferbloat
- app pauses

## 5.20 Network tuning practices

- inspect pps as well as bps
- benchmark congestion-control changes
- verify MTU end to end
- tune RSS or RPS if one CPU is hot
- monitor drops and retransmits continuously
- validate ring-size and coalescing changes

## 5.21 Network quick commands

```bash
ip -s link
ss -s
ss -ti
sar -n DEV 1 5
sar -n TCP,ETCP 1 5
nstat -az
ethtool -S eth0
ethtool -k eth0
ethtool -g eth0
iperf3 -c server -P 4 -t 30
ping -c 10 server
mtr -rw server
```

---
