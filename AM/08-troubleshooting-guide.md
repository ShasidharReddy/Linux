# Questions 8, 9, 10: Troubleshooting

This chapter focuses on operating reality: finding the broken layer quickly, triaging shared incidents under pressure, and converting painful manual fixes into repeatable engineering improvements.

## Section 1: Failure-Domain Isolation

**Question:** “Application unreachable — which layer is broken?”

### Method: Bottom-Up Stack Validation

```text
Physical → Network → Firewall → Hypervisor → Storage → OS → Container → App
```

Start at the lowest layer that could explain the failure, and only move upward when the lower layer is proven healthy.

## Troubleshooting decision tree

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[App unreachable] --> B{Host reachable?}
    B -->|No| C[Check physical and OOB]
    B -->|Yes| D{Network path healthy?}
    D -->|No| E[Check VLAN bond gateway route]
    D -->|Yes| F{Firewall allowing traffic?}
    F -->|No| G[Check policy and packet arrival]
    F -->|Yes| H{VM running?}
    H -->|No| I[Check hypervisor and HA]
    H -->|Yes| J{Storage healthy?}
    J -->|No| K[Check mounts latency capacity]
    J -->|Yes| L{OS and service healthy?}
    L -->|No| M[Check systemd logs kernel]
    L -->|Yes| N{Container healthy?}
    N -->|No| O[Check runtime logs healthcheck]
    N -->|Yes| P[Check app endpoint and app logs]
~~~

## Layer checks and “move on” signals

### 1. Physical

**Check**

- can you ping the host or its management interface?
- is IPMI/iLO responding?
- are hardware sensors normal?

**Commands / checks**

~~~bash
ping -c 3 10.10.99.11
ipmitool -I lanplus -H 10.10.99.11 -U admin -a chassis status
~~~

**Signal to move on**

- OOB is reachable and no obvious power/thermal/hardware fault is present

### 2. Network

**Check**

- interface state
- VLAN tags and bridge membership
- default gateway reachability
- bond status

~~~bash
ip link show
ip -br addr
ip route
cat /proc/net/bonding/bond0
ping -c 3 10.10.30.1
~~~

**Signal to move on**

- links are up, routes are correct, and expected path is reachable

### 3. Firewall

**Check**

- is the port allowed?
- are packets arriving at the host?
- is there a zone mismatch?

~~~bash
nft list ruleset
firewall-cmd --list-all
ss -tulpn | grep 8080
tcpdump -ni any port 8080
~~~

**Signal to move on**

- packets reach the host and the intended service port is allowed

### 4. Hypervisor

**Check**

- is the VM actually running?
- is HA thrashing or restarting it?
- is CPU or memory exhausted?

~~~bash
qm list
qm status 200
pvecm status
ha-manager status
~~~

**Signal to move on**

- VM is up, placed on the expected host, and cluster health is good

### 5. Storage

**Check**

- mount present?
- LUN or NFS export reachable?
- filesystem full?
- latency high?

~~~bash
df -h
iostat -xz 1 5
mount | grep nfs
stat /mnt/pve/am-nfs-vmdata
multipath -ll
~~~

**Signal to move on**

- storage is mounted, not full, and latency is within normal range

### 6. OS (inside VM)

**Check**

- SSH reachable?
- service running?
- kernel errors or OOM?

~~~bash
systemctl status inventory-api
journalctl -u inventory-api -b
journalctl -k -b | tail -50
dmesg | tail -50
~~~

**Signal to move on**

- guest OS is stable and the service layer is the next likely fault domain

### 7. Container

**Check**

- container running?
- health check passing?
- logs show crashes?

~~~bash
podman ps
podman logs inventory-api --tail 100
podman inspect inventory-api | jq '.[0].State.Health'
~~~

**Signal to move on**

- runtime healthy, so investigate application logic itself

### 8. Application

**Check**

- local health endpoint
- upstream dependencies
- error rate and latency

~~~bash
curl -f http://127.0.0.1:8080/health
curl -I http://127.0.0.1:8080/
~~~

## Escalation through layers

