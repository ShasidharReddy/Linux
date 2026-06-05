# SSH

> **📌 Disclaimer**: Any third-party logos, screenshots, or diagrams referenced in this document are used for educational purposes only. All trademarks belong to their respective owners.


SSH is the standard secure remote administration protocol on Linux.

SSH deployments here use ed25519 (Edwards-curve Digital Signature Algorithm using Curve25519 — named after mathematician Harold Edwards; the curve was designed by Daniel J. Bernstein using a 255-bit prime. It's fast, secure, and produces compact 256-bit keys), RSA (Rivest–Shamir–Adleman — named after its creators Ron Rivest, Adi Shamir, and Leonard Adleman, 1977. Based on the difficulty of factoring large prime numbers), and ECDSA (Elliptic Curve Digital Signature Algorithm) host or user keys depending on policy and compatibility.

## 6.1 SSH components
### 📸 SSH Protocol Architecture
![SSH Layers](https://upload.wikimedia.org/wikipedia/commons/0/0f/Ssh_binary_packet_alt.svg)
> *Source: Wikimedia Commons — SSH binary packet protocol*

| Component | Purpose |
|---|---|
| `ssh` | Client command |
| `sshd` | Server daemon |
| `sshd_config` | Server configuration |
| `ssh_config` | Client configuration |
| `ssh-keygen` | Key generation |
| `ssh-agent` | Key cache in memory |
| `scp` | Secure copy |
| `sftp` | Secure file transfer |
| `rsync` over SSH | Efficient sync over encrypted transport |

## 6.2 Basic SSH usage
```bash
ssh user@server
ssh -p 2222 user@server
ssh -i ~/.ssh/id_ed25519 user@192.168.10.20
```

## 6.3 SSH host key verification
When connecting the first time, the client stores the server host key in:

```text
~/.ssh/known_hosts
```

This protects against man-in-the-middle attacks.

## 6.4 `sshd_config` location
Common path:

```text
/etc/ssh/sshd_config
```

Some distributions also use:

```text
/etc/ssh/sshd_config.d/
```

## 6.5 Important `sshd_config` settings
| Setting | Purpose |
|---|---|
| `Port` | SSH listening port |
| `PermitRootLogin` | Root login policy |
| `PasswordAuthentication` | Password auth enable or disable |
| `PubkeyAuthentication` | Public key authentication |
| `AllowUsers` | Restrict allowed users |
| `AllowGroups` | Restrict groups |
| `ListenAddress` | Bind address |
| `PermitEmptyPasswords` | Usually `no` |
| `ClientAliveInterval` | Idle keepalive |
| `MaxAuthTries` | Failed auth threshold |

## 6.6 Recommended hardened `sshd_config` example
```conf
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
AllowUsers admin deploy
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30
MaxAuthTries 3
```

Validate before restart:

```bash
sudo sshd -t
```

Restart service:

```bash
sudo systemctl restart sshd
```

or on Debian/Ubuntu:

```bash
sudo systemctl restart ssh
```

## 6.7 Generate SSH keys
### 6.7.1 Ed25519 key
```bash
ssh-keygen -t ed25519 -a 100 -C "admin@example.com"
```

### 6.7.2 RSA key
```bash
ssh-keygen -t rsa -b 4096 -C "admin@example.com"
```

Ed25519 is generally preferred for modern use.

## 6.8 Key files
Typical files:

- `~/.ssh/id_ed25519`
- `~/.ssh/id_ed25519.pub`
- `~/.ssh/authorized_keys`
- `~/.ssh/known_hosts`
- `~/.ssh/config`

## 6.9 Install public key on server
Easy method:

```bash
ssh-copy-id user@server
```

Manual method:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server 'umask 077 && mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

## 6.10 Permissions matter
Recommended permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

## 6.11 `ssh-agent`
Start agent and add key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

Useful when managing multiple servers without repeatedly typing passphrases.

## 6.12 SSH client config file
Path:

```text
~/.ssh/config
```

Example:

```conf
Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/id_ed25519

Host web-*
    User deploy
    IdentityFile ~/.ssh/id_ed25519
    ProxyJump bastion
```

Benefits:

- Shorter commands
- Consistent options
- Easier use of bastions and identity files

## 6.13 SSH port forwarding overview
Types:

- Local forwarding `-L`
- Remote forwarding `-R`
- Dynamic forwarding `-D`

## 6.14 Local port forwarding
Syntax:

```bash
ssh -L 8080:127.0.0.1:80 user@server
```

Meaning:

- Local port `8080`
- Forwards to remote `127.0.0.1:80`

Use cases:

- Access internal web UI
- Reach database consoles securely
- Temporary admin access without exposing ports

## 6.15 Remote port forwarding
Syntax:

```bash
ssh -R 8443:127.0.0.1:443 user@server
```

Use case:

- Publish a local service to the remote side

## 6.16 Dynamic forwarding with SOCKS proxy
```bash
ssh -D 1080 user@bastion
```

This creates a SOCKS proxy for tunneling application traffic.

## 6.17 `ProxyJump`
`ProxyJump` simplifies bastion-based access.

Example command:

```bash
ssh -J admin@bastion.example.com app@10.10.20.15
```

Equivalent config example:

```conf
Host app01
    HostName 10.10.20.15
    User app
    ProxyJump admin@bastion.example.com
```

## 6.18 `ProxyCommand`
Legacy but sometimes useful for custom transports.

Example:

```conf
Host internal
    HostName 10.10.20.15
    ProxyCommand ssh -W %h:%p bastion.example.com
```

## 6.19 Copy files with `scp`
```bash
scp file.txt user@server:/tmp/
scp user@server:/var/log/app.log .
scp -r ./site/ user@server:/var/www/html/
```

## 6.20 Transfer files with `sftp`
```bash
sftp user@server
```

Useful commands inside `sftp`:

- `put`
- `get`
- `ls`
- `cd`
- `lcd`
- `mkdir`

## 6.21 `rsync` over SSH
```bash
rsync -avz -e ssh ./site/ user@server:/var/www/html/
rsync -avz -e "ssh -i ~/.ssh/id_ed25519" ./backup/ user@server:/srv/backup/
```

Why `rsync` is excellent:

- Transfers only differences
- Preserves metadata
- Efficient over slow links

## 6.22 Restrict SSH access
Techniques:

- Disable root login
- Disable password auth
- Allow only specific users or groups
- Bind to management interfaces only
- Use firewalls to restrict source IPs
- Use MFA or SSO when available

## 6.23 Testing SSH server config safely
```bash
sudo sshd -t
sudo systemctl reload sshd
```

Prefer reload over restart when safe.

Keep an existing session open while validating new config.

## 6.24 SSH keepalive settings
Client side example:

```conf
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Server side example:

```conf
ClientAliveInterval 300
ClientAliveCountMax 2
```

## 6.25 Common SSH troubleshooting
| Symptom | Likely Cause |
|---|---|
| Connection refused | `sshd` not listening or firewall blocked |
| Permission denied (publickey) | Wrong key or permissions |
| Host key verification failed | Changed server key or MITM concern |
| Connection timed out | Routing, firewall, security group, or host down |
| No matching host key type | Old client/server crypto mismatch |

## 6.26 Useful SSH debug options
```bash
ssh -v user@server
ssh -vv user@server
ssh -vvv user@server
```

Check server logs too:

```bash
sudo journalctl -u sshd
sudo tail -f /var/log/auth.log
```

## 6.27 Authorized key restrictions
In `authorized_keys`, you can restrict behavior.

Example:

```text
from="192.168.10.0/24",command="/usr/local/bin/backup.sh",no-port-forwarding ssh-ed25519 AAAA...
```

This is useful for automation keys.

## 6.28 Agent forwarding
Enable only when necessary.

Example:

```bash
ssh -A user@server
```

Risk:

- Remote host may use your forwarded agent if compromised.

Prefer `ProxyJump` over agent forwarding when possible.

## 6.29 SSH multiplexing
Client config example:

```conf
Host *
    ControlMaster auto
    ControlPath ~/.ssh/control-%r@%h:%p
    ControlPersist 10m
```

This speeds up repeated SSH and SCP commands.

## 6.30 SSH best practices
- Use Ed25519 keys.
- Use strong passphrases.
- Disable password authentication on servers.
- Restrict access by source IP.
- Monitor auth logs.
- Rotate keys when staff changes.
- Use bastions for privileged environments.

## 6.31 Summary
SSH is both a remote shell and a secure transport platform. Mastering authentication, configuration, and tunneling makes Linux administration safer and more efficient.

---

## 6.1 Set up a jump host to access production servers
Objective:

- Keep application servers off the public Internet
- Force SSH access through a hardened bastion host
- Audit administrator entry points centrally

Typical design:

- Public bastion: `bastion.example.com`
- Private app servers: `app1.corp.example.internal`, `app2.corp.example.internal`
- Admin users connect with SSH keys and MFA on the bastion

Client SSH config:

```sshconfig
Host bastion
    HostName bastion.example.com
    User ops
    IdentityFile ~/.ssh/ops_ed25519

Host app1 app2
    User ops
    IdentityFile ~/.ssh/ops_ed25519
    ProxyJump bastion
```

Validation:

```bash
ssh app1
ssh app2
ssh -J bastion ops@app1.corp.example.internal
```

Hardening steps:

- Disable password login on the bastion
- Restrict source IPs if possible
- Enable session logging and MFA
- Patch the bastion aggressively
- Keep no application data on the bastion itself

---

## 6.7 Expose a local development service safely to a remote tester
Preferred method:

- Use SSH remote forwarding rather than opening inbound firewall holes broadly

Example:

```bash
ssh -R 127.0.0.1:9000:localhost:3000 dev@shared-lab.example.com
```

Remote validation:

```bash
curl http://127.0.0.1:9000/
```

If wider exposure is needed, coordinate with the remote admin before enabling non-loopback binds.

---

# SSH Command Reference and Checklists

## A.7 SSH and transfer commands

```bash
ssh user@host
scp file user@host:/path/
sftp user@host
rsync -avz -e ssh src/ user@host:/dst/
```

---

## D.3 SSH checklist

- [ ] Disable root login.
- [ ] Disable password authentication where possible.
- [ ] Use key-based auth.
- [ ] Protect private keys with passphrases.
- [ ] Restrict users and groups.
- [ ] Monitor failed login attempts.

---

## E.10 SSH operational patterns

### E.10.1 Bastion model

Pattern:

- Public bastion host
- Private app and DB servers
- Operators connect through bastion using `ProxyJump`

Benefits:

- Reduced public exposure
- Centralized auth logging
- Simpler firewall rules

### E.10.2 Automation accounts

For automation:

- Use dedicated service accounts
- Restrict authorized keys with forced commands or source IPs if practical
- Rotate keys regularly
