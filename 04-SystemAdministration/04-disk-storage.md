# Disk and Storage Management

---

Disk administration includes partitioning, identifying block devices, mounting file systems, managing logical volumes, and handling RAID.

## 4.1 Storage stack overview

Common layers:
- Physical disk or virtual disk.
- Partition table.
- Partition.
- RAID or encryption layer if used.
- LVM physical volume.
- Volume group.
- Logical volume.
- File system.
- Mount point.

Understanding the stack is critical before resizing or recovering storage.

## 4.2 Device discovery tools

Useful commands:

```bash
lsblk
lsblk -f
blkid
fdisk -l
parted -l
cat /proc/partitions
```

What they show:
- `lsblk` shows block device hierarchy.
- `lsblk -f` shows file system and UUID data.
- `blkid` prints UUIDs and labels.
- `fdisk -l` lists partition tables.
- `parted -l` lists disks and labels.

Identify devices carefully.
On modern systems names may be:
- `/dev/sda`
- `/dev/vda`
- `/dev/nvme0n1`
- `/dev/mapper/vg-lv`

## 4.3 Partitioning tools

### 4.3.1 fdisk

Use `fdisk` for MBR and basic GPT tasks.

Example:

```bash
sudo fdisk /dev/sdb
```

Interactive actions commonly used:
- `p` print partition table.
- `n` new partition.
- `d` delete partition.
- `t` change partition type.
- `w` write changes.
- `q` quit without saving.

### 4.3.2 gdisk

Use `gdisk` for GPT-focused management.

```bash
sudo gdisk /dev/sdb
```

Useful on large disks and UEFI-oriented systems.

### 4.3.3 parted

`parted` is convenient for scripting and large disk work.

Example:

```bash
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 1MiB 100%
```

Best practices:
- Align partitions properly.
- Document partition purpose.
- Re-read partition table after changes when needed.

## 4.4 Mounting and unmounting

Mount a file system:

```bash
sudo mount /dev/sdb1 /mnt/data
```

Unmount a file system:

```bash
sudo umount /mnt/data
sudo umount /dev/sdb1
```

Check mounts:

```bash
mount
findmnt
findmnt /mnt/data
```

Common causes of unmount failure:
- Open files.
- Current directory inside mount point.
- Active process using the filesystem.

Find blockers:

```bash
lsof +f -- /mnt/data
fuser -vm /mnt/data
```

## 4.5 /etc/fstab

`/etc/fstab` defines persistent mounts.

Example:

```fstab
UUID=1234-ABCD /data ext4 defaults,noatime 0 2
UUID=5678-EFGH /boot/efi vfat umask=0077 0 1
/dev/mapper/vgdata-lvbackup /backup xfs defaults 0 0
```

Field meanings:
1. Device, UUID, LABEL, or path.
2. Mount point.
3. File system type.
4. Mount options.
5. Dump field.
6. fsck order.

Test before reboot:

```bash
sudo mount -a
```

Use UUIDs rather than raw device names when possible.

## 4.6 Disk usage tools

Check free space:

```bash
df -h
df -i
```

Check directory usage:

```bash
du -sh /var/log
du -xh --max-depth=1 / | sort -h
```

Interpretation:
- `df` shows filesystem-level usage.
- `du` shows file and directory-level usage.
- `df -i` shows inode exhaustion, which can fill a filesystem even if bytes remain.

---

## 4.16 Swap management

Check swap:

```bash
swapon --show
free -h
```

Create a swap file:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Persist in `/etc/fstab`:

```fstab
/swapfile none swap sw 0 0
```

Adjust swappiness temporarily:

```bash
sudo sysctl vm.swappiness=10
```

## 4.17 Storage troubleshooting

Common problems:
- Filesystem full.
- Inodes exhausted.
- Wrong UUID in `fstab`.
- Array degraded.
- Snapshot full.
- Rescan needed after hypervisor disk growth.
- Multipath or SAN issues.

Useful commands:

```bash
lsblk -f
blkid
df -h
df -i
pvs
vgs
lvs
cat /proc/mdstat
journalctl -b | grep -i -E "mount|xfs|ext4|md|lvm"
```

