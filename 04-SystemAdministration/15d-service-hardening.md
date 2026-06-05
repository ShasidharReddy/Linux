# Service Hardening

← Back to [15-system-hardening.md](./15-system-hardening.md)

Firewall policy, intrusion controls, file permissions, onboarding checklists, and service-specific hardening reminders.

---

## 11.4 Firewall basics

Firewalls enforce network exposure policy.

Common tools:
- `nftables`
- `iptables`
- `firewalld`
- `ufw`

### 11.4.1 firewalld examples

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 11.4.2 ufw examples

```bash
sudo ufw status verbose
sudo ufw allow OpenSSH
sudo ufw allow 443/tcp
sudo ufw enable
```

### 11.4.3 nftables note

Modern Linux distributions increasingly prefer `nftables` as the underlying firewall framework.
Understand whether your host uses direct `nft` rules or a higher-level frontend.

## 11.6 AppArmor

AppArmor is another Linux security module used widely on Ubuntu and SUSE systems.

Check status:

```bash
sudo aa-status
```

Modes include:
- enforce
- complain
- disable

Profile management examples:

```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
```

Operational practice:
- Use profiles to constrain high-risk services.
- Investigate denials rather than disabling protection broadly.

## 11.7 auditd

`auditd` provides security auditing of system events.

Common service commands:

```bash
sudo systemctl enable --now auditd
systemctl status auditd
```

Useful tools:

```bash
sudo auditctl -l
sudo ausearch -k passwd_changes
sudo aureport -au
```

Example rule monitoring `/etc/passwd`:

```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

Persist rules according to your distribution's audit framework files.

## 11.8 fail2ban

`fail2ban` can ban hosts showing repeated malicious behavior, often from logs.

Common use case:
- Repeated SSH login failures.

Basic flow:
- Install fail2ban.
- Enable the `sshd` jail.
- Tune bantime and retry policy.

Typical local config file:

```text
/etc/fail2ban/jail.local
```

Example snippet:

```ini
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
bantime = 1h
findtime = 10m
```

Check status:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## 11.12 File permissions and ownership

Essential commands:

```bash
chmod 640 file
chmod 750 dir
chown root:root file
chown -R appuser:appgroup /opt/myapp
umask
```

Security concepts:
- Least privilege on files and directories.
- Protect private keys with strict permissions.
- Use group ownership intentionally.
- Review world-writable paths.

Check for world-writable directories:

```bash
find / -xdev -type d -perm -0002 2>/dev/null
```

Check for SUID binaries:

```bash
find / -xdev -perm -4000 -type f 2>/dev/null
```

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

## 12.1 New server checklist

- Confirm OS version and support window.
- Update packages.
- Configure hostname and time sync.
- Create admin accounts.
- Install SSH keys.
- Disable root SSH login.
- Configure sudo access.
- Configure firewall.
- Configure logging and central forwarding.
- Install monitoring agent if used.
- Verify disk layout and mount points.
- Confirm backups are in scope.
- Document the host role.

---

## 13.11 Hardening commands reference

- `systemctl list-unit-files --type=service`
- `systemctl disable --now <service>`
- `sshd -t`
- `systemctl reload sshd`
- `firewall-cmd --list-all`
- `firewall-cmd --reload`
- `ufw status verbose`
- `ufw allow OpenSSH`
- `getenforce`
- `sestatus`
- `setenforce 0`
- `restorecon -Rv /path`
- `aa-status`
- `aa-enforce <profile>`
- `auditctl -l`
- `auditctl -w /etc/passwd -p wa -k passwd_changes`
- `ausearch -k passwd_changes`
- `fail2ban-client status`
- `chage -l <user>`
- `visudo`
- `find / -xdev -type d -perm -0002 2>/dev/null`
- `find / -xdev -perm -4000 -type f 2>/dev/null`
- `ss -tulpn`

---
