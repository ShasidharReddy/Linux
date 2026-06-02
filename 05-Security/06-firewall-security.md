# Firewall Security

Firewalling is central to Linux host and network defense.

A hardened host should expose only the minimum inbound services and tightly control outbound traffic where business requirements permit.

### 6.1 Host Firewall Philosophy

A host firewall is valuable even when a network firewall already exists.

Why:

- It protects the system if network controls fail.
- It reduces lateral movement inside trusted networks.
- It documents intended exposure close to the workload.
- It helps enforce environment-specific rules during cloud or hybrid operations.

### 6.2 iptables and nftables

Linux historically used `iptables`.

Modern distributions increasingly use `nftables` directly or through compatibility layers.

High-level comparison:

| Tool | Notes |
| --- | --- |
| iptables | Mature, widely documented, legacy syntax on many systems |
| nftables | Newer framework, improved set handling, cleaner rule model |
| firewalld | Higher-level management layer that may use nftables backend |

Check active tooling carefully.

### 6.3 Baseline Inspection

Commands:

```bash
sudo iptables -L -n -v
sudo iptables -S
sudo nft list ruleset
sudo firewall-cmd --state
sudo firewall-cmd --list-all
ss -tulpn
```

Questions:

- Which ports are listening?
- Which rules explicitly allow them?
- Is default policy restrictive?
- Are rules consistent with current services?

### 6.4 iptables Deep Dive

Important chains:

- INPUT
- OUTPUT
- FORWARD

Common policy strategy:

- default drop inbound
- allow loopback
- allow established and related traffic
- allow required service ports
- log limited denied traffic

Example concept:

```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix 'iptables-deny: '
```

Concepts to know:

- order matters
- first matching rule usually wins in practice for chain traversal logic
- broad allow rules early can defeat later restrictions
- logging without rate limits can flood disks

### 6.5 nftables Deep Dive

`nftables` uses tables, chains, sets, and expressions.

Example minimal concept:

```nft
table inet filter {
  chain input {
    type filter hook input priority 0;
    policy drop;
    iif lo accept
    ct state established,related accept
    tcp dport 22 ip saddr 10.0.0.0/24 accept
    tcp dport 443 accept
  }
}
```

Why nftables is powerful:

- one syntax for IPv4 and IPv6 with `inet`
- efficient sets and maps
- cleaner atomic ruleset replacement
- easier large policy management in many cases

### 6.6 Connection Tracking

Connection tracking helps firewalls understand session state.

Common states:

- NEW
- ESTABLISHED
- RELATED
- INVALID

Typical policy:

- Allow `ESTABLISHED,RELATED`.
- Scrutinize or drop `INVALID`.
- Allow only intended `NEW` inbound connections.

Example:

```bash
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

### 6.7 firewalld Zones

`firewalld` provides higher-level policy management using zones and services.

Common zones include:

- drop
- block
- public
- external
- internal
- trusted

Inspect configuration:

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --list-all
```

Example operations:

```bash
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="10.0.0.0/24" service name="ssh" accept'
sudo firewall-cmd --reload
```

Guidance:

- Use the most restrictive zone that fits the interface.
- Avoid marking interfaces as `trusted` unless there is a strong reason.
- Review zone-to-interface mapping after network changes.

### 6.8 Rate Limiting

Rate limiting reduces brute-force and resource abuse impact.

Examples:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --update --seconds 60 --hitcount 6 -j DROP
```

Possible use cases:

- SSH brute-force reduction.
- Basic protection for public APIs.
- Log flood reduction.

Limitations:

- Not a full DDoS solution.
- Needs careful tuning for NAT-heavy clients.

### 6.9 Port Knocking

Port knocking hides service exposure until a client performs a specific sequence of connection attempts.

Benefits:

- obscurity layer for sensitive administrative services

Risks:

- operational complexity
- brittle troubleshooting
- false confidence if used without strong auth and MFA

Use port knocking only as a supplemental control.

Never as the main security boundary.

### 6.10 Logging Strategy

Firewall logs are useful for:

- attack detection
- troubleshooting exposure
- validating denied traffic
- incident investigation

Best practices:

- rate limit logs
- use clear prefixes
- ship logs centrally
- distinguish noisy scans from targeted probes

### 6.11 IPv6 Considerations

Security teams sometimes harden IPv4 and forget IPv6.

That is a serious gap.

Ensure equivalent policy exists for:

- inbound filtering
- outbound filtering if enforced
- management access
- service exposure

### 6.12 Outbound Filtering

Most environments focus on inbound traffic, but outbound control matters too.

Benefits:

- limits malware callouts
- reduces data exfiltration paths
- enforces approved service dependencies

Examples:

- Only app servers can reach database ports.
- Only update infrastructure can reach package mirrors.
- Build systems can access artifact repositories but not arbitrary internet destinations.

### 6.13 Hardening Patterns by Host Type

| Host Type | Typical Exposure Pattern |
| --- | --- |
| Public web server | 80 or 443 inbound, restricted admin SSH, controlled egress |
| Database server | No public inbound, app-tier-only access, limited admin sources |
| Bastion host | SSH inbound from approved ranges, strong MFA, heavy logging |
| CI runner | Restricted outbound to artifact sources, minimal inbound |

### 6.14 Testing Firewall Rules Safely

Before making persistent changes:

- maintain an out-of-band console path
- test from a second session
- stage rules in a rollback-friendly manner
- document current working rules

Useful commands:

```bash
nc -zv host 22
curl -I https://host
nmap -Pn host
```

### 6.15 Common Firewall Mistakes

- Accepting `0.0.0.0/0` for SSH.
- Allowing any-any traffic during troubleshooting and forgetting to remove it.
- Logging every dropped packet without limits.
- Ignoring outbound policy.
- Forgetting IPv6.
- Failing to save or persist rules.

### 6.16 Summary

A Linux firewall should be explicit, minimal, state-aware, and observable.

Whether you use iptables, nftables, or firewalld, the goal is the same: allow only what is required and make the rest fail closed.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.6 Firewall Checklist

- Confirm default deny or equivalent restrictive inbound posture.
- Review rules against current listening ports.
- Review outbound filtering requirements.
- Review IPv6 policy parity.
- Review management source restrictions.
- Review logging rate limits.
- Review temporary troubleshooting rules.
- Review zone assignments in firewalld.
- Confirm persistence after reboot.
- Test from an external validation host.

### B.6 Firewall Commands

```bash
iptables -L -n -v
iptables -S
nft list ruleset
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
ss -tulpn
nc -zv host 22
nmap -Pn host
```

### C.6 Firewalls

66. Why is a host firewall still useful when there is a network firewall?
67. What is the role of connection tracking?
68. What do `ESTABLISHED` and `RELATED` mean conceptually?
69. Why is default deny a common best practice?
70. What risk comes from broad allow rules early in a chain?
71. Why must IPv6 policy be reviewed separately?
72. Why are firewall logs useful?
73. Why should firewall logs be rate-limited?
74. What is port knocking?
75. Why should port knocking only be a supplemental control?
76. Why might outbound filtering help contain compromise?
77. Why should bastion hosts have narrower rules than general servers?
78. Why must rule persistence be tested?
79. Why is safe rollback important during firewall changes?
80. Why should listening ports and firewall rules be reviewed together?
