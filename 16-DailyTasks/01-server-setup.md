# Server Setup and Provisioning

This guide covers day-0 provisioning, baseline hardening, identity, networking, and initial server readiness tasks.

## 1.1 Initial server setup checklist

### Goals

- Standardize new server onboarding.
- Reduce security drift.
- Ensure time, identity, access, and networking are correct.
- Prepare the server for monitoring and configuration management.

### Day-0 checklist

- Set hostname.
- Set timezone.
- Enable NTP.
- Create admin user.
- Harden SSH.
- Disable password login if policy allows.
- Install updates.
- Configure firewall.
- Install baseline packages.
- Configure logging and monitoring agent.
- Register DNS if needed.
- Add asset tags or CMDB entries.
- Verify backups are scheduled.
- Verify security agent is installed.
- Confirm storage layout.
- Confirm swap configuration.
- Confirm cloud-init finished successfully.

### Identify distro and version

```bash
cat /etc/os-release
uname -r
hostnamectl
```

### Set hostname

```bash
sudo hostnamectl set-hostname server01.example.com
hostnamectl status
cat /etc/hostname
```

### Update `/etc/hosts` when needed

```bash
sudo tee -a /etc/hosts >/dev/null <<'EOF'
127.0.0.1 localhost
127.0.1.1 server01.example.com server01
EOF
```

### Set timezone

```bash
timedatectl list-timezones | grep -i kolkata
sudo timedatectl set-timezone Asia/Kolkata
timedatectl
```

### Enable NTP synchronization

```bash
sudo timedatectl set-ntp true
timedatectl status
chronyc tracking || true
systemctl status systemd-timesyncd --no-pager || true
systemctl status chronyd --no-pager || true
```

### Install baseline packages on Debian or Ubuntu

```bash
sudo apt update
sudo apt install -y \
  vim curl wget git rsync unzip zip jq \
  net-tools dnsutils telnet traceroute \
  tcpdump htop iotop sysstat lsof \
  gnupg ca-certificates ufw fail2ban \
  logrotate cron bash-completion
```

### Install baseline packages on RHEL or Rocky or AlmaLinux

```bash
sudo dnf install -y \
  vim curl wget git rsync unzip zip jq \
  net-tools bind-utils telnet traceroute \
  tcpdump htop iotop sysstat lsof \
  gnupg2 ca-certificates firewalld fail2ban \
  logrotate cronie bash-completion
```

### Update packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

```bash
sudo dnf check-update || true
sudo dnf upgrade -y
```

### Verify reboots required

```bash
[ -f /var/run/reboot-required ] && cat /var/run/reboot-required || echo "No reboot required"
needs-restarting -r || true
```

### Create admin user

```bash
sudo useradd -m -s /bin/bash opsadmin
sudo passwd opsadmin
sudo usermod -aG sudo opsadmin
id opsadmin
getent group sudo
```

### RHEL family sudo group

```bash
sudo usermod -aG wheel opsadmin
id opsadmin
getent group wheel
```

### Create `.ssh` directory securely

```bash
sudo install -d -m 700 -o opsadmin -g opsadmin /home/opsadmin/.ssh
sudo touch /home/opsadmin/.ssh/authorized_keys
sudo chown opsadmin:opsadmin /home/opsadmin/.ssh/authorized_keys
sudo chmod 600 /home/opsadmin/.ssh/authorized_keys
```

### SSH hardening checklist

- Disable root login.
- Disable password auth if key auth is enforced.
- Restrict allowed users or groups.
- Set idle timeout.
- Limit authentication attempts.
- Use modern ciphers and MACs if required by policy.
- Change default port only if operationally justified.
- Send logs to central logging.
- Add banner if required by compliance.

### Example `/etc/ssh/sshd_config` baseline

```conf
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding no
UseDNS no
AllowUsers opsadmin deployer
Subsystem sftp /usr/lib/openssh/sftp-server
```

### Validate SSH config before reload

```bash
sudo sshd -t
sudo systemctl reload sshd || sudo systemctl reload ssh
sudo systemctl status sshd --no-pager || sudo systemctl status ssh --no-pager
```

### Firewall quick start with UFW

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

### Firewall quick start with firewalld

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

### Kernel and sysctl baseline

```bash
sudo tee /etc/sysctl.d/99-ops-baseline.conf >/dev/null <<'EOF'
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
vm.swappiness = 10
fs.file-max = 2097152
EOF
sudo sysctl --system
```

### Validate DNS resolution

```bash
resolvectl status || systemd-resolve --status || cat /etc/resolv.conf
getent hosts example.com
```

### Verify storage layout

```bash
lsblk -f
findmnt
sudo pvs || true
sudo vgs || true
sudo lvs || true
```

### Check swap

```bash
swapon --show
free -h
cat /proc/swaps
```

### Configure audit trail basics

```bash
sudo systemctl enable --now rsyslog || true
sudo systemctl enable --now auditd || true
sudo systemctl status rsyslog --no-pager || true
sudo systemctl status auditd --no-pager || true
```

### Register with monitoring system

- Add node to Prometheus inventory.
- Install node exporter.
- Install log shipper.
- Install EDR if required.
- Verify alert labels and environment tags.

### Provisioning validation checklist

- `hostnamectl` shows expected hostname.
- `timedatectl` shows correct timezone.
- NTP synchronized.
- SSH login with key works.
- Root login blocked.
- Firewall active.
- Packages updated.
- Monitoring agent reporting.
- Disk layout documented.
- Backups configured.

## 1.2 User account provisioning

### Standard onboarding steps

