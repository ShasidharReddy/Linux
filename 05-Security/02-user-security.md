# User Security

User and authentication controls are among the most important Linux security boundaries.

Most breaches eventually involve identity abuse.

A well-hardened authentication stack reduces unauthorized access, slows lateral movement, and improves accountability.

Password storage examples in this chapter mention SHA (Secure Hash Algorithm)-512 so the hash family name is explicit when reviewing `/etc/shadow` formats.

## 2.1 Account Inventory

Start by identifying who can log in and who has privilege.

```bash
getent passwd
awk -F: '$3 >= 1000 {print $1, $7}' /etc/passwd
getent shadow | cut -d: -f1,2
sudo getent group sudo
sudo getent group wheel
lastlog
faillog -a
```

Review for:

- Dormant accounts.
- Shared accounts.
- Service accounts with interactive shells.
- Accounts without password aging.
- Unexpected members of sudo or wheel.

### 2.2 Password Policy Basics

Password policy must balance resistance to guessing with usability and MFA adoption.

Recommended baseline:

| Control | Recommendation |
| --- | --- |
| Minimum length | 14+ characters for humans |
| Complexity | Prefer passphrases over forced symbols alone |
| Reuse | Remember at least 5 to 10 previous passwords |
| Rotation | Rotate on suspicion, role change, or policy requirement |
| Lockout | Temporary lock after repeated failures |
| MFA | Require for remote access and privileged access |

Modern guidance favors length and uniqueness over outdated complexity-only rules.

Bad examples:

- Summer2024!
- CompanyName1!
- Admin@123

Better examples:

- mango-lantern-coast-window
- BrickRiverPaperTrainNorth
- violet-orbit-anchor-museum

### 2.3 PAM Overview

PAM stands for Pluggable Authentication Modules.

PAM centralizes authentication, account checks, password policy, and session rules.

Common PAM module types:

| Type | Purpose |
| --- | --- |
| auth | Verify identity |
| account | Check whether access is allowed |
| password | Handle password changes and quality checks |
| session | Perform tasks at login and logout |

Common PAM files:

| Distribution Style | Common Files |
| --- | --- |
| Debian-like | /etc/pam.d/common-auth, common-account, common-password, common-session |
| RHEL-like | /etc/pam.d/system-auth, password-auth, sshd, su, sudo |

Common PAM modules:

- pam_unix.so
- pam_pwquality.so
- pam_faillock.so
- pam_tally2.so on older systems
- pam_limits.so
- pam_access.so
- pam_google_authenticator.so
- pam_lastlog.so
- pam_mkhomedir.so

PAM control flags matter:

| Control Flag | Meaning |
| --- | --- |
| required | Must succeed, but processing continues |
| requisite | Must succeed or fail immediately |
| sufficient | Success can satisfy the stack if no required failures exist |
| optional | Usually non-critical |

### 2.4 Password Quality with PAM

Example RHEL-like configuration using pam_pwquality:

```bash
sudo grep -n 'pwquality\|faillock' /etc/pam.d/system-auth /etc/pam.d/password-auth
sudo grep -n '^minlen\|^minclass\|^dcredit\|^ucredit\|^lcredit\|^ocredit' /etc/security/pwquality.conf
```

Example settings in `/etc/security/pwquality.conf`:

```ini
minlen = 14
difok = 4
minclass = 4
maxrepeat = 3
maxclassrepeat = 4
gecoscheck = 1
enforcing = 1
dictcheck = 1
```

Notes:

- `minlen` controls minimum length.
- `difok` requires difference from the previous password.
- `maxrepeat` reduces repeated-character abuse.
- `dictcheck` helps block dictionary-based passwords.

### 2.5 /etc/login.defs

`/etc/login.defs` influences account creation and password aging behavior.

Key settings often reviewed:

| Setting | Purpose | Example |
| --- | --- | --- |
| PASS_MAX_DAYS | Maximum password age | 90 |
| PASS_MIN_DAYS | Minimum days before change | 1 |
| PASS_WARN_AGE | Warning before expiry | 14 |
| UMASK | Default permission mask | 027 |
| UID_MIN | Start of regular user IDs | 1000 |
| ENCRYPT_METHOD | Hash method | yescrypt or SHA512 |

