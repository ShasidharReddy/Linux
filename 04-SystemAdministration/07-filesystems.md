# Filesystems

---

Choosing the right filesystem matters for reliability, performance, scalability, and feature set.

## 7.1 Common filesystem options
This guide focuses on:
- ext4
- XFS
- Btrfs
- ZFS
- tmpfs
- swap

## 7.2 ext4
`ext4` is a mature, general-purpose Linux filesystem.

Strengths:
- Stable and widely supported.
- Good for root filesystems and general workloads.
- Supports resizing.
- Familiar recovery tools.

Common use cases:
- Small to medium servers.
- General-purpose VMs.
- Boot partitions.
- Conservative production setups.

Create ext4 filesystem:

```bash
sudo mkfs.ext4 /dev/sdb1
```

Label it:

```bash
sudo e2label /dev/sdb1 DATA
```

Check filesystem:

```bash
sudo fsck.ext4 -f /dev/sdb1
```

Tune ext4:

```bash
sudo tune2fs -l /dev/sdb1
sudo tune2fs -m 1 /dev/sdb1
sudo tune2fs -c 30 /dev/sdb1
```

Notes:
- Reserved blocks can be useful on system volumes.
- `tune2fs` is powerful and should be used carefully.

## 7.3 XFS
`XFS` is a high-performance filesystem widely used in enterprise Linux.

Strengths:
- Excellent for large files and large filesystems.
- Strong metadata handling.
- Default on many RHEL-family installations.
- Online growth supported.

Limitations:
- Cannot shrink.
- Plan capacity carefully.

Create XFS:

```bash
sudo mkfs.xfs /dev/sdb1
```

Grow XFS:

```bash
sudo xfs_growfs /mountpoint
```

Repair XFS:

```bash
sudo xfs_repair /dev/sdb1
```

Common use cases:
- Database logs.
- Media or analytics data.
- Large LVM-backed volumes.
- Enterprise servers with predictable growth only in the upward direction.

## 7.4 Btrfs
`Btrfs` is a modern copy-on-write filesystem with advanced features.

Features:
- Snapshots.
- Subvolumes.
- Checksums.
- Compression.
- Send and receive replication.
- Integrated multi-device capabilities.

Create Btrfs:

```bash
sudo mkfs.btrfs /dev/sdb1
```

Basic commands:

```bash
sudo btrfs filesystem show
sudo btrfs subvolume list /mnt
sudo btrfs subvolume create /mnt/@data
sudo btrfs filesystem df /mnt
```

When to use:
- Snapshot-heavy systems.
- Workstations.
- Some server environments with admin familiarity.

When to be cautious:
- If your team lacks operational experience.
- If vendor support policy limits filesystem choice.

## 7.5 ZFS
ZFS combines a filesystem and volume manager.
It is feature-rich and highly respected for integrity-focused storage.

Key features:
- End-to-end checksumming.
- Snapshots and clones.
- Compression.
- Copy-on-write design.
- Integrated RAID-like capabilities.
- Scrubbing and strong data integrity features.

Common commands:

```bash
zpool status
zpool list
zfs list
zfs snapshot pool1/data@daily-2024-01-01
zfs rollback pool1/data@daily-2024-01-01
```

When to use:
- Storage servers.
- Backup systems.
- Environments where integrity features are a major advantage.

When to be cautious:
- RAM requirements.
- Operational complexity.
- Distribution support expectations.

## 7.6 tmpfs
`tmpfs` is a memory-backed filesystem.
It can also use swap.

Use cases:
- Temporary files.
- Build caches.
- Runtime directories.
- Shared memory-style workloads.

Mount example:

```bash
sudo mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk
```

`fstab` example:

```fstab
tmpfs /run/mycache tmpfs defaults,size=512M 0 0
```

Be careful not to over-allocate memory-backed mounts.

## 7.7 swap as virtual memory backing
Swap is not a regular filesystem for user data.
It is used by the kernel for memory management.

Use cases:
- Buffer for memory pressure.
- Hibernation on some systems.
- Preventing immediate OOM on transient spikes.

Do not rely on swap to fix undersized systems.
It is a safety margin, not a substitute for RAM.

## 7.8 When to use which filesystem
| Filesystem | Best for | Key strengths | Key cautions |
|---|---|---|---|
| ext4 | General-purpose servers | Stable, mature, easy recovery | Fewer modern features than Btrfs or ZFS |
| XFS | Large filesystems and enterprise workloads | Performance, scale, online growth | Cannot shrink |
| Btrfs | Snapshot and subvolume workflows | Snapshots, compression, send/receive | Requires familiarity |
| ZFS | Integrity-focused storage | Checksums, snapshots, advanced storage features | Operational overhead and support considerations |
| tmpfs | Temporary memory-backed data | Very fast | Consumes RAM and swap |
| swap | Virtual memory | Helps with spikes | Slow compared to RAM |

