# SSH Hardening

SSH is one of the most critical Linux administration services.

It is also one of the most attacked.

Hardening SSH is mandatory for nearly every production Linux system.

### 7.1 Basic SSH Hardening Goals

- reduce exposed attack surface
- remove password guessing opportunities
- restrict who can log in
- strengthen logging and monitoring
- add layered controls such as MFA and intrusion prevention

### 7.2 Review Current Configuration

Main server config is typically:

- `/etc/ssh/sshd_config`
- additional drop-ins under `/etc/ssh/sshd_config.d/`

Inspect effectively:

```bash
sudo sshd -T
sudo grep -RIn 'PermitRootLogin\|PasswordAuthentication\|PubkeyAuthentication\|AllowUsers\|AllowGroups\|Port\|AuthenticationMethods' /etc/ssh/sshd_config /etc/ssh/sshd_config.d 2>/dev/null
```

### 7.3 Disable Root Login

Direct root SSH login is high risk.

Recommended setting:

```text
PermitRootLogin no
```

Alternative break-glass option:

```text
PermitRootLogin prohibit-password
```

Why disable it:

- forces named-account accountability
- improves audit trails
- reduces direct high-value brute-force target access

### 7.4 Use Key-Only Authentication

Disable password authentication where feasible.

Recommended server settings:

```text
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
```

Key benefits:

- better resistance to guessing and credential stuffing
- support for hardware-backed keys
- improved automation safety when managed correctly

Client key generation examples:

```bash
ssh-keygen -t ed25519 -a 100 -C 'admin@example'
ssh-keygen -t ecdsa-sk -C 'fido2-key'
```

### 7.5 Restrict Who Can Log In

Use allow lists instead of broad access.

Examples:

```text
AllowUsers alice bob
AllowGroups sshadmins
DenyUsers test oldadmin
```

You can also use `Match` blocks for source-based or role-based control.

Example:

```text
Match Address 10.0.0.0/24
    PasswordAuthentication no
```

### 7.6 Change the Port

Changing SSH from port 22 to another port can reduce noise from automated scanners.

Example:

```text
Port 2222
```

Important note:

This is not a substitute for strong authentication.

It reduces opportunistic noise, not targeted attacks.

Remember to:

- update firewall rules
- update SELinux port labels if enforcing
- update documentation and automation

### 7.7 Additional sshd Settings

Common hardening options:

```text
Protocol 2
MaxAuthTries 3
LoginGraceTime 30
PermitEmptyPasswords no
X11Forwarding no
AllowTcpForwarding no
PermitTunnel no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxSessions 4
```

Use case notes:

- Disable forwarding if the system does not need it.
- Lower `MaxAuthTries` to reduce brute-force attempts.
- Use keepalive settings to close abandoned sessions.

### 7.8 File Permissions for SSH

Client and server keys require strict permissions.

Examples:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
sudo chmod 600 /etc/ssh/ssh_host_*_key
```

Bad permissions can cause both security risk and authentication failure.

### 7.9 fail2ban

`fail2ban` monitors logs and temporarily bans abusive sources.

It is useful for SSH brute-force reduction.

High-level workflow:

- read auth logs
- detect repeated failures
- add temporary firewall bans

Example conceptual jail settings:

```ini
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
findtime = 10m
bantime = 1h
```

Operational guidance:

- tune ban times to environment risk
- ensure logs are accurate and timely
- avoid depending on fail2ban as the only control

### 7.10 Two-Factor Authentication with Google Authenticator

A common PAM-based approach uses `pam_google_authenticator.so`.

High-level steps:

1. Install the PAM module package.
2. Enroll each user with `google-authenticator`.
3. Update PAM for SSH.
4. Update `sshd_config` to require the desired methods.
5. Test with a second session before rollout.

Conceptual config snippets vary by distribution, but the design usually involves:

- enabling keyboard-interactive or KbdInteractive support where required
- adding `auth required pam_google_authenticator.so`
- defining `AuthenticationMethods publickey,keyboard-interactive`

Security notes:

- secure recovery codes
- enforce time sync
- document break-glass procedures
- test automation accounts separately

### 7.11 SSH Certificate Authentication

At scale, SSH certificates can be safer and easier to manage than large static authorized_keys sprawl.

Benefits:

- short-lived access
- centralized trust through a CA
- easier revocation strategy in many environments

### 7.12 Bastion Host Model

Instead of exposing SSH everywhere:

- expose a hardened bastion
- require MFA
- log heavily
- restrict downstream access through policy
- separate production from non-production paths

### 7.13 Agent Forwarding and Tunneling Risks

Be careful with:

- agent forwarding
- local port forwarding
- remote port forwarding
- dynamic SOCKS forwarding

These features can be extremely useful but may create lateral movement opportunities.

Disable them where not required.

### 7.14 Example Hardened sshd_config Snippet

```text
Port 2222
Protocol 2
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
AllowGroups sshadmins
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowTcpForwarding no
PermitTunnel no
ClientAliveInterval 300
ClientAliveCountMax 2
```

### 7.15 Testing SSH Changes Safely

Use this rollout pattern:

1. Keep one existing session open.
2. Validate syntax.
3. Reload the service.
4. Test from a fresh client.
5. Confirm MFA and key-only behavior.
6. Confirm root login is blocked.
7. Confirm expected users can still log in.

Helpful commands:

```bash
sudo sshd -t
sudo systemctl reload sshd
ssh -p 2222 alice@host
```

### 7.16 Monitoring SSH

Watch for:

- repeated failures
- successful logins from new geographies or networks
- off-hours admin access
- new keys added to `authorized_keys`
- service restarts or config drift

Sources:

- `journalctl -u sshd`
- `/var/log/auth.log`
- centralized SIEM dashboards

### 7.17 SSH Hardening Checklist

- Disable root login.
- Prefer key-only auth.
- Add MFA for admins.
- Limit users and groups.
- Restrict source networks.
- Consider port change to reduce noise.
- Tune auth attempt and timeout settings.
- Monitor and alert on login anomalies.
- Protect SSH keys and configs with correct permissions.

### 7.18 Summary

SSH hardening combines strong authentication, narrow authorization, network restriction, careful configuration, and active monitoring.

Treat SSH as a privileged control plane, not just a convenience service.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.7 SSH Checklist

- Disable root login.
- Disable password auth where possible.
- Enforce MFA for admins.
- Restrict users and groups.
- Restrict source ranges.
- Review `MaxAuthTries` and timeouts.
- Review host key permissions.
- Review user `authorized_keys` drift.
- Confirm fail2ban or equivalent is active if required.
- Validate `sshd -T` output after changes.

### B.7 SSH Commands

```bash
sshd -T
sshd -t
systemctl reload sshd
ssh-keygen -t ed25519 -a 100
ssh -v user@host
journalctl -u sshd
```

### C.7 SSH

81. Why is SSH such a common attack target?
82. Why are hardware-backed keys valuable?
83. Why is changing the SSH port only a noise-reduction control?
84. Why should `AllowUsers` or `AllowGroups` be considered?
85. Why is `PasswordAuthentication no` often desirable?
86. What does fail2ban add to SSH defense?
87. Why is it important to keep one working SSH session open during changes?
88. Why should agent forwarding be restricted?
89. Why should SSH key file permissions be strict?
90. Why should admin access preferably come from VPN or bastion paths?
