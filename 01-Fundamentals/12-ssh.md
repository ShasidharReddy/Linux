# 14. SSH — Secure Shell
## 14.1 What is SSH?
SSH stands for Secure Shell. It is the standard encrypted protocol for remote administration, secure command execution, port forwarding, and secure file transfer on Linux and Unix-like systems.
Key facts:
- SSH replaced older plaintext tools such as `telnet`, `rsh`, and `rlogin`.
- Default port: `22`.
- Modern systems use SSH protocol version 2 only.
- SSH is used by admins, automation, CI/CD, Git, bastion hosts, and secure tunneling.
Core components:
| Component | Purpose |
|---|---|
| `ssh` | Client used to connect to a server |
| `sshd` | SSH server daemon |
| `~/.ssh/known_hosts` | Trusted host fingerprints |
| `~/.ssh/authorized_keys` | Allowed public keys for a user |
| `~/.ssh/config` | Client-side host shortcuts and defaults |
| `/etc/ssh/sshd_config` | Server-side SSH configuration |
| `ssh-agent` | Holds unlocked private keys in memory |
| `ssh-add` | Loads keys into the agent |
## 14.2 How SSH Login Works
1. The client connects to the server on port `22` or a custom port.
2. The server presents its host key.
3. The client checks whether that host key is already trusted.
4. A secure session is negotiated.
5. The user authenticates with a password, public key, certificate, or another allowed method.
6. The server starts a shell session or runs the requested command.
## 14.3 How to Login to a Server
Basic login:
```bash
ssh username@server-ip
ssh admin@192.168.1.50
ssh admin@web01.example.com
```
Login using a specific port:
```bash
ssh -p 2222 deploy@web01.example.com
```
Login using a specific identity file:
```bash
ssh -i ~/.ssh/prod-admin-ed25519 admin@10.10.20.15
```
Run a remote command without opening an interactive shell:
```bash
ssh admin@server 'hostname && uptime && df -h /'
```
Example output:
```text
web01
 11:42:15 up 18 days,  4:10,  2 users,  load average: 0.09, 0.07, 0.05
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        80G   28G   49G  37% /
```
## 14.4 First-Time Connection and `known_hosts`
The first time you connect, SSH asks whether you trust the host key. That fingerprint must be verified through a trusted channel before you accept it in production.
Example prompt:
```text
The authenticity of host '192.168.1.50 (192.168.1.50)' can't be established.
ED25519 key fingerprint is SHA256:7xQhK9m0examplefingerprint1234567890abcd.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.50' (ED25519) to the list of known hosts.
```
Useful host-key commands:
```bash
ssh-keyscan -t ed25519 web01.example.com | ssh-keygen -lf -
ssh-keygen -R web01.example.com
ssh-keyscan -H web01.example.com >> ~/.ssh/known_hosts
```
## 14.5 SSH Key Setup (Step-by-Step)
### Step 1: Generate a key pair
Recommended:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
Other common choices:
- `ed25519` — recommended for most users.
- `rsa` with `-b 4096` — good for compatibility with older environments.
- `ecdsa` — supported, but less commonly preferred today.
Example output:
```text
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Your identification has been saved in /home/user/.ssh/id_ed25519
Your public key has been saved in /home/user/.ssh/id_ed25519.pub
```
Use a passphrase on important admin keys. A passphrase protects the private key if the file is copied or stolen.
Generate a strong RSA key when compatibility requires it:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
### Step 2: Copy the public key to the server
Preferred method:
```bash
ssh-copy-id user@server
ssh-copy-id admin@192.168.1.50
```
Example output:
```text
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'admin@192.168.1.50'"
```
Manual method if `ssh-copy-id` is unavailable:
```bash
cat ~/.ssh/id_ed25519.pub
```
Append that one-line public key into `~/.ssh/authorized_keys` on the remote server:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
printf '%s\n' 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExamplePublicKeyHere your_email@example.com' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
### Step 3: Set correct permissions
SSH is strict about key permissions.
| Path | Typical mode |
|---|---|
| `~/.ssh` | `700` |
| `~/.ssh/authorized_keys` | `600` |
| private key such as `~/.ssh/id_ed25519` | `600` |
| public key such as `~/.ssh/id_ed25519.pub` | `644` |
Commands:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```
### Step 4: Test the login
```bash
ssh user@server
ssh -vvv user@server
```
Successful verbose output usually includes lines like:
```text
debug1: Offering public key: /home/user/.ssh/id_ed25519 ED25519
debug1: Server accepts key: /home/user/.ssh/id_ed25519 ED25519
debug1: Authentication succeeded (publickey).
```
## 14.6 SSH Config File (`~/.ssh/config`)
A client config file eliminates repeated usernames, ports, keys, and tunnel definitions.
Minimal example:
```sshconfig
Host web-prod
    HostName 10.10.20.15
    User admin
    Port 22
    IdentityFile ~/.ssh/prod-admin-ed25519