## 7.9 mkfs overview
Common filesystem creation commands:

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb1
sudo mkfs.btrfs /dev/sdb1
sudo mkswap /dev/sdb2
```

Warning:
These commands destroy existing filesystem data on the target.
Always verify the device first.

## 7.10 fsck basics
`fsck` checks and repairs filesystems, usually while unmounted.

Examples:

```bash
sudo fsck -f /dev/sdb1
sudo fsck.ext4 -y /dev/sdb1
```

Notes:
- Do not run generic `fsck` blindly on mounted filesystems unless the specific filesystem and scenario support it.
- XFS uses `xfs_repair` rather than classic `fsck` repair logic.

## 7.11 Tune and maintenance utilities
For ext-family:

```bash
sudo tune2fs -l /dev/sdb1
sudo dumpe2fs -h /dev/sdb1
```

For XFS:

```bash
sudo xfs_info /mountpoint
```

For Btrfs:

```bash
sudo btrfs scrub start /mnt
sudo btrfs filesystem usage /mnt
```

For ZFS:

```bash
sudo zpool scrub pool1
sudo zpool status
```

## 7.12 Mount options matter
Common mount options:
- `defaults`
- `noatime`
- `relatime`
- `nodev`
- `nosuid`
- `noexec`
- `ro`
- `rw`

Examples:

```fstab
UUID=1111-2222 /data ext4 defaults,noatime 0 2
UUID=3333-4444 /tmp ext4 defaults,nodev,nosuid,noexec 0 2
```

Security and performance both depend on mount options.

## 7.13 Filesystem troubleshooting
Symptoms:
- Slow I/O.
- Mount failures.
- Read-only remounts.
- Corruption warnings.
- Full filesystem.
- Metadata errors.

Commands to start with:

```bash
dmesg | tail -50
journalctl -b | tail -100
mount | column -t
findmnt
lsblk -f
df -h
```

If the filesystem remounts read-only:
- Stop writes.
- Collect logs.
- Back up what you can.
- Schedule a proper repair.

## 7.14 Filesystem best practices
- Use ext4 or XFS for predictable server defaults.
- Use UUIDs in `fstab`.
- Keep recovery tools available.
- Understand shrink and grow limitations.
- Separate data, logs, and temp spaces when appropriate.
- Monitor inode use.
- Test snapshot and restore workflows.
- Do not experiment on production without rehearsal.

---

## 7.5 Filesystem commands reference
- `mkfs.ext4 /dev/sdb1`
- `mkfs.xfs /dev/sdb1`
- `mkfs.btrfs /dev/sdb1`
- `mkswap /dev/sdb2`
- `fsck -f /dev/sdb1`
- `fsck.ext4 -f /dev/sdb1`
- `xfs_repair /dev/sdb1`
- `tune2fs -l /dev/sdb1`
- `tune2fs -m 1 /dev/sdb1`
- `e2label /dev/sdb1 DATA`
- `xfs_info /mountpoint`
- `btrfs filesystem show`
- `btrfs filesystem usage /mnt`
- `btrfs subvolume list /mnt`
- `zpool status`
- `zfs list`
- `mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk`

---

## B.5 Filesystem quick reminders
- ext4 is a safe default.
- XFS is common for large enterprise volumes.
- Btrfs and ZFS reward familiarity with powerful features.
- tmpfs is fast but consumes memory resources.
- Always know whether a filesystem can grow or shrink.
- Repair tools differ by filesystem type.
- Do not run the wrong repair tool against the wrong filesystem.
- Use labels and UUIDs to make mounts clearer.
- Review mount options for security-sensitive paths.
- Separate temporary and executable paths when policy requires.
- Monitor both space and inode consumption.
- Investigate read-only remounts immediately.
- Save relevant `dmesg` output when corruption is suspected.
- Practice snapshot restore, not only snapshot creation.
- Verify backups before risky filesystem work.

---

### Filesystem inspection examples
```bash
tune2fs -l /dev/sda1 | head -40
xfs_info /
btrfs filesystem usage /
zpool status -v
```

---

## B.28 More filesystem examples
```bash
findmnt -no OPTIONS /
mount | grep tmpfs
swapon --show --bytes
btrfs scrub status /mnt
zpool get all pool1 | head -40
```
- Filesystem mount options directly affect security posture and performance profile.
- Compression settings matter on Btrfs and ZFS datasets.
- tmpfs sizing should be reviewed after application memory growth.
- Swap usage should be analyzed with overall memory pressure, not in isolation.
- Filesystem choice should align with support model and team familiarity.
- Large directory counts can affect operational tasks even when storage is available.
- Snapshot sprawl can become a management problem if retention is not enforced.
- Fragmentation characteristics differ by filesystem and workload.
- Some filesystems excel at large sequential I/O, while others shine in feature depth.
- Measure before assuming performance behavior.
