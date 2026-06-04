<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│  Infrastructure Deployment — High-Level Design (HLD)        │
│  Bare Metal → Virtualization → Containers → Kubernetes      │
└──────────────────────────────────────────────────────────────┘
</pre></div>

# Infrastructure Deployment — High-Level Design (HLD)

## Scenario

- Bare-metal servers, managed switches, hardware firewall, and shared storage already procured or planned.
- Deliver a production-ready platform for VMs, standalone containers, and Kubernetes on the same on-prem foundation.
- Optimize for **Security, HA, Automation, Observability, Cost, and Simplicity**.
- Keep management, storage, workload, and out-of-band traffic isolated from day one.
- Prefer open-source defaults unless support contracts or compliance needs justify paid tiers.
- Build bottom-up and verify each layer before promoting workloads.

## Design Priorities

| Priority | HLD Decision |
|----------|--------------|
| Security | Default deny, segmented VLANs, bastion-only admin access, SELinux/AppArmor, image scanning |
| High Availability | 3-node quorum, dual switches, bonded uplinks, shared storage, HA control plane |
| Automation | Packer + Terraform + Ansible + Git-driven change control |
| Observability | Prometheus, Grafana, Alertmanager, EFK/OpenSearch, synthetic checks |
| Cost | Proxmox, kubeadm, Harbor, Prometheus, and Ansible as primary stack |
| Simplicity | Start with NFS + iSCSI, stacked etcd, Podman for standalone containers |

## Assumptions & Constraints

- One primary site; DR is planned but not the first implementation milestone.
- Internet-exposed services terminate in the DMZ; management never does.
- All critical hosts have OOB access, DNS, NTP, and backup power.
- Shared storage exists before HA VM placement or Kubernetes stateful workloads.
- Automation pipelines are mandatory for repeatable VM and node creation.
- Open-source defaults are acceptable unless compliance or support risk says otherwise.

## Topology Snapshot

| Layer | Minimum HLD Choice |
|------|---------------------|
| Hypervisors | 3 Proxmox nodes |
| Network | 2 managed 10/25 GbE switches + separate OOB path |
| Firewall | 1 HA-capable appliance or pair |
| Storage | 1 shared NFS/iSCSI platform with multipath |
| Utility Services | Bastion, DNS, NTP, registry, monitoring |
| Kubernetes | 3 control planes, 3-5 workers, 2 LB VMs |

## Build Order

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Physical Layer\nRack Power Cabling Firmware] --> B[Network Layer\nVLANs Switches Firewall]
    B --> C[Hypervisor Layer\nKVM Proxmox Cluster]
    C --> D[Shared Storage\nNFS iSCSI Multipath]
    D --> E[VM Provisioning\nGolden Images Terraform]
    E --> F[OS Hardening\nCIS SELinux firewalld]
    F --> G[Container Runtime\nPodman containerd]
    G --> H[Kubernetes Cluster\nHA Control Plane]
    H --> I[Monitoring and Observability]
    I --> J[Day-2 Operations\nBackup Patch Failover Incident Response]
~~~

## Q1: Hypervisor Layer

- **Choice**: Proxmox VE (KVM-based).
- **Why**: No per-CPU license, built-in clustering/HA, strong web UI + API, ZFS/Ceph optionality.
- **Cluster**: Minimum 3 nodes for quorum; 5 nodes if workload density or maintenance windows justify it.
- **Fencing**: IPMI/iDRAC/iLO-based fencing is mandatory before enabling HA.
- **Storage**: Local ZFS mirror for host OS; shared NFS/iSCSI for movable workloads.
- **Ops rule**: Keep one node with enough spare capacity to absorb a failed peer.
- **Verification**: `pvecm status`, `pvesm status`, BMC reachability, NTP sync, live migration test.
- → [Detailed Guide](./01-hypervisor-layer.md)

## Q2: Network Design

- **VLANs**: MGMT(10), Storage(20), VM-Prod(30), VM-Dev(40), DMZ(50), K8S(60), OOB(99).
- **Bonding**: LACP (802.3ad) with 4 NICs minimum per hypervisor; separate bond for storage preferred.
- **Switching**: Dual managed switches with stack/MLAG/VPC equivalent.
- **Addressing**: Reserve low IPs for gateways, VIPs, controllers, and hypervisors; keep DNS + PTR complete.
- **Automation**: Ansible or vendor-native automation for switch configs and host networking.
- **Performance**: MTU 9000 only on the full storage path; standard MTU elsewhere unless required.
- **Verification**: `iperf3`, bond failover test, `ping -M do -s 8972`, trunk/VLAN validation.
- → [Detailed Guide](./02-network-design.md)

