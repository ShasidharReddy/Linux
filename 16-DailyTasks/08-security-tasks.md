# Security Tasks

This guide covers patching, access review, firewall work, certificate checks, and security hygiene tasks.

## 8.1 Checking failed login attempts

```bash
sudo lastb | head -50
```

```bash
sudo grep -Ei 'failed password|invalid user|authentication failure' /var/log/auth.log /var/log/secure 2>/dev/null | tail -100
```

## 8.2 Checking successful logins

```bash
sudo grep 'Accepted ' /var/log/auth.log /var/log/secure 2>/dev/null | tail -100
```

## 8.3 Updating packages and security patches

### Debian or Ubuntu

```bash
sudo apt update
sudo apt list --upgradable
sudo unattended-upgrade --dry-run -d || true
sudo apt upgrade -y
```

### RHEL family

```bash
sudo dnf check-update || true
sudo dnf updateinfo list security
sudo dnf upgrade --security -y
```

## 8.4 Checking open ports

```bash
ss -tulpn
```

```bash
sudo lsof -i -P -n | grep LISTEN
```

## 8.5 Firewall rule management

### UFW

```bash
sudo ufw status numbered
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
sudo ufw delete 3
```

### firewalld

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### nftables quick view

```bash
sudo nft list ruleset
```

## 8.6 Certificate renewal with Let's Encrypt

### Test renewal

```bash
sudo certbot renew --dry-run
```

### Actual renewal via systemd timer or cron

```bash
systemctl list-timers | grep certbot || true
sudo certbot renew
```

### Reload web server after renewal if needed

```bash
sudo systemctl reload nginx
sudo systemctl reload apache2 || sudo systemctl reload httpd
```

## 8.7 Vulnerability scanning

### Package-level checks

```bash
sudo apt list --upgradable 2>/dev/null | grep -i security || true
sudo dnf updateinfo list security || true
```

### File permissions review

```bash
sudo find / -xdev -type f -perm -0002 -ls 2>/dev/null
```

### SUID and SGID review

```bash
sudo find / -xdev \( -perm -4000 -o -perm -2000 \) -type f -ls 2>/dev/null
```

### Check listening services exposed externally

```bash
ss -tulpn | awk 'NR==1 || $5 !~ /127.0.0.1|::1/'
```

### Lynis example

```bash
sudo lynis audit system
```

## 8.8 Inspect security services

```bash
systemctl status fail2ban --no-pager || true
fail2ban-client status || true
systemctl status auditd --no-pager || true
```

## 8.9 SELinux and AppArmor quick checks

```bash
getenforce || true
sudo ausearch -m AVC,USER_AVC -ts recent || true
sudo aa-status || true
```

## 8.10 Security hardening reminders

- Remove unused packages.
- Disable unused services.
- Enforce MFA on jump hosts.
- Rotate keys and credentials.
- Restrict sudo by command where possible.
- Monitor privileged group membership.
- Forward auth logs centrally.
- Patch kernel and browser-exposed apps quickly.

## 8.11 Security one-liners

```bash
sudo awk -F: '($2 == "" ) {print "Empty password field:", $1}' /etc/shadow
```

```bash
sudo grep -R "NOPASSWD" /etc/sudoers /etc/sudoers.d 2>/dev/null
```

```bash
sudo find /home -maxdepth 2 -name authorized_keys -mtime -30 -ls
```

---
