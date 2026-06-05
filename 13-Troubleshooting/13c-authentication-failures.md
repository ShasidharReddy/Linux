# Authentication Failures

← Back to [13-vm-ssh-access-issues.md](./13-vm-ssh-access-issues.md)

Key failures, permission problems, post-authentication blockers, and validation loops for SSH login issues.

---

### 13.2.6 🔍 Key authentication failures

**Severity:** 🟠 High

**Common symptoms**

- `Permission denied (publickey)` appears.
- The same user works from another laptop.
- SSH agent offers too many or the wrong keys.

**Useful checks**

```bash
ssh-add -l
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 -vvv user@target
sudo grep -i 'Failed publickey\|Accepted publickey' /var/log/auth.log
sudo grep -i 'Failed publickey\|Accepted publickey' /var/log/secure
```

**Practical fixes**

1. Force the intended key with `-i` and `IdentitiesOnly=yes`.
2. Verify the matching public key exists in the correct `authorized_keys` file.
3. Use a supported key type and signature algorithm such as Ed25519 or RSA SHA-2.

**How to validate the fix**

- Client debug shows the right key is offered.
- Server logs show the key accepted or a more specific remaining error.
- The user can log in without repeated retries or fallback methods.

**Prevention and operational notes**

- Keep SSH agent contents minimal during troubleshooting.
- Standardize key management for operators.
- Avoid silent drift in `AuthorizedKeysFile` paths.

### 13.2.7 🔍 Permission denied (publickey,password)

**Severity:** 🟠 High

**Common symptoms**

- Neither password nor key login works.
- The username may be wrong for the image or cloud platform.
- Account lockout, expiration, or identity-provider problems may be involved.

**Useful checks**

```bash
getent passwd user
sudo passwd -S user
sudo chage -l user
sudo faillock --user user 2>/dev/null
sudo tail -n 100 /var/log/auth.log
sudo tail -n 100 /var/log/secure
```

**Practical fixes**

1. Confirm the intended username for the OS image.
2. Unlock or restore the account according to policy.
3. If password auth is intentionally disabled, stop testing the wrong method and fix key auth instead.

**How to validate the fix**

- Server auth logs show either success or a clear remaining account-policy error.
- The correct user can authenticate using the approved method.
- Repeat attempts do not trigger lockout or fail2ban again.

**Prevention and operational notes**

- Document standard login users for each image type.
- Avoid repeated random password retries that pollute the evidence trail.
- Monitor identity-provider and PAM dependencies for SSH-dependent systems.

### 13.2.8 🔍 SSH config issues (`sshd_config`)

**Severity:** 🔴 Critical

**Common symptoms**

- The service fails to reload after config changes.
- Only a subset of users or source addresses are affected.
- Unexpected auth methods or ports are in effect.

**Useful checks**

```bash
sudo sshd -t
sudo sshd -T | sed -n '1,120p'
sudo grep -nE '^(Port|ListenAddress|PasswordAuthentication|PubkeyAuthentication|PermitRootLogin|AllowUsers|AllowGroups|DenyUsers|DenyGroups|AuthorizedKeysFile|Match)' /etc/ssh/sshd_config
```

**Practical fixes**

1. Validate syntax before reload or restart.
2. Inspect `Match` blocks carefully because later rules override general settings.
3. Restore known-good configuration from version control or configuration management when in doubt.

**How to validate the fix**

- `sshd -t` passes cleanly.
- `sshd -T` shows the expected effective settings.
- Users affected by the earlier rule can now log in as intended.

**Prevention and operational notes**

- Keep SSH configuration in version control.
- Use peer review for hardening changes.
- Test with a second session before closing the only working admin connection.


**Severity:** 🟢 Low

**Common symptoms**

- The failure stage is unclear without detailed client output.
- ProxyJump, ControlMaster, or agent forwarding hides the real cause.
- Multiple config files may be affecting the final behavior.

**Useful checks**

```bash
ssh -vvv user@target
ssh -F /dev/null -vvv user@target
ssh -vvv -J bastion user@target
ssh -G target | sed -n '1,120p'
```

**Practical fixes**

1. Look for the exact stage: DNS resolution, TCP connect, host key, auth method negotiation, key acceptance, or session setup.
2. Bypass local config with `-F /dev/null` to isolate local drift.
3. Capture sanitized excerpts when escalating to another team.

**How to validate the fix**

- The failing stage is identified clearly in the debug trace.
- Removing a local config override changes or resolves the issue as expected.
- The final debug log shows clean negotiation and session establishment.

**Prevention and operational notes**