~~~mermaid
%%{init:{"theme":"neutral"}}%%
sequenceDiagram
    participant U as User Alert
    participant O as Operator
    participant H as Host or OOB
    participant V as VM or Hypervisor
    participant S as Service
    U->>O: app unreachable
    O->>H: verify physical and network
    O->>V: verify VM, storage, and OS
    O->>S: verify container and app
    S->>O: root cause found
~~~

## Key diagnostic commands by layer

| Layer | Commands |
|-------|----------|
| Physical | `ipmitool`, BMC console, hardware logs |
| Network | `ip`, `ping`, `arping`, `ethtool`, `tcpdump` |
| Firewall | `nft`, `firewall-cmd`, `ss` |
| Hypervisor | `qm`, `pvecm`, `ha-manager` |
| Storage | `df`, `iostat`, `multipath`, `mountstats`, `nfsstat` |
| OS | `systemctl`, `journalctl`, `dmesg` |
| Container | `podman ps`, `podman logs`, `podman inspect` |
| App | `curl`, app-specific CLI, dependency checks |

## Section 2: Multi-Tenant Incident Triage

**Question:** “Storage latency spikes across many VMs, multiple teams paging.”

### First 15 Minutes Playbook

#### Minute 0–2: Acknowledge & communicate

- open the incident channel or war room
- state blast radius and next update time
- check whether it is all tenants or one subset

Example message:

> Investigating storage latency spike affecting multiple VMs. Next update in 10 minutes. Current focus: controller health, storage VLAN, noisy-neighbor analysis.

#### Minute 2–5: Quick triage

~~~bash
iostat -xz 1 5
nfsstat -c
iscsiadm -m session -P 3
sar -n DEV 1 5
~~~

Questions:

- one controller or both?
- one hypervisor or many?
- one VM generating abnormal write storm?
- interface drops or CRC errors on VLAN 20?

#### Minute 5–10: Isolate root cause

| Suspicion | Checks |
|-----------|--------|
| Controller issue | storage dashboard, controller failover state, cache status |
| Disk shelf issue | hardware alerts, SMART/vendor telemetry |
| Storage VLAN issue | switch errors, LACP status, MTU mismatch, drops |
| Noisy neighbor VM | top I/O VM on hypervisor, cgroup or disk stats |

#### Minute 10–15: Decide on action

| Scenario | Action | Reasoning |
|----------|--------|-----------|
| Controller failure | Fail over to standby controller | reduce blast radius quickly |
| Single VM causing I/O storm | throttle or stop that VM, notify owner | protect shared platform |
| Network issue on storage VLAN | fix switch or bond path | surgical path repair |
| Unknown | communicate ETA and continue | avoid random harmful changes |

## Incident triage flowchart

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Storage latency alert] --> B{All tenants affected?}
    B -->|Yes| C[Check controller and storage network]
    B -->|No| D[Find noisy neighbor or localized host]
    C --> E{Controller failed?}
    E -->|Yes| F[Fail over or engage vendor]
    E -->|No| G[Check switch drops MTU bond]
    D --> H[Throttle offending VM or move workload]
    G --> I[Apply low-blast-radius fix]
    H --> J[Communicate and monitor recovery]
    I --> J
~~~

## Section 3: Post-Incident & Repeatability

### Immediate timeline

1. within 1 hour: write incident summary
2. within 24 hours: hold blameless post-mortem
3. within 1 week: publish action items and owners

## Post-incident workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[Incident Resolved] --> B[Write Summary]
    B --> C[Hold Blameless Review]
    C --> D[Identify Root Cause and Contributors]
    D --> E[Create Action Items]
    E --> F[Automate Alert or Redesign]
    F --> G[Verify Improvement in Future Drills]
~~~

## Post-mortem template

- Title
- Date and duration
- Impact and affected users or teams
- Severity
- Detection method
- Timeline minute by minute
- Root cause analysis using 5 Whys
- Contributing factors
- What went well
- What went poorly
- Action items with owner and deadline

## One-off patch vs structural change

| Signal | One-Off Patch | Structural Change |
|--------|--------------|-------------------|
| First occurrence, clear cause | ✅ | |
| Recurring (2+ times) | | ✅ |
| Affects single component | ✅ | |
| Affects multiple layers | | ✅ |
| Design assumption violated | | ✅ |
| Quick fix under 1 hour | ✅ | |
| Requires architecture change | | ✅ |