- Create user.
- Create primary group if required.
- Add supplementary groups.
- Set password expiration policy.
- Deploy SSH key.
- Configure sudo if required.
- Verify shell and home directory.
- Record owner and purpose.

### Create a normal user

```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
id alice
getent passwd alice
```

### Create service account without interactive shell

```bash
sudo useradd -r -M -s /usr/sbin/nologin appsvc
id appsvc
getent passwd appsvc
```

### Create user with explicit UID and group

```bash
sudo groupadd -g 1050 developers
sudo useradd -m -u 1050 -g developers -s /bin/bash bob
id bob
```

### Set password aging

```bash
sudo chage -M 90 -m 1 -W 7 alice
sudo chage -l alice
```

### Lock or unlock account

```bash
sudo usermod -L alice
sudo usermod -U alice
sudo passwd -S alice
```

### Expire password immediately

```bash
sudo chage -d 0 alice
```

## 1.3 SSH key deployment

### Generate an Ed25519 key on admin workstation

```bash
ssh-keygen -t ed25519 -C "alice@example.com"
```

### Deploy public key with `ssh-copy-id`

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server01.example.com
```

### Manually deploy SSH key

```bash
sudo install -d -m 700 -o alice -g alice /home/alice/.ssh
sudo tee -a /home/alice/.ssh/authorized_keys >/dev/null <<'EOF'
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExamplePublicKeyData alice@example.com
EOF
sudo chown alice:alice /home/alice/.ssh/authorized_keys
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

### Verify key login

```bash
ssh -o PreferredAuthentications=publickey -o PasswordAuthentication=no alice@server01.example.com 'hostname && id'
```

### Restrict a key to a command

```text
command="/usr/local/bin/backup-runner",no-port-forwarding,no-agent-forwarding,no-pty ssh-ed25519 AAAAC3... backup-key
```

### Rotate SSH keys

- Add new key first.
- Verify login using new key.
- Remove old key.
- Record rotation date.
- Update inventory or password vault notes.

## 1.4 Complete example initial setup script

```bash
#!/usr/bin/env bash
set -euo pipefail

NEW_HOSTNAME="server01.example.com"
ADMIN_USER="opsadmin"
ADMIN_PUBKEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExamplePublicKey opsadmin@example"

if command -v apt >/dev/null 2>&1; then
  PM_UPDATE='apt update'
  PM_INSTALL='apt install -y'
  SUDO_GROUP='sudo'
elif command -v dnf >/dev/null 2>&1; then
  PM_UPDATE='dnf makecache'
  PM_INSTALL='dnf install -y'
  SUDO_GROUP='wheel'
else
  echo "Unsupported package manager"
  exit 1
fi

sudo hostnamectl set-hostname "$NEW_HOSTNAME"
sudo timedatectl set-timezone UTC
sudo timedatectl set-ntp true
sudo bash -c "$PM_UPDATE"
sudo bash -c "$PM_INSTALL vim curl wget git rsync jq htop lsof fail2ban logrotate"

if ! id "$ADMIN_USER" >/dev/null 2>&1; then
  sudo useradd -m -s /bin/bash "$ADMIN_USER"
fi

sudo usermod -aG "$SUDO_GROUP" "$ADMIN_USER"
sudo install -d -m 700 -o "$ADMIN_USER" -g "$ADMIN_USER" "/home/$ADMIN_USER/.ssh"
sudo touch "/home/$ADMIN_USER/.ssh/authorized_keys"
sudo chmod 600 "/home/$ADMIN_USER/.ssh/authorized_keys"
sudo chown "$ADMIN_USER:$ADMIN_USER" "/home/$ADMIN_USER/.ssh/authorized_keys"
if ! sudo grep -qF "$ADMIN_PUBKEY" "/home/$ADMIN_USER/.ssh/authorized_keys"; then
  echo "$ADMIN_PUBKEY" | sudo tee -a "/home/$ADMIN_USER/.ssh/authorized_keys" >/dev/null
fi

SSHD_CONFIG='/etc/ssh/sshd_config'
sudo cp "$SSHD_CONFIG" "${SSHD_CONFIG}.bak.$(date +%F-%H%M%S)"
sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' "$SSHD_CONFIG"
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' "$SSHD_CONFIG"
sudo sshd -t
sudo systemctl reload sshd || sudo systemctl reload ssh

echo "Initial setup completed for $NEW_HOSTNAME"
```

## 1.5 Example user provisioning script

```bash
#!/usr/bin/env bash
set -euo pipefail

USERNAME="${1:?Usage: $0 <username> <pubkey-file>}"
PUBKEY_FILE="${2:?Usage: $0 <username> <pubkey-file>}"

sudo useradd -m -s /bin/bash "$USERNAME" || true
sudo install -d -m 700 -o "$USERNAME" -g "$USERNAME" "/home/$USERNAME/.ssh"
sudo install -m 600 -o "$USERNAME" -g "$USERNAME" "$PUBKEY_FILE" "/home/$USERNAME/.ssh/authorized_keys"
sudo chage -M 90 -m 1 -W 7 "$USERNAME"
id "$USERNAME"
sudo chage -l "$USERNAME"
```

## 1.6 Provisioning one-liners

```bash
hostnamectl; timedatectl; ip -br a; ss -tulpn
```

```bash
id opsadmin && sudo -l -U opsadmin
```

```bash
sudo sshd -T | egrep 'permitrootlogin|passwordauthentication|maxauthtries|clientaliveinterval'
```

```bash
sudo ufw status numbered || sudo firewall-cmd --list-all
```

---