- Keep SSH client config organized and commented.
- Avoid unnecessary complexity in jump-host chains.
- Teach responders the meaning of key debug phrases such as `Offering public key` and `Authentications that can continue`.

### 13.2.12 🔍 Account, shell, or post-authentication policy issue

**Severity:** 🟠 High

**Common symptoms**

- Auth succeeds and then the session closes immediately.
- Home directory ownership or shell startup files are broken.
- PAM, MFA, forced commands, or a full disk break session setup.

**Useful checks**

```bash
getent passwd user
sudo ls -ld /home/user
sudo ls -la /home/user
sudo df -h
sudo tail -n 100 /var/log/auth.log
sudo tail -n 100 /var/log/secure
```

**Practical fixes**

1. Ensure the user has a valid shell and a readable home directory.
2. Fix broken shell startup files or forced command definitions.
3. Free disk space if session creation or logging fails due to full storage.

**How to validate the fix**

- The user can run an interactive shell and a non-interactive command.
- Server logs show a complete login session rather than immediate teardown.
- The same user can reconnect repeatedly without reproducing the failure.

**Prevention and operational notes**

- Test login health after changing shells, PAM, or forced commands.
- Monitor root filesystem utilization.
- Keep shell customization minimal on service accounts.

### 13.2.13 🔍 Fail2ban, MaxStartups, or connection throttling

**Severity:** 🟡 Medium

**Common symptoms**

- SSH failures are intermittent under load or repeated retries.
- `kex_exchange_identification` appears during bursts.
- Only some source IPs fail because they were temporarily banned.

**Useful checks**

```bash
sudo fail2ban-client status 2>/dev/null
sudo fail2ban-client status sshd 2>/dev/null
sudo grep -i MaxStartups /etc/ssh/sshd_config
sudo journalctl -u sshd --since '1 hour ago' --no-pager | tail -100
```

**Practical fixes**

1. Unban legitimate source IPs only after confirming why they were banned.
2. Tune `MaxStartups` carefully if many legitimate concurrent connections are expected.
3. Reduce bad retries in automation and ensure the correct key is offered first.

**How to validate the fix**

- The affected source IP can connect consistently again.
- Automation no longer floods the daemon with repeated bad attempts.
- Logs show normal negotiation rather than rate limiting or bans.

**Prevention and operational notes**

- Track SSH retry rates from CI or automation systems.
- Use connection pooling or agent alternatives for high-volume automation.
- Document fail2ban and rate-limit behavior for responders.

### 13.2.14 🔐 SSH Permission and Ownership Rules That Commonly Break Logins

