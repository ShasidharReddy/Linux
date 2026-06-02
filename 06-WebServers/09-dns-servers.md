# DNS Servers

## 9.1 Overview

DNS translates names to data such as IPs and service records.

Common server software:

- BIND9
- PowerDNS
- Knot DNS
- Unbound for recursive resolving

## 9.2 DNS Basics

Important concepts:

- Authoritative DNS
- Recursive resolver
- Zone
- Record types
- TTL
- Delegation

## 9.3 Common Record Types

| Record | Purpose |
|---|---|
| A | IPv4 address |
| AAAA | IPv6 address |
| CNAME | Canonical alias |
| MX | Mail exchanger |
| TXT | Free-form text, SPF, verification |
| NS | Name server delegation |
| SRV | Service locator |
| PTR | Reverse mapping |
| SOA | Zone authority data |
| CAA | Certificate authority authorization |

## 9.4 BIND9 Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y bind9 bind9utils bind9-doc dnsutils
sudo systemctl enable --now bind9
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y bind bind-utils
sudo systemctl enable --now named
```

## 9.5 Key BIND Files

| Purpose | Debian/Ubuntu | RHEL Family |
|---|---|---|
| Main config | `/etc/bind/named.conf` | `/etc/named.conf` |
| Zone files | `/etc/bind/` or `/var/cache/bind/` | `/var/named/` |
| Logs | journal/syslog | journal/messages |

## 9.6 Basic named.conf Example

```conf
options {
    directory "/var/cache/bind";
    recursion no;
    allow-query { any; };
    listen-on { any; };
    listen-on-v6 { any; };
    dnssec-validation auto;
};

zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
```

## 9.7 Zone File Example

```dns
$TTL 3600
@   IN  SOA ns1.example.com. admin.example.com. (
        2025010101
        3600
        900
        604800
        86400 )

    IN  NS      ns1.example.com.
    IN  NS      ns2.example.com.

ns1 IN  A       203.0.113.10
ns2 IN  A       203.0.113.11
@   IN  A       203.0.113.20
www IN  CNAME   @
mail IN  A       203.0.113.30
@   IN  MX 10    mail.example.com.
@   IN  TXT      "v=spf1 mx -all"
_sip._tcp IN SRV 10 60 5060 sip.example.com.
```

## 9.8 SOA Fields Explained

- Serial
- Refresh
- Retry
- Expire
- Minimum/negative TTL

Serial best practice:

- Increment on every zone change
- Common format: `YYYYMMDDNN`

## 9.9 Validation Commands

```bash
named-checkconf
named-checkzone example.com /etc/bind/db.example.com
```

## 9.10 Query Testing

```bash
dig @127.0.0.1 example.com A
dig @127.0.0.1 www.example.com CNAME
dig @127.0.0.1 example.com MX
host example.com 127.0.0.1
```

## 9.11 Reverse DNS Zone Example

```dns
$TTL 3600
@   IN  SOA ns1.example.com. admin.example.com. (
        2025010101
        3600
        900
        604800
        86400 )

    IN  NS ns1.example.com.
20  IN  PTR example.com.
30  IN  PTR mail.example.com.
```

## 9.12 Split-Horizon DNS

Split-horizon DNS serves different answers to internal and external clients.

Use cases:

- Internal service addresses
- Private admin panels
- VPN-only resources

Concept:

- Internal view returns private IPs
- External view returns public IPs

### BIND View Example

```conf
acl internal_nets { 10.0.0.0/8; 192.168.0.0/16; };

view "internal" {
    match-clients { internal_nets; };
    recursion yes;
    zone "example.com" {
        type master;
        file "/etc/bind/db.example.com.internal";
    };
};

view "external" {
    match-clients { any; };
    recursion no;
    zone "example.com" {
        type master;
        file "/etc/bind/db.example.com.external";
    };
};
```

## 9.13 DNS Security Basics

- Restrict zone transfers
- Disable recursion on public authoritative servers
- Use TSIG for transfers when appropriate
- Patch BIND regularly
- Limit query access where needed
- Use DNSSEC if required and supported operationally

### Restrict Zone Transfers Example

```conf
allow-transfer { 203.0.113.11; };
```

## 9.14 Secondary DNS

Having at least two authoritative name servers improves resilience.

Typical pattern:

- Primary/master source of truth
- Secondary/slave receives zone transfer

## 9.15 DNS Troubleshooting Tips

Check:

- Zone serial increments
- NS glue records
- TTL behavior
- Propagation expectations
- Firewall allowing TCP and UDP 53
- SOA and MX correctness

## 9.16 Common DNS Commands

```bash
dig example.com ANY
# Some servers restrict ANY queries

dig +short example.com A
dig +trace example.com
rndc reload
systemctl status bind9
systemctl status named
```

## 9.17 DNS Best Practices Summary

- Keep authoritative and recursive roles separate
- Use at least two authoritative servers
- Validate every zone change
- Document TTL choices
- Protect transfers and recursion

---

### 12.8 DNS Checklist

- SOA serial incremented
- Zone validates
- NS records correct
- Glue records correct if needed
- TTL reasonable
- Public authoritative server recursion disabled

### 13.6 DNS Troubleshooting

```bash
dig example.com A
dig @ns1.example.com example.com MX
dig +trace example.com
```

Look for:

- Wrong zone serial
- Missing records
- Delegation issues
- Firewall blocking TCP/UDP 53

### 19.9 DNS Reinforcement

- Authoritative and recursive roles should usually be separated.
- TTL affects propagation speed and cache load.
- Zone serial increments must be consistent.
- Always validate zone files before reload.
- Split-horizon DNS can simplify internal naming but increases complexity.

### 20.4 DNS Commands

```bash
dig example.com A
host example.com
nslookup example.com
rndc reload
```

### 22.4 Validate DNS

```bash
dig @ns1.example.com example.com SOA
dig @ns1.example.com example.com MX
```

### 24.11 BIND Zone Transfer Restriction Example

```conf
allow-transfer { 203.0.113.11; 203.0.113.12; };
```
