# Recovery Procedures

## 11.1 Recovery priorities
1. Protect data.
2. Restore service safely.
3. Preserve evidence.
4. Reduce time pressure.
5. Implement durable repair.

## 11.2 Live USB or ISO rescue workflow
1. Boot rescue media.
2. Confirm disks are visible.
3. Identify root, boot, EFI, LVM, and RAID devices.
4. Assemble storage layers.
5. Mount the installed system.
6. Bind mount `dev`, `proc`, and `sys`.
7. `chroot` into the system.
8. Repair bootloader, initramfs, config, or packages.
9. Exit and cleanly unmount.
10. Reboot and validate.

## 11.3 Discover storage layout
```bash
lsblk -f
blkid
pvs
vgs
lvs
cat /proc/mdstat
```

## 11.4 Mount and chroot example
```bash
mount /dev/mapper/vg-root /mnt
mount /dev/sdX1 /mnt/boot
mount /dev/sdY1 /mnt/boot/efi
mount --bind /dev /mnt/dev
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys
chroot /mnt /bin/bash
```

## 11.5 Password reset
Method 1 using rescue shell:

```bash
passwd username
```

Method 2 from GRUB on some systems:

- Edit kernel line.
- Append `rd.break` or `init=/bin/bash` depending on distro and policy.
- Remount root read-write if needed.
- Reset password.
- Restore SELinux contexts if required.

SELinux relabel hint:

```bash
touch /.autorelabel
```

## 11.6 Filesystem repair workflow
- Identify filesystem type.
- Ensure it is unmounted.
- Capture current error messages.
- Run non-destructive checks first if possible.
- Repair.
- Mount read-only for validation if prudent.

## 11.7 Package repair from rescue
Inside chroot:

```bash
apt update && apt install -f
apt reinstall grub-common linux-image-amd64
```

Or:

```bash
dnf reinstall kernel grub2
dracut -f
```

## 11.8 Restoring from backup
Checklist:

- Verify backup age.
- Verify backup completeness.
- Verify target host compatibility.
- Restore to alternate path first if possible.
- Verify ownership, ACLs, and contexts.
- Test before cutover.

## 11.9 Recovering deleted but open files
If a deleted file is still open:

- Prefer service restart or reopen signal.
- In rare cases inspect `/proc/<pid>/fd/`.

## 11.10 Rebuild initramfs and GRUB after recovery
```bash
update-initramfs -u -k all && update-grub
```

Or:

```bash
dracut -f --regenerate-all
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 11.11 Validate after recovery
- Boot succeeds.
- Filesystems mount read-write as expected.
- Critical services start.
- Logs are clean enough.
- Monitoring is green.
- Backups still run.

## 11.12 Recovery checklist
- Protect data.
- Assemble storage layers.
- Mount the system.
- Chroot.
- Repair the minimum required.
- Validate.
- Document exact actions.

---