## Feeding back into automation

- if the fix was manual, create Ansible or Terraform to make it repeatable
- if monitoring missed the event, add alert rules or synthetic checks
- if recovery was slow, write a runbook and rehearsal test
- if config drift caused the issue, move the setting into the image, role, or compliance control

## Checklist after every significant incident

1. was blast radius clearly identified?
2. were customer-facing updates timely?
3. did monitoring show leading indicators?
4. did documentation help or was it outdated?
5. what control will stop recurrence or reduce impact?

## Common mistakes during incidents

- jumping straight to app debugging when the host is unreachable
- changing many things at once without knowing which mattered
- skipping communication for the first 15 minutes
- failing to preserve logs, dashboards, and exact timestamps
- declaring “fixed” before the platform stabilizes


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### Incident command model

| Role | Responsibility |
|------|----------------|
| Incident Commander | owns timeline, prioritization, and stakeholder updates |
| Technical Lead | drives diagnosis and fix path |
| Communications Lead | updates app teams, leadership, and status page |
| Scribe | captures exact timestamps, actions, and evidence |

### Escalation matrix template

| Trigger | First Escalation | Second Escalation | Notes |
|--------|------------------|-------------------|------|
| shared storage degraded | storage engineer | vendor + infra lead | attach latency graphs and controller status |
| network-wide reachability | network engineer | ISP/vendor + incident commander | include blast radius map |
| hypervisor quorum loss | virtualization engineer | storage/network leads | confirm fencing and BMC state |
| Kubernetes API unavailable | platform engineer | infra lead | decide failover vs rollback quickly |

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

### War room setup guide

- Open one bridge for operators and one stakeholder channel for business updates.
- Pin dashboards for network, hypervisor, storage, auth, and Kubernetes/API health.
- Assign a scribe within 2 minutes; every action needs actor + timestamp + result.
- Use severity-based update cadence: every 15 minutes for Sev-1, every 30 minutes for Sev-2.

### Stakeholder communication templates

**Initial update**

> We are investigating a `Sev-X` infrastructure incident affecting `<scope>`. Start time: `<UTC>`. Current impact: `<summary>`. Next update in 15 minutes.

**Stabilization update**

> Service is partially restored as of `<UTC>`. We are validating storage/network/application health before declaring recovery. Risk of recurrence: `<low/medium/high>`.

**Resolution update**

> Incident resolved at `<UTC>`. Root cause is under review. A post-incident summary and action plan will follow within 24 hours.

### Compliance and automation notes

- SOC 2 and PCI expect evidence of incident handling, ownership, and timeline retention.
- Auto-remediation should be limited to reversible, low-blast-radius actions such as restarting a failed exporter or isolating a known noisy non-prod VM.
- Multi-site environments should define when to invoke DR or site failover versus localized repair.

## Building & Deployment Runbook

1. Define severity levels, paging rules, escalation matrix, and incident roles.
2. Integrate monitoring with paging/chat tools and test with a synthetic alert.
3. Build a shared war-room template with pinned dashboards, note-taking area, and stakeholder update placeholders.
4. Create per-layer runbooks and store them with the alert definitions.
5. Run a game day and capture metrics:
   ~~~bash
   date -u
   journalctl -S -15m -p warning..alert
   kubectl get nodes 2>/dev/null || true
   pvecm status 2>/dev/null || true
   ~~~
6. Validation gate: every Sev-1 simulation must show acknowledgment, role assignment, stakeholder update, and root-cause isolation in the recorded timeline.

### Common mistakes to avoid during build

- Buying an incident tool and calling the incident process “done.”
- Measuring MTTR from when an engineer noticed the issue manually instead of from detection/open time.
- Running war rooms with no communications lead and then overwhelming the technical lead.
- Skipping drills because the environment seems small.


## Cross-references

- Monitoring context: [07-containers-and-monitoring.md](./07-containers-and-monitoring.md)
- Shared storage incidents: [04-shared-storage.md](./04-shared-storage.md)
- K8s-specific recovery later: [09-kubernetes-deployment.md](./09-kubernetes-deployment.md)
