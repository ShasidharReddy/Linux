# Permission Issues

## 8.1 Permission denied categories

- UNIX mode bits.
- Ownership mismatch.
- Parent directory execute bit missing.
- ACL override.
- SELinux or AppArmor block.
- Capability missing.
- Read-only mount.
- NFS root squash.

## 8.2 Basic checks

```bash
namei -l /path/to/file
ls -ld /path /path/to /path/to/file
id
getfacl /path/to/file 2>/dev/null
mount | grep ' /path '
```

## 8.3 Remember directory execute bit

To access a file by name inside a directory, the process needs execute permission on the directory.

## 8.4 `namei` is extremely useful

Example:

```bash
namei -l /var/www/html/index.html
```

This reveals ownership and modes for every path component.

## 8.5 Ownership mismatch

Common symptoms:

- Service can read config but cannot write PID or socket file.
- App deploy user created files a runtime user cannot access.

Fix carefully with `chown` on the right path only.

## 8.6 ACL conflicts

ACLs can override assumptions based on mode bits.

Check:

```bash
getfacl /path/to/file
```

## 8.7 SELinux basics

Modes:

- Enforcing.
- Permissive.
- Disabled.

Check context:

```bash
ls -Z /path/to/file
ps -eZ | head
getenforce
```

Restore default contexts:

```bash
restorecon -Rv /var/www/html
```

## 8.8 AppArmor basics

Check status:

```bash
aa-status 2>/dev/null
journalctl -k | grep -i apparmor
```

## 8.9 Capabilities

Use capabilities instead of root when possible.

Check file capabilities:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null | head -50
```

Common example:

- Binding to ports below 1024 without root using `cap_net_bind_service`.

Set example:

```bash
setcap 'cap_net_bind_service=+ep' /usr/local/bin/myapp
```

## 8.10 Read-only filesystems

A permission-looking issue may really be mount state.

Check:

```bash
mount | grep ' / '
dmesg | tail -100
```

## 8.11 NFS root squash

On NFS, root may be mapped to nobody.

Symptoms:

- Root cannot write.
- Ownership behaves unexpectedly.

Review export options on the server.

## 8.12 Sudo and path confusion

Check:

- Effective user.
- `sudo -u` behavior.
- environment sanitization.
- home directory assumptions.

## 8.13 Sticky bit issues

Example location:

- `/tmp`

The sticky bit allows only file owner or root to delete entries.

## 8.14 Permission issue checklist

- Check user and groups.
- Check path component permissions.
- Check ownership.
- Check ACLs.
- Check SELinux or AppArmor.
- Check capabilities.
- Check mount state.
- Check remote filesystem semantics.

---