## Q3: Firewall & Security

- **Layers**: Hardware firewall → inter-VLAN ACLs → host firewalld/nftables → guest controls → app/WAF.
- **Zones**: External, DMZ, Production, Storage, Management, Kubernetes, OOB.
- **Policy**: Default deny; every allow rule needs owner, reason, review date, and logging decision.
- **Admin path**: VPN + bastion + MFA; no direct Internet exposure for MGMT, storage, or hypervisor APIs.
- **Safety**: Stage in log-only mode, use timed rollback, and keep OOB access active during changes.
- **Compliance baseline**: CIS hardening, audit logging, privileged access control, vulnerability scans.
- **Verification**: External scan, east-west rule validation, SELinux enforcing, bastion-only SSH success.
- → [Detailed Guide](./03-firewall-and-security.md)

## Q4: Shared Storage

- **Why before VMs**: HA, live migration, and consistent backup workflows depend on shared disks.
- **Protocols**: NFS for templates/general VM disks; iSCSI for databases and latency-sensitive block workloads.
- **Resilience**: Dual controllers, dual switches, multipath I/O, RAID aligned to workload profile.
- **Network**: Dedicated storage VLAN 20; jumbo frames only if every hop supports them.
- **Capacity model**: Size for usable TB, snapshots, replicas, and 30-40% free headroom.
- **Performance rule**: Benchmark random read/write and migration traffic before onboarding production.
- **Verification**: `fio`, `multipath -ll`, `showmount -e`, `iscsiadm -m session`, live migration under load.
- → [Detailed Guide](./04-shared-storage.md)

## Q5: VM Provisioning & Hardening

- **Factory pattern**: Packer golden image → Terraform VM creation → Ansible post-config.
- **Golden image includes**: Base OS, qemu guest agent, chrony, node_exporter, SELinux, auditd, CIS L1 baseline.
- **Per-VM via cloud-init**: Hostname, IP/DNS, SSH keys, app packages, environment-specific config.
- **Hardening**: Separate `/var/log`, PAM lockout, disabled unused services, centralized logging, image versioning.
- **Governance**: Template promotion requires smoke test + security scan + change record.
- **Scale rule**: Rebuild images on schedule; do not let long-lived snowflake VMs drift indefinitely.
- **Verification**: `cloud-init status --wait`, SSH via bastion, `qm agent <vmid> ping`, oscap/agent checks.
- → [Detailed Guide](./05-vm-provisioning-and-hardening.md)

## Q6: Linux OS Layer

- **Service model**: systemd units, dependencies, restart policy, resource limits, journal integration.
- **Packages**: Local mirror or approved repos for reproducibility, patch control, and offline resilience.
- **Filesystem**: LVM with separated `/`, `/var`, `/var/log`, `/tmp`, `/opt`; XFS default for larger estates.
- **Security**: SELinux enforcing, sudo least privilege, PAM faillock, audited privileged actions.
- **Repeatability**: Ansible as source of truth; `etckeeper` or Git-backed config history for `/etc`.
- **Lifecycle**: Patch non-prod first, snapshot or backup before change, then promote in waves.
- **Verification**: `systemctl`, `sestatus`, `firewall-cmd --state`, repo mirror availability, drift check.
- → [Detailed Guide](./06-linux-os-layer.md)

## Q7: Containers & Monitoring

- **Runtime**: Podman for standalone containers; containerd for Kubernetes nodes.
- **Placement**: Dedicated container-host VMs or worker nodes, not on hypervisors.
- **Registry**: Harbor private registry with Trivy or equivalent scanning before promotion.
- **Monitoring**: Prometheus + Grafana + Alertmanager + EFK/OpenSearch for logs.
- **Alerts**: Critical (node down, disk >90%, API unavailable), Warning (CPU/memory pressure), Info (drift, cert expiry).
- **Ops model**: Rootless where possible, image signing/scanning, runbooks for alert actions.
- **Verification**: `podman ps`, exporter targets up, alert route test, dashboard and log search validation.
- → [Detailed Guide](./07-containers-and-monitoring.md)

## Q8: Troubleshooting — Failure Domain Isolation