Example inspection:

```bash
grep -E '^(PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE|UMASK|ENCRYPT_METHOD)' /etc/login.defs
```

Recommended direction:

- Use a restrictive `UMASK` such as `027` or `077` where appropriate.
- Ensure password aging aligns with policy.
- Use modern password hashing supported by the distribution.

### 2.6 Password Aging and Expiration

Use `chage` to inspect and enforce password aging.

```bash
sudo chage -l alice
sudo chage -M 90 -m 1 -W 14 alice
sudo chage -E 2025-12-31 contractor1
```

Useful fields:

- Last password change.
- Password expires.
- Password inactive.
- Account expires.
- Minimum days between changes.
- Maximum days between changes.
- Warning period.

When to use expiration:

- Temporary contractors.
- Emergency vendor accounts.
- Project-based access.

### 2.7 Account Locking

Account lockout reduces brute-force success.

Modern RHEL-like systems commonly use `pam_faillock`.

Example concept:

```ini
auth        required      pam_faillock.so preauth silent deny=5 unlock_time=900
auth        [default=die] pam_faillock.so authfail deny=5 unlock_time=900
account     required      pam_faillock.so
```

Example operations:

```bash
sudo faillock --user alice
sudo faillock --user alice --reset
```

Guidance:

- Use temporary lockouts rather than permanent lockouts.
- Combine with MFA for remote access.
- Monitor lockouts centrally to detect brute-force attempts.

### 2.8 Disabling Unused Accounts

To lock an account without deleting it:

```bash
sudo usermod -L username
sudo passwd -l username
sudo chage -E 0 username
```

To restore access:

```bash
sudo usermod -U username
sudo chage -E -1 username
```

To prevent interactive login for service accounts:

```bash
sudo usermod -s /usr/sbin/nologin serviceuser
# or
sudo usermod -s /sbin/nologin serviceuser
```

### 2.9 Restricting Login Sources

PAM can restrict access by source, service, or user group.

Examples:

- `pam_access.so` can allow or deny users from specific origins.
- `/etc/security/access.conf` defines access rules.
- SSH can further restrict login with `AllowUsers`, `AllowGroups`, `Match`, and source-based conditions.

Example `access.conf` concept:

```ini
+:root:LOCAL
+:@sysadmins:10.0.0.0/24
-:ALL:ALL
```

### 2.10 Environment and Resource Limits

Use `/etc/security/limits.conf` and `pam_limits.so` to restrict user resource consumption.

Examples:

```ini
*               hard    core            0
*               hard    nproc           4096
@developers     soft    nofile          8192
@developers     hard    nofile          16384
```

Why this matters:

- Prevent fork bombs from consuming all processes.
- Constrain accidental or malicious file descriptor exhaustion.
- Reduce data leakage through core dumps.

### 2.11 sudo Best Practices

`sudo` is safer than direct root login when implemented correctly.

Core rules:

- Do not share root credentials.
- Give users named accounts.
- Grant only required commands.
- Require re-authentication where appropriate.
- Log sudo usage.
- Use `visudo` for edits.
- Prefer group-based assignment for manageability.

Inspect current privileges:

```bash
sudo -l
sudo grep -RIn 'NOPASSWD\|ALL=(ALL) ALL\|ALL=(ALL:ALL) ALL' /etc/sudoers /etc/sudoers.d
```

Examples of good sudo patterns:

```sudoers
%webadmins ALL=(root) /bin/systemctl restart nginx, /bin/systemctl status nginx
%dbops ALL=(postgres) /usr/bin/psql
Cmnd_Alias LOGS = /usr/bin/journalctl -u nginx, /usr/bin/journalctl -u sshd
%auditors ALL=(root) LOGS
```

Examples of risky patterns:

```sudoers
alice ALL=(ALL) NOPASSWD: ALL
%dev ALL=(ALL) ALL
bob ALL=(root) /bin/bash
```

Risk explanation:

