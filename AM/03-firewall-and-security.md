# Question 3: Firewall & Security Posture

A platform built on bare metal needs **layered security**, not a single “big firewall” answer. This chapter uses defense in depth across perimeter, internal segmentation, host controls, workload controls, and identity boundaries.

## Defense in Depth — Layered Security

1. **Hardware firewall appliance** for north-south control
2. **Switch or routed ACLs** for east-west segmentation between VLANs
3. **Host firewall** on every hypervisor and management VM
4. **Guest VM firewall** inside each workload
5. **Application-layer controls** such as WAF, auth, TLS, and rate limiting

## Defense in depth diagram

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Internet Perimeter Hardware Firewall] --> B[Inter-VLAN Controls ACLs Routing Policy]
    B --> C[Host Firewall firewalld nftables]
    C --> D[VM Firewall inside guest OS]
    D --> E[Application Security WAF Auth TLS RBAC]
~~~

## Security goals for this environment

- admin surfaces reachable only from approved management paths
- default deny between trust zones
- shared storage never exposed to internet-facing workloads
- every workload has a clear allowed-flow list
- safe staged rollout to avoid lockouts
- observability on security controls, not only on workloads

## Firewall Zone Design

| Zone | Networks | Allowed Inbound | Allowed Outbound |
|------|----------|-----------------|------------------|
| EXTERNAL | Internet | HTTP/HTTPS to DMZ only | None by default |
| DMZ | VLAN 50 | HTTP/HTTPS from external, selected internal admin access | To PROD on app-specific ports |
| PRODUCTION | VLAN 30 | From DMZ on app ports, from MGMT for SSH | To STORAGE, DNS, NTP, updates |
| STORAGE | VLAN 20 | From PROD and K8S for NFS/iSCSI only | None except return traffic |
| MANAGEMENT | VLAN 10 | From admin IPs only for SSH, API, UI | To all internal zones for administration |
| K8S | VLAN 60 | From DMZ for ingress, from MGMT for admin | To STORAGE, registry, DNS, NTP |

## Zone relationships

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    EXT[EXTERNAL] --> DMZ[DMZ]
    MGMT[MANAGEMENT] --> DMZ
    MGMT --> PROD
    MGMT --> K8S
    DMZ --> PROD
    PROD --> STOR[STORAGE]
    K8S --> STOR
    PROD --> OUT[Internet for updates via NAT]
    K8S --> OUT
~~~

## Hardware firewall configuration approach

The exact syntax differs across **pfSense**, **FortiGate**, **Palo Alto**, and other appliances, but the production pattern should stay consistent.

### Core rules

1. default deny all inbound between zones
2. explicit allow rules with documented owner and reason
3. NAT only where required
4. VPN required for admin access from untrusted networks
5. IDS/IPS enabled for internet-exposed zones
6. logging enabled on deny and high-value allow rules

### Example concept policy set

| Rule Order | Source | Destination | Ports | Action | Why |
|------------|--------|-------------|-------|--------|-----|
| 10 | Internet | DMZ LB | 80,443 | Allow | public web entry |
| 20 | Admin VPN | MGMT | 22,443,8006 | Allow | admin access |
| 30 | DMZ | PROD app tier | 8080,8443 | Allow | reverse proxy to apps |
| 40 | PROD | STORAGE | 2049,3260 | Allow | NFS/iSCSI |
| 50 | K8S | Registry/DNS/NTP | 443,53,123 | Allow | cluster ops |
| 999 | any | any | any | Deny + log | least privilege |

### NAT examples

- outbound source NAT for update access
- inbound port-forward from public VIP to DMZ reverse proxy
- no direct inbound NAT to PROD or STORAGE

## Host firewall with firewalld

Host firewall is the last internal control if network policy is misapplied.

### Why still use host firewalls on hypervisors

- protects the host even if a VLAN rule is wrong
- limits attack surface on management APIs and SSH
- provides local logging for incident review
- supports change-by-change testing before broader rollout

### firewalld zone mapping example

| Interface | Zone | Reason |
|-----------|------|--------|
| vmbr0.10 or vmbr0 | mgmt | hypervisor UI and SSH |
| vmbr1 | storage | NFS/iSCSI only |
| vmbr0.50 | dmz | if host terminates edge service or LB |

### Example commands

