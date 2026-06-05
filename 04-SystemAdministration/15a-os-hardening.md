# OS Hardening

← Back to [15-system-hardening.md](./15-system-hardening.md)

Baseline host hardening, patch hygiene, kernel settings, and secure-operating habits.

---

## 11.1 Hardening principles

- Minimize exposed services.
- Use least privilege.
- Keep systems patched.
- Log and audit important actions.
- Encrypt sensitive data.
- Restrict remote access.
- Standardize configuration.
- Review regularly.

## 11.2 Disable unnecessary services

List services:

```bash
systemctl list-unit-files --type=service
systemctl list-units --type=service
```

Disable unused services:

```bash
sudo systemctl disable --now avahi-daemon
sudo systemctl disable --now cups
```

Mask if you want to prevent accidental activation:

```bash
sudo systemctl mask telnet.socket
```

Before disabling anything, confirm business need and dependency impact.

## 11.13 Package and patch hygiene

Hardening includes staying current.

Routine:
- Apply security updates promptly.
- Remove unused packages.
- Review installed services and listening ports.
- Monitor end-of-life operating systems.

Inventory listening ports:

```bash
ss -tulpn
```

## 11.14 Kernel and network hardening quick wins

Examples:
- Enable SYN cookies where appropriate.
- Disable IP forwarding unless needed.
- Restrict core dumps if policy requires.
- Disable uncommon protocols or modules if not needed.

Potential sysctl examples:

```conf
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
kernel.kptr_restrict = 2
fs.suid_dumpable = 0
```

Tune based on environment and application needs.

## 11.15 Security monitoring essentials

At minimum, monitor:
- Failed SSH logins.
- sudo activity.
- new listening ports.
- service failures.
- audit events for critical files.
- firewall drops on sensitive hosts.
- SELinux or AppArmor denials.

## 11.16 Hardening workflow

1. Build from a minimal OS image.
2. Patch fully.
3. Remove unnecessary packages and services.
4. Harden SSH.
5. Configure firewall policy.
6. Enable logging and auditing.
7. Enable MAC controls like SELinux or AppArmor.
8. Enforce authentication and password policy.
9. Validate application functionality.
10. Scan and review regularly.

## 11.17 Hardening best practices

- Use configuration management for consistency.
- Avoid one-off manual changes without records.
- Test hardening baselines on staging.
- Keep emergency access procedures documented.
- Harden but do not blind yourself operationally.
- Prefer measured policy over copy-paste checklists.

---

## 13.14 Final advice

Linux system administration is a craft built on repetition, caution, and disciplined observation.
The best administrators are not the ones who memorize the most commands.
They are the ones who:
- verify before changing,
- automate after understanding,
- log before guessing,
- back up before risking,
- and document after succeeding.

Keep this guide as a practical reference.
Practice each section in a lab.
Then carry the habits into production carefully and consistently.

---

## B.11 Hardening quick reminders
- Remove what you do not need.
- Restrict who can log in.
- Prefer keys over passwords for SSH.
- Keep a firewall enabled with explicit policy.
- Use SELinux or AppArmor rather than disabling them casually.
- Audit privileged actions.
- Review sudo scope regularly.
- Harden permissions on secrets and keys.
- Scan for unexpected listening ports.
- Patch regularly.
- Use MFA where possible.
- Monitor failed logins and privilege escalation.
- Standardize secure defaults across hosts.
- Test hardening controls with applications.
- Document break-glass procedures.

---

### Hardening inspection examples
```bash
ss -tulpn
find /etc/sudoers.d -type f -maxdepth 1 -print -exec cat {} \;
getenforce
sudo aa-status
sudo fail2ban-client status sshd
```

---