```
Then connect with:
```bash
ssh web-prod
```
Complete example with multiple hosts, a bastion, and defaults:
```sshconfig
Host web-prod
    HostName 10.10.20.15
    User admin
    Port 22
    IdentityFile ~/.ssh/prod-admin-ed25519
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
Host app-prod
    HostName 10.10.20.25
    User deploy
    Port 2222
    IdentityFile ~/.ssh/prod-deploy-ed25519
    IdentitiesOnly yes
    ForwardAgent no
    Compression yes
Host db-prod
    HostName 10.10.30.10
    User db-admin
    IdentityFile ~/.ssh/prod-db-ed25519
    LocalForward 3306 127.0.0.1:3306
    ServerAliveInterval 30
Host bastion
    HostName 203.0.113.20
    User ops
    IdentityFile ~/.ssh/bastion-ed25519
Host internal-*.example.com
    User admin
    ProxyJump bastion
    IdentityFile ~/.ssh/internal-admin-ed25519
    StrictHostKeyChecking yes
```
Important options:
- `IdentityFile` — private key for that host.
- `IdentitiesOnly yes` — stops the client from offering too many keys.
- `ProxyJump` — route through a bastion host.
- `LocalForward` — create a default local tunnel.
- `ServerAliveInterval` and `ServerAliveCountMax` — keep sessions healthy across flaky links.
Connect through a bastion without a config file:
```bash
ssh -J ops@203.0.113.20 admin@10.10.20.15
```
## 14.7 SSH Server Configuration (`/etc/ssh/sshd_config`)
Production servers should use explicit settings rather than relying only on defaults.
Example hardened configuration:
```sshconfig
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
AllowUsers admin deploy
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 10
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding yes
PermitEmptyPasswords no
UsePAM yes
```
Important settings:
- `PermitRootLogin no` — block direct SSH login as `root`.
- `PasswordAuthentication no` — require keys once rollout is complete.
- `AllowUsers` or `AllowGroups` — restrict who may log in.
- `MaxAuthTries` and `LoginGraceTime` — reduce brute-force risk.
- `ClientAliveInterval` — clean up dead sessions.
Validate the configuration before restart:
```bash
sudo sshd -t
```
Good syntax produces no output. A typo might look like:
```text
/etc/ssh/sshd_config line 17: Bad configuration option: PasswordAuthentcation
/etc/ssh/sshd_config: terminating, 1 bad configuration options
```
Restart or reload SSH after changes:
```bash
sudo systemctl restart sshd
sudo systemctl restart ssh
sudo systemctl status sshd --no-pager
```
On Debian or Ubuntu, the service name is often `ssh`. Always keep one existing session open while testing changes so you do not lock yourself out.
## 14.8 SSH Agent
`ssh-agent` stores unlocked keys in memory so you can enter the passphrase once and reuse the key.
Commands:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-add -D
```
Typical output from the first command:
```text
Agent pid 2487
```
## 14.9 Agent Forwarding
Agent forwarding lets a remote host use your local agent for onward connections.
```bash
ssh -A bastion
```
This is convenient, but use it carefully. A compromised intermediate host may ask your forwarded agent to authenticate while your session is active. Prefer `ProxyJump`, dedicated per-environment keys, or short-lived certificates when possible.
## 14.10 SCP, SFTP, and `rsync` over SSH
Copy a file to a remote host:
```bash
scp backup.tar.gz user@server:/path/
scp ./nginx.conf admin@web01.example.com:/etc/nginx/nginx.conf
```
Copy a file back:
```bash
scp user@server:/var/log/syslog ./syslog.copy
```
Copy a directory recursively:
```bash
scp -r ./site-content admin@web01.example.com:/var/www/html/
```
Interactive secure file transfer with SFTP:
```bash
sftp user@server
```
Common `sftp` commands:
```text
pwd
lpwd
ls
cd /var/www/html
put index.html
get backup.sql
mkdir uploads
bye
```
For repeatable deployments or backups, `rsync` is usually better than `scp` because it transfers only changes:
```bash
rsync -avz -e ssh source/ user@server:/dest/
rsync -avz --delete --dry-run -e ssh ./release/ deploy@web01.example.com:/srv/www/release/
rsync -avz -e "ssh -i ~/.ssh/prod-deploy-ed25519" ./release/ deploy@web01.example.com:/srv/www/release/
```
## 14.11 Port Forwarding and Tunneling
Local port forwarding exposes a remote service on your workstation:
```bash
ssh -L 8080:127.0.0.1:80 admin@web01.example.com
```
After the tunnel is active, `http://127.0.0.1:8080` on your laptop reaches port `80` on the remote server.
Remote forwarding exposes a local service on the remote side:
```bash
ssh -R 9000:127.0.0.1:3000 user@remote-host
```
Dynamic forwarding creates a SOCKS proxy:
```bash
ssh -D 1080 user@bastion
```
## 14.12 How to Login to a Database Server
A common production pattern is to SSH to the host or bastion first, then run the database client locally on that server, or create a local SSH tunnel from your workstation.
MySQL or MariaDB directly on the DB server:
```bash
ssh db-admin@db-server
mysql -u root -p
```
Example output:
```text
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 72
Server version: 10.11.6-MariaDB Ubuntu 22.04
```
PostgreSQL directly on the DB server:
```bash
ssh db-admin@db-server
psql -U postgres
```
Example output:
```text
psql (15.6)
Type "help" for help.
postgres=#
```
MySQL tunnel from your local machine:
```bash
ssh -L 3306:localhost:3306 db-admin@db-server
mysql -h 127.0.0.1 -P 3306 -u root -p
```
PostgreSQL tunnel:
```bash
ssh -L 5432:localhost:5432 db-admin@db-server
psql -h localhost -U postgres
```
MongoDB tunnel:
```bash
ssh -L 27017:localhost:27017 admin@mongo-server
mongosh --host localhost
```
Redis tunnel:
```bash
ssh -L 6379:localhost:6379 admin@redis-server
redis-cli -h localhost
```
## 14.13 SSH Security Best Practices
- Disable direct root login.
- Use Ed25519 or strong RSA keys.
- Protect private keys with passphrases.
- Disable password authentication once keys are deployed.
- Restrict login with `AllowUsers`, `AllowGroups`, firewalls, and cloud security groups.
- Monitor `/var/log/auth.log`, `/var/log/secure`, and `journalctl -u sshd`.
- Remove stale keys from `authorized_keys`.
- Rotate keys when people or roles change.
- Use bastion hosts for private network access.
- Consider MFA or SSH certificates in larger environments.
## 14.14 Common SSH Troubleshooting
| Problem | Likely cause | Fix |
|---|---|---|
| `Permission denied (publickey)` | Wrong key, bad permissions, missing public key | Check `authorized_keys`, key path, and `chmod` values |
| `Connection refused` | SSH service stopped or firewall blocked | Verify `sshd` is running and port is open |
| `Host key verification failed` | Host key changed or stale `known_hosts` entry | Verify the change and update the old entry |
| Connection hangs | Routing issue, wrong port, firewall, security group | Test with `ping`, `nc`, `ss`, and firewall rules |
| Too many authentication failures | Client offered too many keys | Use `IdentitiesOnly yes` |
Useful commands:
```bash
ssh -vvv user@server
sudo journalctl -u sshd --no-pager -n 50
sudo ss -tulpn | grep :22
sudo firewall-cmd --list-services
sudo ufw status verbose
```
## 14.15 Mermaid: SSH Connection Flow
```mermaid
graph LR
    CLIENT["💻 Your Machine"] -->|"ssh -i key.pem user@server"| SSHD["🖥️ SSH Server<br/>Port 22"]
    SSHD -->|"Key Exchange<br/>Authentication"| SHELL["🐚 Shell Session"]
```