~~~bash
firewall-cmd --permanent --new-zone=mgmt
firewall-cmd --permanent --new-zone=storage
firewall-cmd --permanent --new-zone=dmz
firewall-cmd --permanent --zone=mgmt --add-source=10.10.10.0/24
firewall-cmd --permanent --zone=storage --add-source=10.10.20.0/24
firewall-cmd --permanent --zone=mgmt --add-service=ssh
firewall-cmd --permanent --zone=mgmt --add-port=8006/tcp
firewall-cmd --permanent --zone=storage --add-port=2049/tcp
firewall-cmd --permanent --zone=storage --add-port=3260/tcp
firewall-cmd --permanent --zone=public --set-target=DROP
firewall-cmd --reload
firewall-cmd --get-active-zones
~~~

### Rich rule example

~~~bash
firewall-cmd --permanent --zone=mgmt \
  --add-rich-rule='rule family="ipv4" source address="10.10.10.50/32" service name="ssh" accept'
~~~

## Advanced nftables example

`/etc/nftables.conf`

~~~bash
table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;
    ct state established,related accept
    iif "lo" accept
    ip saddr 10.10.10.0/24 tcp dport {22,8006} accept
    ip saddr 10.10.20.0/24 tcp dport {2049,3260} accept
    ip protocol icmp accept
    counter log prefix "nft-drop-input: "
  }

  chain forward {
    type filter hook forward priority 0; policy drop;
    ct state established,related accept
    counter log prefix "nft-drop-forward: "
  }

  chain output {
    type filter hook output priority 0; policy accept;
  }
}
~~~

## Staging & Validation Strategy — do not lock yourself out

1. Always have OOB/IPMI access as fallback.
2. Stage rules in log-only mode first.
3. Use timed auto-revert if testing a risky host rule set.
4. Test from a separate management host.
5. Roll to one host, verify, then expand.
6. Keep a break-glass management SSH rule.

## Safe firewall rollout workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Draft Rules in Git] --> B[Apply to Test Host]
    B --> C[Validate from Separate MGMT Host]
    C --> D[Observe Logs Counters and Access]
    D --> E[Promote to One Production Host]
    E --> F[Cluster-Wide Rollout via Automation]
    F --> G[Post-Change Verification]
~~~

## Timed rollback pattern

Use `at` or a scheduled revert before applying risky rules:

~~~bash
cp /etc/nftables.conf /root/nftables.conf.bak
printf 'cp /root/nftables.conf.bak /etc/nftables.conf && systemctl restart nftables\n' | at now + 5 minutes
nft -f /etc/nftables.conf
# validate from a second session
atq
atrm <jobid>
~~~

## SELinux / AppArmor

### Recommendation

Use **SELinux enforcing** on RHEL-family systems and keep AppArmor enabled on Ubuntu where that is the host standard.

### Why SELinux matters here

- limits lateral movement after service compromise
- protects service boundaries even with file permission mistakes
- makes least-privilege behavior measurable

### Commands

~~~bash
getenforce
sestatus
setenforce 1
ausearch -m avc -ts recent
sealert -a /var/log/audit/audit.log
cat /var/log/audit/audit.log | audit2why
~~~

### Common troubleshooting workflow

1. reproduce the denial
2. check audit log for AVCs
3. confirm whether label, boolean, or policy is wrong
4. fix context or boolean before considering custom policy
5. only create custom allow rules when truly necessary

## SSH hardening

### Baseline controls

- key-only authentication
- `PermitRootLogin no` or `prohibit-password`
- `AllowGroups ops-admins` or equivalent
- `PasswordAuthentication no`
- `MaxAuthTries 3`
- `AllowTcpForwarding no` unless specifically needed
- fail2ban or equivalent for exposed jump services

### SSHD example

`/etc/ssh/sshd_config.d/99-am.conf`

~~~bash
Protocol 2
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AllowGroups ops-admins
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding no
LoginGraceTime 30
MaxAuthTries 3
~~~

### Bastion / jump host pattern

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Admin[Admin Laptop] --> VPN[VPN]
    VPN --> Bastion[Bastion Host in MGMT]
    Bastion --> PVE[Hypervisors]
    Bastion --> VMS[Management VMs]
    Bastion --> K8S[Kubernetes Nodes]
~~~

### Example SSH client config

~~~bash
Host bastion
  HostName bastion.infra.example.com
  User ops
  IdentityFile ~/.ssh/am_ops_ed25519

