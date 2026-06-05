# 7. Users and Groups

Linux account administration relies on `sudo` (Super User Do), `su` (Switch User), UID (User ID), GID (Group ID), PAM (Pluggable Authentication Modules), and LDAP (Lightweight Directory Access Protocol) concepts that appear throughout this chapter.

## 7.1 Why User Management Matters

Linux separates identities to control access and accountability.

Users and groups are foundational for:

- security
- auditing
- service isolation
- multiuser collaboration

## 7.2 Important Account Files

### `/etc/passwd`

Stores basic account information.

Fields include:

- username
- password placeholder
- UID
- GID
- comment field
- home directory
- login shell

Example:

```bash
cat /etc/passwd
```

### `/etc/shadow`

Stores password hashes and aging information.

Access is restricted.

Example:

```bash
sudo cat /etc/shadow
```

### `/etc/group`

Stores group definitions and memberships.

Example:

```bash
cat /etc/group
```

## 7.3 UIDs and GIDs

- UID = user ID
- GID = group ID

Special cases:

- UID 0 is `root`
- system/service accounts often use lower numeric ranges
- normal interactive users usually have higher numeric ranges

## 7.4 `useradd`

### Purpose

Create a user account.

### Examples

```bash
sudo useradd alice
sudo useradd -m -s /bin/bash bob
sudo useradd -m -c "App User" -s /usr/sbin/nologin appuser
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-m` | Create home directory |
| `-s` | Set login shell |
| `-c` | Comment or full name |
| `-u` | Set UID |
| `-g` | Primary group |
| `-G` | Supplementary groups |

## 7.5 `usermod`

### Purpose

Modify an existing user account.

### Examples

```bash
sudo usermod -aG sudo alice
sudo usermod -s /bin/zsh bob
sudo usermod -d /home/newhome -m bob
```

### Notes

Use `-aG` when adding supplementary groups.

Without `-a`, group membership may be replaced.

## 7.6 `userdel`

### Purpose

Delete a user account.

### Examples

```bash
sudo userdel tempuser
sudo userdel -r olduser
```

### Notes

`-r` removes the home directory and mail spool.

Use carefully.

## 7.7 `groupadd`

### Purpose

Create a group.

### Examples

```bash
sudo groupadd developers
sudo groupadd -g 1500 analytics
```

## 7.8 `passwd`

### Purpose

Set or change a password.

### Examples

```bash
passwd
sudo passwd alice
sudo passwd -l alice
sudo passwd -u alice
```

### Notes

- `-l` locks an account password
- `-u` unlocks it

## 7.9 Checking Identity

Useful commands:

```bash
id
whoami
groups
getent passwd alice
getent group developers
```

## 7.10 `su`

### Purpose

Switch user identity.

### Examples

```bash
su -
su - alice
```

### Notes

`su -` starts a full login shell for the target user.

## 7.11 `sudo`

### Purpose

Run a command with elevated privileges.

### Examples

```bash
sudo ls /root
sudo systemctl restart nginx
sudo -u postgres psql
```

### Benefits

- reduces direct root logins
- creates audit trails
- allows controlled delegation

## 7.12 `visudo`

### Purpose

Safely edit the sudoers policy file.

### Example

```bash
sudo visudo
```

Why use it:

- syntax checking
- prevents simultaneous unsafe edits

## 7.13 Sudoers Concepts

Common sudo policy locations:

- `/etc/sudoers`
- `/etc/sudoers.d/`

Example policy line:

```text
alice ALL=(ALL) ALL
```

Meaning:

- user `alice`
- on all hosts
- may run commands as all users
- with password prompt unless configured otherwise

## 7.14 Common Administrative Tasks

### Create a new admin user

```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
sudo usermod -aG sudo alice
```

On some distributions the admin group may be `wheel` instead of `sudo`.

### Create a service account

```bash
sudo useradd -r -s /usr/sbin/nologin appsvc
```

### Check sudo access

```bash
sudo -l
```

## 7.15 Security Best Practices

- avoid direct root logins when possible
- give sudo only to trusted admins
- use service accounts for services
- disable interactive shells for non-login accounts
- review group membership periodically
- protect `/etc/shadow`

## 7.16 Troubleshooting User Issues

Common questions:

- does the user exist?
- is the shell valid?
- does the home directory exist?
- is the account locked?
- is the user in the correct group?
- does sudo policy allow the command?

Useful checks:

```bash
id alice
getent passwd alice
sudo passwd -S alice
sudo -l -U alice
```

> Warning:
> Never edit `/etc/sudoers` directly with a normal editor unless you are absolutely sure what you are doing.
> A syntax error can block administrative access.

---