| Path | Recommended ownership | Typical mode | Why it matters |
|---|---|---|---|
| `~/.ssh` | user:user | `700` | Directory must not be writable by others |
| `~/.ssh/authorized_keys` | user:user | `600` | Loose permissions commonly cause key rejection |
| Client private key | user:user | `600` | Loose permissions can cause client refusal |
| Home directory | user:user or approved owner | `755` or stricter | Broken home perms can trigger policy failure |
| `/etc/ssh/sshd_config` | root:root | `600` or distro default | Unauthorized edits must be prevented |

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chown -R user:user ~/.ssh
```

### 13.2.15 🧪 Real-World SSH Failure Scenarios

#### SSH Scenario 1: After hardening, all SSH logins fail

**Observed behavior**

- A new hardening change was deployed.
- Users now fail before or during authentication.

**Likely root cause**

- A bad `Match` block or restrictive allow-list excluded legitimate admins.

**Resolution steps**

1. Run `sshd -t` and inspect `sshd -T`.
2. Restore or fix the hardening change.
3. Retest with a second admin session before ending the maintenance window.

**Key lesson**

- Always keep one verified fallback session when editing SSH controls.

#### SSH Scenario 2: Only one engineer gets `Permission denied (publickey)`

**Observed behavior**

- The same user works from another machine.
- The failing client offers multiple keys.

**Likely root cause**

- The SSH agent is offering the wrong key and the intended identity is never used.

**Resolution steps**

1. Use `IdentitiesOnly=yes` and specify the right key.
2. Clean stale agent identities.
3. Confirm acceptance in server auth logs.

**Key lesson**

- Client-side key confusion is extremely common and easy to prove.

#### SSH Scenario 3: Host key mismatch after autoscaling replacement

**Observed behavior**

- A node was rebuilt automatically.
- Users now get host key warnings.

**Likely root cause**

- The hostname points to a new instance with a different legitimate host key.

**Resolution steps**

1. Validate the new fingerprint from a trusted source.
2. Remove only the stale known_hosts entry.
3. Retest to the verified endpoint.

**Key lesson**

- Treat host identity as a security event until proven otherwise.

#### SSH Scenario 4: Connection refused but ping works

**Observed behavior**

- ICMP succeeds.
- TCP/22 fails immediately with refused.

**Likely root cause**

- `sshd` is down or not listening on the expected address or port.

**Resolution steps**

1. Inspect service state and `ss -ltnp`.
2. Repair config and restart the daemon.
3. Confirm port 22 listens again.

**Key lesson**

- Refused generally means you reached the host and it answered.

#### SSH Scenario 5: Auth succeeds then the shell exits immediately

**Observed behavior**

- The key is accepted.
- The user is disconnected right after login.

**Likely root cause**

- The shell environment, home directory, PAM stack, or disk state is broken.

**Resolution steps**

1. Check shell startup files.
2. Inspect auth logs and disk utilization.
3. Test both interactive and non-interactive commands.

**Key lesson**

- Not every post-auth disconnect is an SSH daemon problem.

#### SSH Scenario 6: CI jobs randomly fail to SSH

**Observed behavior**

- Failures occur in bursts under automation load.
- Manual logins still work sometimes.

**Likely root cause**

- The target is hitting `MaxStartups`, fail2ban, or another throttling control.

**Resolution steps**

1. Reduce concurrency or retry storms.
2. Tune limits carefully if justified.
3. Whitelist approved automation ranges only when policy allows.

**Key lesson**

- Automation can DoS your own admin plane if left untuned.

#### SSH Scenario 7: Password login never works on a cloud image

**Observed behavior**

- Users keep trying passwords.
- The image was built for key-only access.

**Likely root cause**

- The wrong auth method is being tested repeatedly.

**Resolution steps**

1. Confirm image defaults.
2. Install the correct key or use the provider-approved access path.
3. Stop wasting cycles on disabled password auth.

**Key lesson**

- Use the auth method the platform was designed for.

#### SSH Scenario 8: ProxyJump chain fails silently

**Observed behavior**

- Direct bastion login works.
- Final target login through the chain fails unexpectedly.

**Likely root cause**

- The local SSH config or one hop in the chain is stale or mis-specified.

**Resolution steps**

1. Inspect `ssh -G` output.
2. Test each hop independently.
3. Fix jump-host user, key, or host entries.

**Key lesson**

- Multi-hop SSH needs step-by-step validation.

---

### 13.5.2 SSH diagnostics commands

- **Verbose SSH**
```bash
ssh -vvv user@target
```

- **Bypass local config**
```bash
ssh -F /dev/null -vvv user@target
```

- **Force specific key**
```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 -vvv user@target
```

- **Show effective client config**
```bash
ssh -G target | sed -n "1,120p"
```

- **List agent keys**
```bash
ssh-add -l
```

- **Check daemon status**
```bash
sudo systemctl status sshd --no-pager
```

- **Show SSH logs**
```bash
sudo journalctl -u sshd --since '1 hour ago' --no-pager
```

- **Validate daemon config**
```bash
sudo sshd -t
```

- **See listen sockets**
```bash
sudo ss -ltnp | grep ':22'
```

- **Inspect failed login counters**
```bash
sudo faillock --user user 2>/dev/null
```

### 13.6.2 SSH failure evidence

- Exact SSH error text copied verbatim.
- Sanitized `ssh -vvv` excerpt showing the failing stage.
- Relevant server logs from `journalctl -u sshd`, `/var/log/auth.log`, or `/var/log/secure`.
- Output of `sshd -t`, `ss -ltnp`, and critical SSH configuration directives.
- Any recent changes to keys, users, MFA, PAM, image type, or hardening policy.

### SSH validation loop

- Retest with `ssh -vvv` once to confirm the failure stage is gone.
- Confirm `sshd` is active and listening on the intended address and port.
- Verify the intended user can log in with the approved auth method.
- If a host key changed, confirm clients updated only after identity verification.
- Document any key, account, or config changes in the change log.

### FAQ 5: Why do I care about `sshd -t`?

It validates SSH daemon configuration before a restart so you do not make the outage worse.

### FAQ 6: Why not just `chmod 644` the log file?

Because it is often too broad, may expose sensitive data, and may break again after rotation.

### FAQ 7: When should I use ACLs instead of groups?

Use ACLs when one team needs narrow access to one path without broader group exposure.

### FAQ 8: Why is centralized logging better for app teams?

It gives them searchable access without root shell and scales better for many teams.

### FAQ 9: What if the VM is healthy in the console but unreachable?

Check effective policy, routes, and whether the guest OS or firewall is actually listening and responding.

### FAQ 10: What if only one engineer fails to connect?