- `NOPASSWD: ALL` removes a strong control and accountability barrier.
- Full `ALL` access often exceeds role requirements.
- Shell access as root usually enables unrestricted privilege escalation.

### 2.12 su and root Shell Hygiene

Prefer `sudo -i` or tightly scoped `sudo` commands instead of `su -`.

If `su` is required:

- Restrict membership in the relevant admin group.
- Log all attempts.
- Use MFA where feasible.

On many systems, PAM for `su` can limit usage to wheel or sudo members.

### 2.13 Session Logging and Accountability

You cannot secure what you cannot attribute.

Recommended practices:

- Unique user accounts only.
- Centralized auth logs.
- Sudo logging.
- TTY or session recording for high-risk environments.
- Time synchronization with NTP or chrony.

Commands:

```bash
last
lastb
journalctl -u sshd
journalctl _COMM=sudo
who
w
```

### 2.14 Password Storage and Hashes

Modern password hashes are stored in `/etc/shadow`.

Inspect hash types carefully.

Examples:

- `$6$` commonly indicates SHA-512.
- `yescrypt` may appear on newer systems.
- Empty or locked fields deserve review.

Never:

- Copy `/etc/shadow` casually.
- Store password backups in tickets or wiki pages.
- Use weak local scripts for password rotation.

### 2.15 MFA for Linux Logins

MFA is strongly recommended for:

- SSH access from untrusted networks.
- Privileged administrative access.
- VPN-connected jump hosts.

Common approaches:

- TOTP through `pam_google_authenticator.so`.
- Enterprise identity via SSSD with MFA-backed IdP.
- Hardware-backed SSH keys with FIDO2.

### 2.16 User Security Checklist

- Review users and groups monthly.
- Remove dormant accounts quickly.
- Lock service accounts to non-interactive shells.
- Enforce password quality.
- Enforce password aging where policy requires it.
- Enable lockouts for repeated failures.
- Restrict sudo to role-specific commands.
- Log authentication and privilege use.
- Prefer MFA for remote administration.
- Avoid shared credentials.

### 2.17 Example New Admin Workflow

1. Create named user account.
2. Add user to a limited admin group.
3. Provision SSH key.
4. Enroll MFA.
5. Set password aging policy.
6. Confirm `sudo -l` shows only intended commands.
7. Record approval and ticket linkage.
8. Review access at the next recertification cycle.

### 2.18 Summary

Linux user security depends on disciplined account lifecycle management, a correctly configured PAM stack, strong password handling, tight sudo policy, and clear auditability.

When identity is strong, many later controls become far more effective.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.2 User Security Checklist

- Remove or lock dormant accounts.
- Set service accounts to non-interactive shells.
- Review sudo and admin group membership.
- Confirm MFA for remote admins.
- Review password aging exceptions.
- Review failed login patterns.
- Review shared service credentials.
- Confirm SSH keys are rotated according to policy.
- Confirm emergency accounts are documented and controlled.
- Confirm vendor accounts have expiration dates.

### B.1 Identity and Account Commands

```bash
getent passwd
getent group
last
lastb
lastlog
faillog -a
passwd -S username
chage -l username
id username
who
w
sudo -l
```

### B.2 PAM and Password Policy Commands

```bash
grep -RIn 'pam_' /etc/pam.d
cat /etc/security/pwquality.conf
grep -E '^(PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE|UMASK|ENCRYPT_METHOD)' /etc/login.defs
faillock --user username
faillock --user username --reset
```

### C.2 User Security

11. What is PAM?
12. What are the four common PAM module types?
13. What does `required` mean in a PAM stack?
14. What does `sufficient` mean in a PAM stack?
15. Why is `pam_faillock` useful?
16. What settings in `/etc/login.defs` are commonly reviewed?
17. Why is password length more important than simplistic complexity rules?
18. When should an account expiration date be used?
19. Why should service accounts usually have non-interactive shells?
20. Why are shared admin accounts dangerous?
21. Why is `visudo` preferred for sudoers changes?
22. What makes `NOPASSWD: ALL` risky?
23. Why is MFA strongly recommended for remote administration?
24. Why should root SSH login be disabled?
25. Why should you review dormant accounts regularly?
