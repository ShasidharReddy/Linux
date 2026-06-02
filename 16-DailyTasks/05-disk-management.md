# Disk and Storage Management

This guide covers filesystem usage, cleanup, LVM workflows, and storage expansion tasks.

## 5.1 Checking disk usage

```bash
df -hT
```

```bash
df -ih
```

```bash
sudo du -xhd1 / | sort -h
```

### ncdu for interactive analysis

```bash
sudo ncdu /
```

## 5.2 Finding large files

```bash
sudo find / -xdev -type f -size +1G -printf '%10s %p
' 2>/dev/null | sort -nr | head -50
```

```bash
sudo find /var/log -type f -size +100M -ls
```

```bash
sudo lsof +L1
```

### Top 20 directories under `/var`

```bash
sudo du -xhd1 /var | sort -h | tail -20
```

## 5.3 Cleaning up disk space safely

### Debian or Ubuntu package cache

```bash
sudo apt clean
sudo apt autoremove -y
```

### RHEL family package cache

```bash
sudo dnf clean all
```

### Journald cleanup

```bash
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=500M
```

### Remove old kernels on Debian or Ubuntu

```bash
dpkg -l 'linux-image*' | awk '/^ii/ { print $2 }'
```

```bash
sudo apt autoremove --purge -y
```

### Remove old kernels on RHEL family

```bash
sudo dnf remove $(dnf repoquery --installonly --latest-limit=-2 -q)
```

### Clean temp files

```bash
sudo find /tmp -xdev -type f -mtime +7 -delete
sudo find /var/tmp -xdev -type f -mtime +7 -delete
```

### Truncate oversized logs after copying if needed

```bash
sudo cp /var/log/myapp.log /var/log/myapp.log.bak.$(date +%F-%H%M%S)
sudo truncate -s 0 /var/log/myapp.log
```

### Remove orphaned Docker objects

```bash
docker system df
docker system prune -f
docker image prune -af
```

## 5.4 LVM extend and resize operations

### Check current LVM layout

```bash
sudo pvs
sudo vgs
sudo lvs -a -o +devices
```

### Extend logical volume and filesystem in one step

```bash
sudo lvextend -r -L +20G /dev/vgdata/lvdata
```

### Extend to all free space

```bash
sudo lvextend -r -l +100%FREE /dev/vgdata/lvdata
```

### XFS grow manually

```bash
sudo lvextend -L +10G /dev/vgdata/lvlogs
sudo xfs_growfs /var/log
```

### ext4 grow manually

```bash
sudo lvextend -L +10G /dev/vgdata/lvapp
sudo resize2fs /dev/vgdata/lvapp
```

## 5.5 Adding a new disk

### Discover disk

```bash
lsblk
sudo fdisk -l
```

### Partition disk with GPT

```bash
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 0% 100%
```

### Format and mount

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkdir -p /data
sudo mount /dev/sdb1 /data
sudo blkid /dev/sdb1
```

### Add to `/etc/fstab`

```bash
UUID=$(sudo blkid -s UUID -o value /dev/sdb1)
echo "UUID=$UUID /data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
sudo mount -a
findmnt /data
```

### Add new disk to LVM

```bash
sudo pvcreate /dev/sdb1
sudo vgextend vgdata /dev/sdb1
sudo vgs
```

## 5.6 Filesystem checks

```bash
sudo fsck -N /dev/sdb1
sudo tune2fs -l /dev/sdb1 | egrep 'Filesystem volume name|Block count|Reserved block count|Mount count'
xfs_info /var || true
```

## 5.7 Inode exhaustion checks

```bash
df -ih
sudo find /var -xdev -type f | cut -d/ -f1-3 | sort | uniq -c | sort -nr | head -20
```

## 5.8 Storage troubleshooting workflow

- Check `df -hT`.
- Check `df -ih`.
- Check `du -xhd1 /`.
- Check deleted but open files with `lsof +L1`.
- Check logs and package caches.
- Check container runtime directories.
- Check backup leftovers.
- Check snapshots if using LVM or storage platform.

## 5.9 Disk management one-liners

```bash
sudo find / -xdev -type f -printf '%s %p
' 2>/dev/null | sort -nr | head -20 | numfmt --to=iec
```

```bash
sudo du -xah /var | sort -hr | head -30
```

---