## B.13 Command outcome interpretation guide
- If `df -h` shows 100 percent usage, confirm whether logs, backups, or deleted-open files are the cause.
- If `df -i` shows 100 percent inode usage, investigate directories with many small files.
- If `systemctl status` shows exit code failures, correlate with journal messages and recent config edits.
- If `vmstat` shows steady swap activity, investigate memory pressure before performance collapses.
- If `iostat` shows high `await`, investigate storage latency, queueing, or failing hardware.
- If `journalctl -k` shows I/O errors, prioritize data safety before routine performance tuning.
- If `ausearch` shows repeated denials, adjust policy carefully instead of disabling security globally.
- If `ss -tulpn` shows unexpected listeners, verify ownership and exposure immediately.
- If `sar` history shows regression after a deployment, compare packages, configs, and workload changes.
- If `mdadm` shows a degraded array, replace or re-add devices promptly and monitor rebuild.

---

## B.14 Role-based learning path

### Beginner sysadmin focus
- Learn package updates.
- Learn `systemctl` and `journalctl`.
- Learn `ps`, `top`, and signals.
- Learn `df`, `du`, `mount`, and `fstab`.
- Learn basic cron usage.
- Learn backup basics with `rsync` and `tar`.

### Intermediate sysadmin focus
- Learn LVM thoroughly.
- Learn `vmstat`, `iostat`, `sar`, and `mpstat`.
- Learn journald and logrotate deeply.
- Learn custom systemd units and timers.
- Learn sysctl tuning and module management.
- Learn SSH and firewall hardening.

### Advanced sysadmin focus
- Learn RAID recovery workflows.
- Learn snapshot-based backup strategies.
- Learn SELinux or AppArmor in depth.
- Learn auditd tuning and evidence collection.
- Learn kernel boot troubleshooting and crash handling.
- Learn standardization through automation and policy.

---

## B.15 Production anti-patterns to avoid
- Editing critical config files without backups.
- Running destructive commands against unverified devices.
- Using `chmod 777` as a fix.
- Disabling SELinux or AppArmor as a first response.
- Leaving cron jobs undocumented.
- Relying on local logs only.
- Ignoring inode exhaustion.
- Using `kill -9` reflexively.
- Treating snapshots as backups forever.
- Updating production blindly without review.
- Mixing package sources casually.
- Running services as root unnecessarily.
- Leaving old unused packages and daemons installed.
- Trusting backups that have never been restored.
- Making emergency changes without later cleanup or documentation.

---

## B.34 More hardening examples
```bash
passwd -S username
faillock --user username
chage -M 90 -W 7 username
nft list ruleset
```
- Account lockout tooling differs across distributions.
- Password aging should align with organization policy.
- Firewall policy should be explicit, minimal, and reviewed.
- New services should not assume public exposure.
- Bastion models often reduce direct admin surface area.
- MFA and centralized identity improve account control.
- Review dormant accounts regularly.
- Ensure security controls are logged and monitored.
- Validate that hardening does not break backup, monitoring, or automation.
- Security posture is a process, not a one-time checklist.

---

## B.35 Extended glossary
- Artifact: a build output or package ready for deployment.
- AVC: access vector cache message from SELinux denials.
- Cgroup: kernel mechanism for grouping and controlling resources.
- Daemon: background service process.
- Filesystem UUID: unique identifier for a filesystem instance.
- Initramfs: early boot image used before the real root filesystem is mounted.
- Inode: metadata structure representing a file.
- Journal: structured log storage used by systemd.
- LUN: logical unit presented from shared storage.
- Mountpoint: directory where a filesystem is attached.
- OOM: out of memory event.
- PV: physical volume in LVM.
- VG: volume group in LVM.
- LV: logical volume in LVM.
- Run queue: tasks ready to run on CPU.
- SELinux context: label used for mandatory access control decisions.
- Snapshot: point-in-time copy using storage or filesystem mechanisms.
- Socket activation: service starts when traffic arrives on a systemd-managed socket.
- Swappiness: kernel tendency to move memory pages toward swap.
- Unit: a systemd configuration object such as a service or timer.
- Zombie: terminated process not yet reaped by its parent.

---

<a id="nfs-network-file-system"></a>
