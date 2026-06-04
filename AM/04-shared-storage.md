# Question 4: Shared Storage

Shared storage is the layer that turns a cluster of hosts into a movable, recoverable platform. If storage is weak, VM mobility, HA restart, backups, and performance all become fragile.

## Why Storage Must Be Ready Before VMs

- VM disks live on shared storage, so a storage outage can impact the whole platform.
- HA and live migration assume any node can access the same VM disk.
- golden images, templates, and ISO libraries are easier to distribute centrally.
- storage paths, MTU, and authentication should be validated before production workloads exist.

## Storage Protocol Choice

| Protocol | Pros | Cons | Best For |
|----------|------|------|----------|
| NFS | Simple, file-level, easy to manage | Higher latency, lower IOPS ceiling | General VMs, templates, ISOs |
| iSCSI | Block-level, better IOPS, lower latency | More moving parts, LUN management | Databases and high-performance VMs |
| Ceph | Distributed, scalable, self-healing | Operational complexity, RAM hungry, needs 3+ nodes | Larger environments and K8s-native storage |

## Recommendation

Use **NFS for general-purpose VM storage and templates**, plus **iSCSI for performance-critical workloads**.

Why this dual-protocol model works:

- simple operational model for most VMs
- performance path for databases and latency-sensitive workloads
- no need to adopt Ceph complexity on day one
- can be automated cleanly from the hypervisor side

## Decision tree

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Need shared VM storage] --> B{Performance critical?}
    B -->|No| C[NFS datastore]
    B -->|Yes| D{iSCSI skills and multipath ready?}
    D -->|Yes| E[iSCSI LUNs]
    D -->|No| F[NFS first, revisit later]
    E --> G[Benchmark and isolate DB workloads]
    C --> H[Use for templates general VMs backups]
~~~

## Storage-side configuration first

### NFS design choices

Use NFSv4 where possible. Prefer:

- dedicated export path per environment or tier
- static host allowlists
- root squash unless a documented exception exists
- Kerberos if the environment already has the identity plumbing to support it

### Example `/etc/exports`

~~~bash
/srv/nfs/proxmox-images 10.10.20.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/proxmox-backups 10.10.20.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/proxmox-vmdata 10.10.20.0/24(rw,sync,no_subtree_check,no_root_squash)
~~~

### NFS services

~~~bash
dnf install -y nfs-utils
systemctl enable --now nfs-server rpcbind
exportfs -rav
showmount -e localhost
~~~

### Kerberos-capable NFS notes

If the site already uses FreeIPA or AD-integrated Kerberos, consider `sec=krb5p` for stronger integrity and privacy on sensitive exports.

### iSCSI design choices

Use:

- dedicated target IQNs
- CHAP authentication
- explicit ACLs per initiator IQN
- thin provisioning only when capacity monitoring is mature

### Example targetcli workflow

~~~bash
dnf install -y targetcli
systemctl enable --now target

targetcli /backstores/block create vm-db-lun /dev/mapper/vg_iscsi-lun0
targetcli /iscsi create iqn.2025-06.com.example:am.storage
targetcli /iscsi/iqn.2025-06.com.example:am.storage/tpg1/luns create /backstores/block/vm-db-lun
targetcli /iscsi/iqn.2025-06.com.example:am.storage/tpg1/acls create iqn.1993-08.org.debian:01:pve01
targetcli /iscsi/iqn.2025-06.com.example:am.storage/tpg1/set attribute authentication=1
targetcli /iscsi/iqn.2025-06.com.example:am.storage/tpg1 set auth userid=proxmoxuser password='REPLACE_ME'
targetcli saveconfig
~~~

## RAID, snapshots, and capacity policy

| Choice | Why | Trade-off |
|--------|-----|-----------|
| RAID-10 | Better write latency, faster rebuilds | Lower usable capacity |
| RAID-6 | Better usable capacity, tolerates two drive failures | Slower writes and rebuild behavior |
| Thick provisioning | Predictable capacity usage | Lower flexibility |
| Thin provisioning | Better utilization | Risk of surprise exhaustion without monitoring |

**Recommendation:**

- RAID-10 for performance tiers
- RAID-6 for capacity-oriented general tiers
- snapshots on a policy, not ad hoc forever
- replication to a second system when the business impact justifies it

## Snapshot and replication schedule example

| Dataset | Snapshot Frequency | Retention | Replication |
|---------|--------------------|-----------|-------------|
| VM templates | daily | 14 days | weekly |
| General VMs | hourly | 48 hours | daily |
| Database LUNs | every 15 min app-consistent where possible | 24 hours | hourly or near-real-time |

