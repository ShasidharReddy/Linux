# Network Security

Network security on Linux extends beyond simple firewall rules.

It includes exposure management, service trust boundaries, VPN architecture, anti-scan posture, and safe protocol usage.

### 11.1 Service Exposure Review

Start with the basics:

```bash
ss -tulpn
sudo nmap -sV -Pn localhost
ip addr
ip route
```

Questions:

- Which interfaces are public?
- Which services bind to all addresses?
- Which services should bind only to loopback or a private subnet?

### 11.2 TCP Wrappers

TCP wrappers are legacy but may still appear on older systems and services.

They historically used:

- `/etc/hosts.allow`
- `/etc/hosts.deny`

Example concept:

```text
sshd: 10.0.0.0/255.255.255.0
ALL: ALL
```

Important note:

Many modern services do not rely on TCP wrappers.

Use firewalls and service-native access controls as primary defenses.

### 11.3 Port Scanning Defense

You cannot always stop scanning, but you can reduce useful information leakage.

Methods:

- close unused ports
- use stateful firewalls
- rate limit responses where appropriate
- avoid unnecessary service banners
- monitor scan patterns

### 11.4 Banner Reduction

Information disclosure from banners helps attackers fingerprint systems.

Examples:

- hide exact software versions where possible
- reduce verbose error pages
- avoid disclosing internal hostnames in public responses

### 11.5 DDoS Mitigation Concepts

Host-level controls are limited against large DDoS events.

Still, Linux hosts can help with:

- connection limits
- rate limiting
- SYN flood protections
- upstream proxy or load balancer integration
- tight service timeouts

Kernel/network tuning examples may include:

- SYN cookies
- backlog sizing
- connection timeout review

Always coordinate DDoS strategy with upstream network providers or cloud controls.

### 11.6 VPN Security

VPNs secure remote connectivity and can segment administration traffic away from public networks.

Common VPN options:

- WireGuard
- OpenVPN
- IPsec

Security goals:

- encrypt admin traffic
- reduce public exposure of sensitive services
- require stronger identity verification
- simplify source-based firewalling

### 11.7 Certificate Pinning

Certificate pinning attempts to restrict trust to expected certificates or keys.

It can reduce some CA misuse risks but introduces operational complexity.

Considerations:

- key rotation planning is critical
- stale pins can cause outages
- modern web ecosystems often prefer alternatives such as strong PKI management and certificate transparency monitoring

### 11.8 DNS Security Considerations

Review:

- trusted resolvers
- DNS over TLS or DNS over HTTPS policy where relevant
- split-horizon DNS exposure
- cache poisoning risks
- restrictive zone transfers for authoritative servers

### 11.9 Reverse Proxy Security

For internet-facing services:

- terminate TLS safely
- enforce secure headers where relevant
- limit methods and request sizes
- log client IPs correctly
- patch proxy software promptly

### 11.10 Segmentation

Strong network security uses segmentation.

Examples:

- web tier can talk to app tier only on required ports
- app tier can talk to database tier only on required ports
- admin access originates from bastion or VPN only
- backup traffic uses dedicated paths where possible

### 11.11 Outbound Network Review

Outbound traffic often reveals compromise.

Review for:

- rare destinations
- newly observed geographies
- connections from services that normally do not initiate outbound sessions
- cleartext transfers of sensitive data

### 11.12 Network Security Checklist

- minimize listening services
- bind services to the right interfaces
- filter inbound and outbound traffic
- use VPN for administration
- suppress unnecessary banners
- monitor scanning and anomalies
- coordinate DDoS protections with upstream controls
- segment workloads by role

### 11.13 Summary

Linux network security is strongest when exposure is minimal, trust boundaries are explicit, and traffic behavior is monitored continuously.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.11 Network Security Checklist

- Review listening ports.
- Review service bind addresses.
- Review segmentation rules.
- Review VPN access scope.
- Review exposed service banners.
- Review DDoS protections with upstream providers.
- Review DNS trust and resolver settings.
- Review reverse proxy hardening.
- Review outbound anomaly monitoring.
- Review scanning telemetry.

### B.11 Network Commands

```bash
ip addr
ip route
resolvectl status
ss -s
curl -I https://example.com
openssl s_client -connect example.com:443 -servername example.com
```

### C.11 Network Security

121. Why should services bind only to required interfaces?
122. What are TCP wrappers, and why are they mostly legacy?
123. Why does reducing service banners matter?
124. Why is DDoS defense often an upstream responsibility as well as a host responsibility?
125. Why are VPNs useful for admin access?
126. What problem does segmentation solve?
127. Why is outbound anomaly monitoring valuable?
128. Why should DNS settings be reviewed in security assessments?
129. What is certificate pinning, and what operational risk does it create?
130. Why should reverse proxies be hardened like any other exposed service?