- **Method**: Bottom-up validation by layer, never jump straight to the app.
- **Order**: Physical → Network → Firewall → Hypervisor → Storage → OS → Container → Application.
- **Rule**: Each layer has one or two decisive checks before moving higher.
- **Goal**: Reduce blast radius quickly, isolate the failing domain, and avoid random concurrent changes.
- **Signals**: Link state, route/VLAN health, quorum, storage latency, host pressure, runtime health, app SLOs.
- **Escalation**: If the lower layer is unhealthy, hold upper-layer changes until baseline is restored.
- **Verification**: Use saved commands/runbooks and timestamp every observation.
- → [Detailed Guide](./08-troubleshooting-guide.md)

## Q9: Multi-Tenant Incident Triage

- **First 15 minutes**: Acknowledge → identify blast radius → confirm shared dependency → decide failover/rollback/hold.
- **Storage incidents**: Check controller health, `iostat`, queue depth, multipath status, switch drops, noisy neighbors.
- **Containment**: Isolate the offender, preserve logs, and avoid broad reboots unless a shared layer is proven bad.
- **Communication**: Send time-stamped updates to tenants, app owners, and leadership based on severity.
- **Decision rule**: Fast failover if signal is clear; hold and investigate if the root cause is still ambiguous.
- **Metrics**: Track MTTA, MTTR, customer impact window, and recurrence frequency.
- **Verification**: Recovery is complete only after the platform stabilizes and alerts clear.
- → [Detailed Guide](./08-troubleshooting-guide.md)

## Q10: Post-Incident & Repeatability

- **Timeline**: Incident summary within 1 hour, blameless review within 24 hours, action plan within 1 week.
- **Feedback loop**: Manual fix becomes automation; missed alert becomes rule; weak runbook becomes rehearsal item.
- **Decision point**: One-off patch for isolated first-time defects; structural change for recurring or multi-layer faults.
- **Documentation**: Preserve timeline, impact, root cause, contributing factors, and clear owners.
- **Governance**: Track action items until closed; re-test the scenario in a drill.
- **Success metric**: Lower MTTR, smaller blast radius, fewer repeat incidents, higher operator confidence.
- **Verification**: Confirm alert, dashboard, and runbook updates were actually shipped.
- → [Detailed Guide](./08-troubleshooting-guide.md)

## Q11: Kubernetes on Bare-Metal

- **Topology**: 3 control plane VMs + 3-5 worker VMs + 2 LB VMs for API VIP.
- **Install choice**: kubeadm HA with stacked etcd for the first production platform.
- **Networking**: Calico CNI, MetalLB (L2) for service IPs, NGINX Ingress for edge routing.
- **Storage**: NFS CSI for general persistent volumes; graduate to block/object options only when needed.
- **Security**: RBAC, Pod Security Standards, network policies, sealed-secrets/external-secrets.
- **Day-2**: etcd snapshots, staged upgrades, cert rotation, namespace quotas, cluster backup tests.
- **Verification**: `kubectl get nodes`, API VIP health, PVC bind test, ingress test, upgrade rehearsal.
- → [Detailed Guide](./09-kubernetes-deployment.md)

## Delivery Phases

| Phase | Outcome | Gate to Move Forward |
|------|---------|----------------------|
| Phase 0 | Rack, power, firmware, OOB | Hardware healthy, firmware standardized |
| Phase 1 | VLANs, trunks, routing, DNS | Dual-homed hosts and tested segmentation |
| Phase 2 | Hypervisor cluster | Quorum healthy, fencing proven |
| Phase 3 | Shared storage | Multipath healthy, migration test passes |
| Phase 4 | VM factory | Golden image + Terraform + Ansible succeed |
| Phase 5 | Linux baseline | CIS-aligned controls and patch workflow verified |
| Phase 6 | Containers and K8s | Runtime, registry, monitoring, and LB path validated |
| Phase 7 | Day-2 operations | Backups, alerts, drills, rollback playbooks ready |

## Verification Gates

| Layer | Primary Check | Healthy Signal |
|------|---------------|----------------|
| Physical | BMC reachability, sensors | No hardware faults, remote console works |
| Network | `iperf3`, failover pull test | Expected bandwidth, links converge cleanly |
| Hypervisor | `pvecm status` | All nodes online, quorum intact |
| Storage | `multipath -ll`, `fio` | Paths active, latency baseline recorded |
| VM Factory | `cloud-init status --wait` | VM boots and config applies cleanly |
| OS | `sestatus`, `auditctl -s` | Enforcing, audited, drift controlled |
| Monitoring | alert test + dashboard check | End-to-end signal visible |
| Kubernetes | `kubectl get nodes` | Control plane stable, workers Ready |