## Hypervisor-side configuration second

### NFS on Proxmox

~~~bash
pvesm add nfs am-nfs-vmdata \
  --server 10.10.20.50 \
  --export /srv/nfs/proxmox-vmdata \
  --path /mnt/pve/am-nfs-vmdata \
  --content images,iso,vztmpl,backup \
  --options vers=4.2,soft,timeo=600,retrans=2

pvesm status
~~~

### iSCSI on Proxmox

~~~bash
pvesm add iscsi am-iscsi-db \
  --portal 10.10.20.60 \
  --target iqn.2025-06.com.example:am.storage \
  --content images

iscsiadm -m discovery -t sendtargets -p 10.10.20.60
iscsiadm -m node --login
~~~

### NFS mount options

Recommended options for modern Linux clients:

- `vers=4.2`
- `nconnect=4`
- `rsize=1048576`
- `wsize=1048576`
- `noatime`

### Example fstab entry for validation VM

~~~bash
10.10.20.50:/srv/nfs/proxmox-vmdata /mnt/testnfs nfs4 vers=4.2,nconnect=4,rsize=1048576,wsize=1048576,noatime 0 0
~~~

## Storage connectivity diagram

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    P1[pve01] --> STSW[Storage VLAN 20]
    P2[pve02] --> STSW
    P3[pve03] --> STSW
    STSW --> NFS[NFS Service]
    STSW --> ISCSI[iSCSI Targets]
    NFS --> APP[General VM disks and templates]
    ISCSI --> DB[Database and high-IO workloads]
~~~

## Multipath I/O for iSCSI

Use `dm-multipath` when each hypervisor sees multiple target paths.

### Install and enable

~~~bash
apt update
apt install -y open-iscsi multipath-tools
mpathconf --enable
systemctl enable --now multipathd open-iscsi
multipath -ll
~~~

### Example `/etc/multipath.conf`

~~~bash
defaults {
    user_friendly_names yes
    find_multipaths yes
}

blacklist {
    devnode "^sda"
}

devices {
    device {
        vendor "LIO-ORG"
        product "TCMU device"
        path_grouping_policy multibus
        path_selector "round-robin 0"
        failback immediate
        rr_min_io 100
        no_path_retry 12
    }
}
~~~

## Storage HA architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    P1[pve01] --> A[Path A]
    P1 --> B[Path B]
    P2[pve02] --> A
    P2 --> B
    P3[pve03] --> A
    P3 --> B
    A --> C1[Storage Controller A]
    B --> C2[Storage Controller B]
    C1 --- C2
    C1 --> D[Shared RAID pools]
    C2 --> D
~~~

## Redundancy & availability patterns

### Storage appliance

- prefer dual controllers if available
- dual power feeds
- export config backups regularly
- monitor controller failovers and cache battery status

### NFS redundancy options

| Option | Pros | Cons |
|--------|------|------|
| Single NFS head on reliable appliance | simple | single service head risk |
| HA NFS pair with Pacemaker/DRBD | can survive node failure | more Linux HA complexity |
| Vendor NAS with HA controllers | clean operational model | vendor dependence and cost |

### iSCSI redundancy options

- multiple target portals
- dual switches on storage VLAN
- multipath on every initiator
- CHAP secrets stored in password vault and rotation workflow

## Performance tuning

### Jumbo frames

Only enable `MTU 9000` if **every** device on the storage path matches.

### NFS tuning

~~~bash
mount -t nfs4 -o vers=4.2,nconnect=4,rsize=1048576,wsize=1048576,noatime 10.10.20.50:/srv/nfs/proxmox-vmdata /mnt/testnfs
nfsstat -m
~~~

### iSCSI tuning notes

- confirm queue depth from HBA or software initiator
- set readahead appropriately for sequential workloads
- separate noisy database loads into their own LUNs

## Benchmarking with fio

### NFS baseline

~~~bash
fio --name=nfs-randread --directory=/mnt/testnfs --size=4G --rw=randread --bs=4k --iodepth=32 --numjobs=4 --runtime=60 --time_based
~~~

### iSCSI baseline

~~~bash
fio --name=iscsi-randrw --filename=/dev/mapper/mpatha --rw=randrw --rwmixread=70 --bs=8k --iodepth=64 --numjobs=4 --runtime=60 --time_based --direct=1
~~~

Track:

- IOPS
- average latency
- P95/P99 latency
- bandwidth
- CPU usage during test