Host pve01 pve02 pve03 *.infra.example.com
  User ops
  ProxyJump bastion
  IdentityFile ~/.ssh/am_ops_ed25519
~~~

## Additional security controls worth adding early

- MFA on VPN and bastion
- secrets stored in Vault or an enterprise password manager
- image scanning for container workloads
- auditd for privileged command tracking
- central syslog forwarding from switches, firewall, and hypervisors
- NTP locked to trusted internal hierarchy

## Verification checklist

| Control | Validation |
|---------|------------|
| Perimeter policy | external scan sees only intended DMZ services |
| Inter-VLAN policy | PROD cannot talk to MGMT except approved paths |
| Host firewall | only expected ports visible on each host |
| SELinux | `sestatus` shows enforcing |
| SSH hardening | password login rejected, key auth works through bastion |
| Logging | deny events visible in SIEM or log platform |

## Common mistakes

- relying only on the hardware firewall and skipping host controls
- allowing management interfaces from too many sources
- permitting storage access from the DMZ
- changing firewall rules without OOB access or rollback timer
- disabling SELinux permanently instead of fixing the policy issue

## Exit criteria

You are ready for [04-shared-storage.md](./04-shared-storage.md) and [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) when:

- perimeter and inter-VLAN policies are documented
- host firewalls are templated and tested
- bastion access works
- SELinux/AppArmor is enforced on chosen OS families
- rollback-safe change procedure is proven on at least one node


## Procurement & Cost Analysis

### Firewall appliance comparison

| Option | Example Model | Budgetary Range | Best Fit | Watch-outs |
|--------|---------------|-----------------|----------|------------|
| pfSense Plus / Netgate | 6100 / 8200 | $1K-$4K | budget-conscious teams with Linux/network skill | commercial support and advanced security ecosystem are lighter |
| Fortinet | FortiGate 100F / 200F | $3K-$9K + subscriptions | balanced enterprise branch/DC edge | renewals for UTM/IPS matter |
| Palo Alto | PA-440 / PA-450 | $5K-$12K + subscriptions | security-first environments needing premium visibility | higher CapEx and subscription costs |

### What to procure

- HA-capable firewall pair if downtime tolerance is low.
- VPN, IDS/IPS, and URL filtering subscriptions if Internet exposure is meaningful.
- Bastion host VM, MFA integration, password vault, and certificate management tooling.
- Vulnerability scanning tooling such as Nessus, Qualys, or OpenVAS depending on budget.
- External penetration test budget at least annually and after major exposure changes.

### Warranty, sourcing, licensing

- Buy through approved security resellers to align hardware, subscription, and support terms.
- Plan 3-year support minimum; 5 years is safer for slower on-prem refresh cycles.
- Free/open-source tiers lower cost, but enterprise audit/compliance often needs signed support contracts.
- Lead times can run 2-6 weeks for entry appliances and longer for HA bundles or premium subscriptions.

## Resource Planning

### Security capacity planning

| Dimension | Rule | Notes |
|-----------|------|-------|
| Firewall throughput | size for 2x expected peak with security services enabled | vendors quote higher numbers without IPS/TLS inspection |
| Concurrent sessions | model Internet-facing apps + east-west flows | add 30% headroom for incident spikes |
| Log retention | 90 days hot + 1 year cold for many audits | depends on SOC 2/PCI/HIPAA scope |
| Vulnerability scans | at least monthly infra scans, weekly Internet-facing scans | more often after critical CVEs |

### Compliance planning map

| Framework | Impact on This Layer |
|-----------|----------------------|
| CIS | host firewall, SSH hardening, auditd, secure services |
| NIST 800-53 | access control, logging, vulnerability management, incident response |
| SOC 2 | documented reviews, least privilege, change approval, evidence retention |
| PCI DSS | CDE segmentation, MFA, quarterly scans, tighter admin access |
| HIPAA | access logs, encryption, break-glass procedures, vendor BAAs where applicable |

### Staffing

- Minimum: 1 security-aware infra engineer plus shared network support.
- Production 24x7: security engineer, infra engineer, and on-call incident manager rotation.
- Scale out security tooling before scale-out of appliances if people are the bottleneck.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | defense-in-depth model | single control failures should not expose the estate | perimeter-only design rejected |
| ADR-02 | bastion + VPN admin path | centralizes logging and MFA | broad direct SSH exposure rejected |
| ADR-03 | default deny east-west | limits blast radius | allow-by-default rejected |
| ADR-04 | staged rollout with timed rollback | reduces lockout risk | direct production changes rejected |

