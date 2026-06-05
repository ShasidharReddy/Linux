# Access Control

← Back to [15-system-hardening.md](./15-system-hardening.md)

SSH, passwords, PAM, sudo, and authentication troubleshooting guidance.

---

## 11.3 SSH hardening

The SSH daemon is a primary admin entry point.
Treat it carefully.

Config file:

```text
/etc/ssh/sshd_config
```

Typical hardening settings:

```conf
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers adminuser opsuser
```

After changes:

```bash
sudo sshd -t
sudo systemctl reload sshd
```

Recommended practices:
- Use SSH keys.
- Disable direct root login.
- Restrict user access.
- Use MFA where possible.
- Consider moving SSH to a management network or bastion.

## 11.9 Password policies

Strong password policy matters even with SSH keys because local accounts, sudo, and some services may still use passwords.

Areas to review:
- Password complexity.
- Password aging.
- Account lockout.
- MFA where possible.

On many systems, policy is influenced by:
- `/etc/login.defs`
- PAM configuration under `/etc/pam.d/`
- tools like `chage`

Examples:

```bash
sudo chage -l username
sudo chage -M 90 -W 14 -I 7 username
```

## 11.10 PAM

PAM stands for Pluggable Authentication Modules.
It controls authentication, account policy, session policy, and password handling.

Common PAM files:
- `/etc/pam.d/sshd`
- `/etc/pam.d/system-auth`
- `/etc/pam.d/common-auth`
- `/etc/pam.d/sudo`

PAM module categories:
- `auth`
- `account`
- `password`
- `session`

Use caution.
Bad PAM changes can lock out administrators.

## 11.11 Sudo hardening

Use `sudo` instead of direct root usage where practical.

Inspect permissions safely with:

```bash
sudo visudo
sudo visudo -f /etc/sudoers.d/admins
```

Best practices:
- Use least privilege rules.
- Prefer group-based sudo where practical.
- Log sudo usage.
- Require MFA if your environment supports it.
- Avoid overly broad `NOPASSWD` rules.

## 13.12 One-line troubleshooting cookbook

- Service fails to start: `systemctl status <svc> && journalctl -u <svc> -b`
- Disk full: `df -h && du -xh --max-depth=1 / | sort -h | tail`
- Memory pressure: `free -h && vmstat 1 5 && journalctl -k | grep -i oom`
- CPU spike: `uptime && mpstat -P ALL 1 5 && ps -eo pid,%cpu,cmd --sort=-%cpu | head`
- Port conflict: `ss -tulpn | grep :80`
- Filesystem issue: `dmesg -T | tail -100 && lsblk -f && mount`
- RAID degraded: `cat /proc/mdstat && mdadm --detail /dev/md0`
- LVM state: `pvs && vgs && lvs`
- SSH failures: `journalctl -u sshd --since -1h | grep -i failed`
- SELinux issue: `ausearch -m avc -ts recent`

---

## B.19 Mini runbook: failed SSH access
1. Confirm network reachability.
2. Confirm sshd service status.
3. Confirm firewall policy.
4. Review auth logs.
5. Check user account lock or shell.
6. Check permissions on home and `.ssh`.
7. Check SELinux or AppArmor denials.
8. Validate `sshd_config` syntax.
9. Reload or restart carefully.
10. Confirm restored access before ending maintenance.

---

### Security cluster
- `ss -tulpn`
- `getenforce`
- `aa-status`
- `fail2ban-client status`
- `ausearch -m avc -ts recent`
- `journalctl -u sshd --since -1h`

---