## NFS vs iSCSI summary diagram

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    NFS[NFS] --> N1[Simple management]
    NFS --> N2[Templates and general VM disks]
    NFS --> N3[File-level exports]
    ISCSI[iSCSI] --> I1[Lower latency]
    ISCSI --> I2[Database and high-IO VMs]
    ISCSI --> I3[Block-level LUNs]
~~~

## Verification checklist

| Check | Command | Healthy Signal |
|-------|---------|----------------|
| NFS export | `showmount -e 10.10.20.50` | expected export visible |
| iSCSI session | `iscsiadm -m session -P 3` | active session(s) present |
| Multipath | `multipath -ll` | multiple active paths |
| MTU | `ping -M do -s 8972 10.10.20.50 -c 3` | no fragmentation |
| Performance | `fio ...` | baseline recorded |
| Proxmox storage | `pvesm status` | storage online on all nodes |
| Mobility | live-migrate test VM | succeeds without disk disconnect |

## Failure tests worth scheduling

1. disconnect one storage link and confirm multipath survives
2. restart one controller in a maintenance window if vendor guidance allows it
3. live-migrate a test VM during synthetic I/O load
4. fill a test datastore near warning thresholds and verify alerting

## Common mistakes

- putting storage on the same oversubscribed path as VM data traffic
- enabling jumbo frames on only half the path
- using thin provisioning without alerting and capacity reviews
- skipping CHAP or export restrictions because the network is “internal”
- deploying VMs before running even one real performance baseline

## Exit criteria

You are ready for [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) when:

- NFS and/or iSCSI are mounted from every hypervisor
- path redundancy is validated
- baseline performance is documented
- shared storage supports a successful live migration test


## Procurement & Cost Analysis

### Storage platform comparison

| Option | Example | Budgetary Range | Best Fit | Watch-outs |
|--------|---------|-----------------|----------|------------|
| TrueNAS Enterprise / SCALE | M-series or self-built validated nodes | $8K-$20K | cost-efficient NFS/iSCSI, open ZFS model | requires stronger in-house storage skill |
| Synology RackStation | RS3621xs+ / HD6500 | $6K-$18K | SMB/mid-market simplicity | less ideal for very high IOPS databases |
| NetApp | AFF / FAS entry systems | $20K-$60K+ | enterprise support, snapshots, replication | higher license and support cost |
| Pure Storage | FlashArray //X or //C | $40K-$100K+ | premium low-latency all-flash workloads | premium budget required |

### Disk media economics

| Media | Typical Use | Cost per TB | Relative IOPS per Dollar | Notes |
|------|-------------|-------------|--------------------------|------|
| 7.2K HDD | backups, cold VM tiers | $20-$40/TB | Low | capacity-focused only |
| SATA/SAS SSD | general VM datastore | $80-$180/TB | Medium | strong balance for mixed workloads |
| NVMe SSD | high-IO DB and metadata | $120-$300/TB | High | needs enough PCIe lanes and cooling |

### Procurement guidance

- Buy dual controllers or dual heads where supported if HA matters.
- Include 10/25 GbE ports, redundant PSUs, cache protection, and spare drives in the BOM.
- Verify HCL support for HBAs, multipath, and target features before purchase.
- Plan shelf expansion and rack depth early; storage chassis often drive rack design.
- Source from the storage vendor, a certified reseller, or an integrator that will own the support chain.

## Resource Planning

### Capacity planning formulas

1. `Required usable TB = current dataset + snapshots + backups in tier + projected growth`
2. `Projected 18-month usable TB = current usable TB x 2` for the requested growth target.
3. `Raw TB = usable TB ÷ RAID efficiency`
4. `Thin provisioning ceiling = allocated ÷ physical ≤ 1.5:1` unless strong alerts and cleanup discipline exist.

### Example sizing table

| Workload | Current Need | 18-Month Target | Recommended Tier |
|----------|--------------|-----------------|------------------|
| General VMs | 8 TB usable | 16 TB usable | NFS on SSD-backed NAS |
| Databases | 2 TB usable | 4 TB usable | iSCSI LUNs on SSD/NVMe |
| Snapshots/overhead | 3 TB | 6 TB | separate reserve pool |
| Total usable | 13 TB | 26 TB | procure ~40-45 TB raw depending on RAID |

### IOPS and staffing guidance