### Integration points

- Network zones must match [02-network-design.md](./02-network-design.md).
- Host baseline and SELinux controls must align with [06-linux-os-layer.md](./06-linux-os-layer.md).
- Kubernetes RBAC, ingress, and network policies build on this layer in [09-kubernetes-deployment.md](./09-kubernetes-deployment.md).

## Planning & Timeline

### Security rollout timeline

| Week | Goal | Deliverables |
|------|------|--------------|
| Week 1 | policy design | zone matrix, allowed flows, admin path, logging destinations |
| Week 2 | staged implementation | HA firewall baseline, host firewall templates, bastion rollout |
| Week 3 | audit and testing | vulnerability scans, access review, external/internal validation |
| Week 4 | assurance | penetration test scoping, evidence pack, remediation backlog |

### Penetration testing plan

- Internal test after inter-VLAN rules and bastion controls are live.
- External test after DMZ services and VPN are exposed.
- Re-test after major network, firewall, or identity changes.
- Preserve test findings as inputs to runbooks and backlog grooming.

### Risk register and rollback

| Risk | Impact | Mitigation | Rollback |
|------|--------|------------|----------|
| lockout from management | admins lose access | OOB + timed revert + bastion test first | restore prior rules from console |
| over-broad allow rule | lateral movement risk | peer review and rule owner field | remove rule and audit logs |
| under-sized firewall with IPS on | packet loss / latency | size for security-service throughput | disable non-critical inspection temporarily |
| weak logging retention | lost evidence | forward logs centrally | export and snapshot logs immediately |

## Advanced Production Configurations

- Use HA firewall pairs with config sync and stateful failover where the platform cannot tolerate single-appliance downtime.
- Treat TLS interception as an exception requiring legal, privacy, and performance review; do not enable by default for east-west traffic.
- For DR, replicate policy objects, certificates, VPN settings, and admin accounts to the secondary site.
- Capacity alerts: session table >80%, CPU >70% sustained with IPS enabled, VPN failure, log disk >75%, IDS signature update failures.
- Auto-remediation can disable a noisy deny-log rule or rotate traffic to the standby firewall, but policy changes still need change control.
- Keep evidence for compliance by exporting rulebases, access reviews, vulnerability scan results, and exception approvals.

## Building & Deployment Runbook

1. Define the zone matrix, approved flows, log destinations, and admin path.
2. Install or rack the firewall appliance(s), upgrade to the approved firmware, and back up the baseline config.
3. Configure interfaces/VLANs, default deny policy, VPN, and bastion reachability.
4. Apply host-level controls on one test host first:
   ~~~bash
   firewall-cmd --reload
   firewall-cmd --list-all --zone=mgmt
   sestatus
   ssh -J bastion ops@pve01
   ~~~
5. Run internal and external validation scans, then review deny logs before broad rollout.
6. Schedule a timed revert before risky host firewall changes:
   ~~~bash
   cp /etc/nftables.conf /root/nftables.conf.prechange
   printf '%s\n' 'cp /root/nftables.conf.prechange /etc/nftables.conf && systemctl restart nftables' | at now + 5 minutes
   nft -f /etc/nftables.conf
   ~~~
7. Validation gate: confirm admin access, expected app flows, denied unexpected paths, and log visibility in SIEM.
8. Freeze the rule set in Git or config backup before onboarding production tenants.

### Common mistakes to avoid during build

- Comparing vendor throughput numbers without security services enabled.
- Running a pentest before the environment is documented and then losing the findings in email.
- Enabling many subscriptions/features without staffing the operational reviews they create.
- Treating compliance as paperwork instead of a design constraint.


## Cross-references

- Network segmentation: [02-network-design.md](./02-network-design.md)
- OS hardening details: [06-linux-os-layer.md](./06-linux-os-layer.md)
- K8s policy later: [09-kubernetes-deployment.md](./09-kubernetes-deployment.md)
- Related repo reference: [../Physical-Setup/02-os-installation-and-hardening.md](../Physical-Setup/02-os-installation-and-hardening.md)
