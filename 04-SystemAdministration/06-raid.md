# Software RAID with mdadm

> **📌 Disclaimer**: Any third-party logos, screenshots, or diagrams referenced in this document are used for educational purposes only. All trademarks belong to their respective owners.


---

## 4.14 RAID basics with mdadm

Software RAID can be managed using `mdadm`.

Common RAID levels:
- RAID 0: striping, no redundancy.
- RAID 1: mirroring.
- RAID 5: striping with single parity.
- RAID 6: striping with double parity.
- RAID 10: mirrored stripes.

## 4.15 RAID levels comparison

### 📸 RAID Levels Comparison
![RAID 0](https://upload.wikimedia.org/wikipedia/commons/9/9b/RAID_0.svg)
> *RAID 0 — Striping (performance, no redundancy)*

![RAID 1](https://upload.wikimedia.org/wikipedia/commons/b/b7/RAID_1.svg)
> *RAID 1 — Mirroring (redundancy)*

![RAID 5](https://upload.wikimedia.org/wikipedia/commons/6/64/RAID_5.svg)
> *RAID 5 — Striping with distributed parity*

![RAID 10](https://upload.wikimedia.org/wikipedia/commons/b/bb/RAID_10.svg)
> *RAID 10 — Striped mirrors (performance + redundancy)*

```mermaid
graph TD
    A["RAID 0<br/>Fast<br/>No redundancy"] --> B["Use for scratch or non-critical workloads"]
    C["RAID 1<br/>Mirror<br/>High read reliability"] --> D["Use for boot or simple resilience"]
    E["RAID 5<br/>Single parity<br/>Efficient capacity"] --> F["Use for balanced capacity and redundancy"]
    G["RAID 6<br/>Double parity<br/>Better fault tolerance"] --> H["Use for larger arrays"]
    I["RAID 10<br/>Mirror plus stripe<br/>High performance"] --> J["Use for databases and critical workloads"]
```

### 4.15.1 Create a RAID 1 array

Example:

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

Create filesystem:

```bash
sudo mkfs.ext4 /dev/md0
```

Watch sync status:

```bash
cat /proc/mdstat
```

Array detail:

```bash
sudo mdadm --detail /dev/md0
```

Persist mdadm config on supported distributions:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```