## 4.18 Safe storage change workflow

1. Identify the exact device.
2. Confirm backups or snapshots exist.
3. Confirm the current mount and filesystem.
4. Confirm application impact.
5. Apply the change.
6. Validate with `lsblk`, `df`, and test I/O.
7. Persist in `fstab` or array config if needed.
8. Document it.

---

## 12.3 Disk expansion checklist

- Confirm backup or snapshot exists.
- Confirm hypervisor or cloud disk is enlarged.
- Rescan device if needed.
- Verify new size in `lsblk`.
- Expand partition if needed.
- Expand PV if LVM is used.
- Extend LV.
- Grow filesystem.
- Validate with `df -h`.
- Document the change.

---

## 13.4 Storage commands reference

- `lsblk`
- `lsblk -f`
- `blkid`
- `fdisk -l`
- `parted -l`
- `mount`
- `findmnt`
- `mount /dev/sdb1 /mnt/data`
- `umount /mnt/data`
- `df -h`
- `df -i`
- `du -sh /path`
- `du -xh --max-depth=1 /path`
- `fuser -vm /mountpoint`
- `lsof +f -- /mountpoint`
- `pvs`
- `vgs`
- `lvs`
- `pvcreate /dev/sdb1`
- `vgcreate vgdata /dev/sdb1`
- `lvcreate -L 50G -n lvdata vgdata`
- `lvextend -L +10G /dev/vgdata/lvdata`
- `lvextend -r -L +10G /dev/vgdata/lvdata`
- `resize2fs /dev/vgdata/lvdata`
- `xfs_growfs /mountpoint`
- `mdadm --detail /dev/md0`
- `cat /proc/mdstat`
- `swapon --show`
- `free -h`

---

## B.4 Storage quick reminders
- Verify the exact disk before partitioning or formatting.
- Prefer UUIDs in mount configuration.
- Use `findmnt` to confirm active mount relationships.
- Test `mount -a` after editing `fstab`.
- Keep separate filesystems for fast-growing logs or data where sensible.
- Use `df -i` when users report full disks but bytes remain.
- Grow storage carefully and reduce it only with complete understanding.
- Never attempt to shrink XFS.
- Monitor LVM snapshot space closely.
- Watch `/proc/mdstat` during RAID rebuilds.
- Replace failed RAID members with the correct workflow.
- Document device naming changes on cloud and virtual platforms.
- Verify hypervisor disk expansion before guest-side changes.
- Keep rescue media or remote console access for storage work.
- Treat `dd` as a last-resort precision tool.

---

### Storage inspection examples
```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID
blkid -o full
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
pvs -o +pv_used
vgs -o +vg_free
lvs -a -o +devices
```

---

## B.17 Mini runbook: disk full
1. Confirm filesystem and mountpoint with `df -h`.
2. Check inode usage with `df -i`.
3. Identify largest directories with `du`.
4. Check logs and caches.
5. Check deleted-open files with `lsof | grep deleted`.
6. Clean safely.
7. Expand storage if needed.
8. Add monitoring or retention fixes.
9. Validate application recovery.
10. Document the root contributor.

---

### Storage cluster
- `lsblk -f`
- `blkid`
- `findmnt`
- `pvs`
- `vgs`
- `lvs`
- `cat /proc/mdstat`

---

## B.27 More storage examples
```bash
partprobe /dev/sdb
udevadm settle
pvresize /dev/sdb2
lvresize -l +100%FREE /dev/vgdata/lvdata
mdadm --examine /dev/sdb1
```
- `partprobe` helps the kernel notice partition table changes.
- `udevadm settle` can help after device events.
- `pvresize` is needed after enlarging an LVM-backed partition or disk.
- `+100%FREE` can consume all free extents in a volume group.
- RAID metadata examination helps identify members during recovery.
- Always verify whether your array uses partition members or whole disks.
- Name storage consistently so emergency work is easier.
- Prefer labels and documentation for critical arrays and LVM groups.
- Watch rebuild time and performance impact during RAID recovery.
- Keep spare capacity planning realistic, not theoretical.
