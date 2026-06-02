# User Management

This guide covers account lifecycle tasks, SSH access, sudo policy, and auditing user changes.

## 4.1 Creating users

```bash
sudo useradd -m -s /bin/bash dev1
sudo passwd dev1
id dev1
```

### Create multiple users from a file

```bash
while read -r u; do sudo useradd -m -s /bin/bash "$u" || true; done < users.txt
```

## 4.2 Removing users

### Remove account but keep home

```bash
sudo userdel dev1
```

### Remove account and home directory

```bash
sudo userdel -r dev1
```

### Find files owned by a deleted UID

```bash
sudo find / -xdev -uid 1050 -ls 2>/dev/null
```

## 4.3 Password resets

```bash
sudo passwd dev1
sudo chage -d 0 dev1
sudo passwd -S dev1
```

## 4.4 Sudo access management

### Add user to sudo or wheel

```bash
sudo usermod -aG sudo dev1
sudo usermod -aG wheel dev1
```

### Validate sudo rights

```bash
sudo -l -U dev1
id dev1
```

### Use `/etc/sudoers.d`

```bash
echo 'dev1 ALL=(ALL) NOPASSWD:/bin/systemctl,/usr/bin/journalctl' | sudo tee /etc/sudoers.d/dev1-ops
sudo chmod 440 /etc/sudoers.d/dev1-ops
sudo visudo -cf /etc/sudoers
```

## 4.5 SSH key management

### List keys for a user

```bash
sudo ls -la /home/dev1/.ssh
sudo cat /home/dev1/.ssh/authorized_keys
```

### Remove old key safely

```bash
sudo cp /home/dev1/.ssh/authorized_keys /home/dev1/.ssh/authorized_keys.bak.$(date +%F-%H%M%S)
sudo sed -i '/old-key-comment@example.com/d' /home/dev1/.ssh/authorized_keys
```

### Check permissions

```bash
namei -l /home/dev1/.ssh/authorized_keys
```

## 4.6 Auditing user activity

### Who is logged in now

```bash
w
who
users
```

### Last login history

```bash
last -a | head -50
lastlog | head -50
```

### Failed login attempts

```bash
lastb | head -50
```

### Recent sudo activity

```bash
sudo journalctl _COMM=sudo --since today --no-pager
```

### Accounts with shell access

```bash
getent passwd | awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}'
```

### Accounts with UID 0

```bash
awk -F: '($3 == 0) {print}' /etc/passwd
```

### Password status for all users

```bash
sudo awk -F: '{print $1}' /etc/passwd | xargs -I{} sudo passwd -S {}
```

## 4.7 Lockdown and offboarding checklist

- Disable user account.
- Remove from sudo or wheel.
- Remove SSH keys.
- Kill user processes.
- Archive home directory if policy requires.
- Transfer ownership of service files.
- Remove cron jobs.
- Remove API or deploy credentials.
- Update access management system.

### Offboard example

```bash
USER=dev1
sudo usermod -L "$USER"
sudo gpasswd -d "$USER" sudo || true
sudo gpasswd -d "$USER" wheel || true
sudo pkill -u "$USER" || true
sudo tar -czf "/root/${USER}-home-$(date +%F).tgz" "/home/$USER"
sudo userdel -r "$USER"
```

## 4.8 Group management

```bash
sudo groupadd dockerops
sudo usermod -aG dockerops alice
getent group dockerops
```

## 4.9 User management one-liners

```bash
comm -23 <(getent passwd | cut -d: -f1 | sort) <(lastlog | awk 'NR>1 {print $1}' | sort)
```

```bash
sudo find /home -maxdepth 2 -name authorized_keys -exec grep -H 'ssh-ed25519' {} \;
```

---
