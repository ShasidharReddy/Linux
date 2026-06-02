# Network Tasks

This guide covers connectivity, DNS, routing, VPN, and bandwidth-related operational tasks.

## 9.1 Checking connectivity

### Ping

```bash
ping -c 4 8.8.8.8
ping -c 4 example.com
```

### TCP connectivity

```bash
nc -vz example.com 443
curl -I https://example.com
```

### Route and interface overview

```bash
ip -br a
ip route
ip rule
```

## 9.2 DNS troubleshooting

```bash
dig example.com
```

```bash
dig +short example.com
```

```bash
dig @8.8.8.8 example.com
```

```bash
host example.com
nslookup example.com
```

### Reverse lookup

```bash
dig -x 8.8.8.8 +short
```

### Resolver status

```bash
resolvectl status || cat /etc/resolv.conf
```

## 9.3 Bandwidth monitoring

```bash
sar -n DEV 1 5
ip -s link
nload || true
iftop -nP || true
```

## 9.4 Port forwarding

### Temporary local forward

```bash
ssh -L 8080:127.0.0.1:80 user@server01.example.com
```

### Remote forward

```bash
ssh -R 2222:127.0.0.1:22 jumpbox.example.com
```

### Dynamic SOCKS proxy

```bash
ssh -D 1080 user@server01.example.com
```

## 9.5 VPN connection management

### OpenVPN example

```bash
sudo systemctl status openvpn-client@corp --no-pager
sudo systemctl restart openvpn-client@corp
```

### WireGuard example

```bash
sudo wg show
sudo systemctl restart wg-quick@wg0
ip a show wg0
```

## 9.6 Network interface management

### Bring interface up or down

```bash
sudo ip link set eth0 down
sudo ip link set eth0 up
```

### Add temporary IP

```bash
sudo ip addr add 192.168.1.10/24 dev eth0
```

### Remove temporary IP

```bash
sudo ip addr del 192.168.1.10/24 dev eth0
```

### Check ARP or neighbor table

```bash
ip neigh
```

## 9.7 Troubleshooting ports and sockets

```bash
ss -s
ss -antp | head -50
ss -lntp
```

### Identify process using a port

```bash
sudo lsof -i :8080
sudo fuser -n tcp 8080
```

## 9.8 Packet capture basics

```bash
sudo tcpdump -i eth0 -nn host 192.168.1.10 and port 443
```

```bash
sudo tcpdump -i any -nn 'tcp port 53 or udp port 53'
```

## 9.9 MTU and route diagnostics

```bash
ip link show eth0
tracepath example.com
traceroute example.com
```

## 9.10 Network task one-liners

```bash
for h in 8.8.8.8 1.1.1.1 example.com; do echo "== $h =="; ping -c 2 "$h"; done
```

```bash
ss -ant state established '( sport = :443 or dport = :443 )' | awk 'NR>1 {print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head
```

---