## Minimum Viable Production Baseline

| Component | Minimum Sensible Baseline |
|-----------|---------------------------|
| Hypervisors | 3 Proxmox nodes with spare headroom |
| Switching | 2 managed switches with MLAG/stack or equivalent |
| Firewall | VPN + bastion + default-deny inter-zone policy |
| Storage | Shared NFS + iSCSI with backups and replication plan |
| Automation | Git + Packer + Terraform + Ansible + vault |
| Observability | Prometheus, Alertmanager, Grafana, central logs |
| Kubernetes | 3 control planes, 3 workers, API VIP, Calico, MetalLB |

## Cost & Resource Summary

| Component | Qty | Est. Cost Range | Notes |
|-----------|-----|----------------|-------|
| Bare-metal servers (2U, 2x Xeon/EPYC, 256 GB RAM) | 3-5 | $8K-$15K each | Dell R750, HPE DL380, Supermicro equivalent |
| 10/25 GbE managed switches | 2 | $2K-$5K each | Stack/MLAG-capable pair |
| Hardware firewall | 1-2 | $3K-$10K | pfSense appliance, FortiGate, Palo Alto entry enterprise |
| Shared storage appliance | 1 | $10K-$30K | Dual-controller NAS/SAN, 10/25 GbE |
| Rack + PDU + UPS | 1 | $3K-$8K | 42U rack, redundant PDUs, UPS |
| LB / utility VMs | 2-4 | Included in host cost | HAProxy, bastion, monitoring, registry |
| Software | — | $0 core stack | Proxmox, Kubernetes, Prometheus, Harbor, Ansible |
| Support contracts (optional) | — | $5K-$25K/yr | Hardware warranty, firewall subscriptions, enterprise OS support |
| **Total estimate** | — | **$35K-$100K** | Depends on CPU/RAM density, storage media, and support tier |

## Quick Reference

| Need | Primary Doc | Related Repo Track |
|------|-------------|-------------------|
| Hypervisor clustering | [01-hypervisor-layer.md](./01-hypervisor-layer.md) | [../Virtual-Setup/01-virtualization-fundamentals.md](../Virtual-Setup/01-virtualization-fundamentals.md) |
| VLANs, switches, DNS | [02-network-design.md](./02-network-design.md) | [../Physical-Setup/03-network-architecture.md](../Physical-Setup/03-network-architecture.md) |
| Firewall and host hardening | [03-firewall-and-security.md](./03-firewall-and-security.md) | [../Physical-Setup/02-os-installation-and-hardening.md](../Physical-Setup/02-os-installation-and-hardening.md) |
| Shared storage | [04-shared-storage.md](./04-shared-storage.md) | [../Physical-Setup/06-advanced-production-setup.md](../Physical-Setup/06-advanced-production-setup.md) |
| VM factory | [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) | [../Virtual-Setup/08-ci-cd-and-automation.md](../Virtual-Setup/08-ci-cd-and-automation.md) |
| Linux operations baseline | [06-linux-os-layer.md](./06-linux-os-layer.md) | [../Physical-Setup/02-os-installation-and-hardening.md](../Physical-Setup/02-os-installation-and-hardening.md) |
| Containers and observability | [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) | [../Virtual-Setup/07-production-kubernetes-setup.md](../Virtual-Setup/07-production-kubernetes-setup.md) |
| Incident handling | [08-troubleshooting-guide.md](./08-troubleshooting-guide.md) | [../13-Troubleshooting/](../13-Troubleshooting/) |
| Kubernetes deployment | [09-kubernetes-deployment.md](./09-kubernetes-deployment.md) | [../Virtual-Setup/07-production-kubernetes-setup.md](../Virtual-Setup/07-production-kubernetes-setup.md) |

## Interview Delivery Notes

- Lead with dependency order, not tools.
- State the default technology choices quickly, then justify trade-offs.
- Show how you prevent split-brain, lockouts, flat networking, and storage bottlenecks.
- Mention automation and observability as day-one requirements, not phase-two nice-to-haves.
- Close with verification gates, rollback paths, and cost-aware scaling decisions.