- Size storage on latency and IOPS before raw TB for databases and Kubernetes stateful sets.
- Rule of thumb: 4K random write-heavy DBs need NVMe or strong write cache; mixed VM estates often fit SSD-backed hybrid systems.
- Minimum staffing: 1 infra engineer plus shared storage specialist or strong vendor support contract.
- Scale up with more cache/NVMe for latency bottlenecks; scale out with additional arrays or tiers when capacity and fault isolation are the main issues.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | NFS for templates/general VM data | simplest operational model | all-iSCSI rejected for unnecessary complexity |
| ADR-02 | iSCSI for high-performance block workloads | better latency and queueing control | databases on shared NFS rejected |
| ADR-03 | Dedicated storage VLAN | predictable latency and troubleshooting | shared flat network rejected |
| ADR-04 | Multipath on all initiators | no single-port dependency | single path rejected |

### Integration points

- Network MTU and switch design from [02-network-design.md](./02-network-design.md) directly affect this layer.
- Hypervisor storage definitions in [01-hypervisor-layer.md](./01-hypervisor-layer.md) and VM layout in [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) consume the outputs.
- Kubernetes PV strategy in [09-kubernetes-deployment.md](./09-kubernetes-deployment.md) depends on stable exports and snapshots.

## Planning & Timeline

### Rollout plan

| Week | Goal | Deliverables |
|------|------|--------------|
| Week 1 | platform selection and media plan | vendor shortlist, RAID profile, raw/usable sizing |
| Week 2 | rack and configure | controller setup, VLAN 20 connectivity, exports/LUNs |
| Week 3 | attach and validate | multipath, benchmarks, live migration, snapshot tests |

### Prerequisites checklist

- Storage VLAN and jumbo-frame decision finalized.
- Disk layout, RAID policy, and snapshot retention approved.
- Backup target and replication target identified.
- CHAP/export restrictions stored securely.
- Vendor firmware and support portal access ready.

### Risk register and rollback

| Risk | Impact | Mitigation | Rollback |
|------|--------|------------|----------|
| Under-sized cache/media | poor latency | benchmark with realistic fio profiles | move hot workloads to faster tier |
| Thin provisioning exhaustion | outage risk | 70/80/90% alerts and monthly review | reclaim space, extend pool, pause growth |
| Jumbo-frame mismatch | intermittent I/O failures | end-to-end MTU tests | revert to MTU 1500 |
| Controller firmware issue | degraded storage | staged upgrade and support case plan | revert firmware if vendor supports |

## Advanced Production Configurations

- Use SSD metadata or special vdevs on ZFS platforms where metadata latency becomes the bottleneck.
- For DR, replicate snapshots to a second appliance or site and test mount/promote workflows quarterly.
- Compliance impact: encryption at rest, access logging, and backup immutability requirements increase under PCI/HIPAA.
- Alert on controller failover, disk SMART errors, pool fragmentation, P95 latency, write cache battery issues, and capacity >70/80/90%.
- Auto-remediation may move a datastore to read-only or fail traffic to controller B, but keep destructive storage actions manual.
- Revisit NFS versus CSI/block design when Kubernetes stateful workloads become a large part of the estate.

## Building & Deployment Runbook

1. Rack the storage appliance, cable dual controllers to both storage switches, and update firmware.
2. Create storage pools or aggregates based on the approved media/RAID design.
3. Publish NFS exports and iSCSI targets/LUNs with export restrictions or CHAP.
4. On each hypervisor, discover and connect:
   ~~~bash
   showmount -e 10.10.20.50
   iscsiadm -m discovery -t sendtargets -p 10.10.20.60
   iscsiadm -m node --login
   multipath -ll
   ~~~
5. Add storage into Proxmox and verify all nodes see it.
6. Run fio baselines for NFS and iSCSI, then record latency, IOPS, and throughput.
7. Validation gate: live-migrate a test VM while synthetic storage load runs; confirm no path loss or datastore disconnects.
8. Enable snapshots, backup jobs, and alert thresholds before onboarding production VMs.

### Common mistakes to avoid during build

- Buying mostly HDD because usable TB looks cheap, then discovering latency is the real requirement.
- Forgetting expansion shelves, rack depth, or power draw in the first procurement cycle.
- Treating thin provisioning as free capacity.
- Benchmarking only sequential throughput and missing random-write pain.


## Cross-references

- Hypervisor cluster: [01-hypervisor-layer.md](./01-hypervisor-layer.md)
- Network and MTU planning: [02-network-design.md](./02-network-design.md)
- VM provisioning next: [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md)
- Related repo reference: [../Physical-Setup/06-advanced-production-setup.md](../Physical-Setup/06-advanced-production-setup.md)
