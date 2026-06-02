# File System Security

File system security defines who can read, write, execute, or modify data and binaries.

It is one of the oldest Linux security domains, yet many compromises still begin with poor ownership, unsafe permissions, or integrity blind spots.

### 3.1 Review Permissions Basics

Standard permissions include read, write, and execute for:

- user
- group
- others

Inspect permissions with:

```bash
ls -l
namei -om /path/to/file
stat /path/to/file
find /etc -maxdepth 1 -type f -printf '%m %u %g %p\n'
```

Common notation:

| Octal | Meaning |
| --- | --- |
| 644 | Owner read/write, group read, others read |
| 640 | Owner read/write, group read, others none |
| 600 | Owner read/write only |
| 755 | Owner full, group read/execute, others read/execute |
| 700 | Owner full only |

Secure default guidance:

- Configuration files should rarely be world-writable.
- Private keys should usually be `600`.
- Home directories should usually be `700` or `750`.
- Sensitive application configs should not be readable by all users.

### 3.2 Ownership Review

Ownership problems can silently create privilege boundaries that do not behave as expected.

Review for:

- Files owned by deleted users.
- Root-owned files inside user-controlled directories.
- Application directories writable by generic groups.
- Unexpected ownership changes after deployments.

Commands:

```bash
find / -xdev -nouser -o -nogroup
find /var/www -type f -printf '%u:%g %m %p\n' | sort
sudo rpm -Va    # RHEL-like integrity signals
sudo debsums -s # Debian-like where available
```

### 3.3 World-Writable Files and Directories

World-writable paths are dangerous because any local user may tamper with content, plant files, or manipulate execution flow.

Find them carefully:

```bash
find / -xdev -type d -perm -0002 -print
find / -xdev -type f -perm -0002 -print
```

Review each result.

Questions to ask:

- Is the path expected to be writable by everyone?
- Does the directory also have the sticky bit?
- Can files placed here influence privileged processes?

### 3.4 SUID and SGID

SUID means an executable runs with the effective user ID of its owner.

SGID means an executable runs with the effective group ID of its owner.

These bits are useful but high risk.

Examples of legitimate binaries may include:

- `passwd`
- `sudo`
- `mount` on some systems
- `su`

Find SUID and SGID files:

```bash
find / -xdev -perm -4000 -type f -print
find / -xdev -perm -2000 -type f -print
```

Risks:

- Vulnerable SUID binaries can become privilege escalation paths.
- Misowned or custom SUID tools are major red flags.
- SGID on data directories can unintentionally widen access.

Operational guidance:

- Keep the SUID/SGID inventory small.
- Investigate any custom or third-party SUID binaries.
- Remove SUID where not required.
- Monitor changes to these files.

### 3.5 Sticky Bit

The sticky bit on a directory means users can create files there but typically cannot delete files owned by others.

This is why `/tmp` is usually `1777`.

Example:

```bash
stat -c '%a %n' /tmp
chmod 1777 /shared/dropbox
```

Do not confuse sticky bit protection with strong confidentiality.

A sticky directory prevents some deletion abuse, not arbitrary reading.

### 3.6 umask

`umask` determines default permissions for newly created files and directories.

Inspect current shell setting:

```bash
umask
umask -S
```

Common values:

| umask | Effect |
| --- | --- |
| 022 | Files commonly 644, directories 755 |
| 027 | Files commonly 640, directories 750 |
| 077 | Files commonly 600, directories 700 |

Use stricter masks for:

- administrators
- security tooling
- private key generation
- sensitive data processing jobs

### 3.7 Immutable and Append-Only Attributes

Linux extended attributes can add extra protection on supported filesystems.

Useful flags with `chattr`:

| Flag | Meaning |
| --- | --- |
| +i | Immutable |
| +a | Append-only |

Examples:

```bash
sudo chattr +i /etc/passwd
sudo chattr +i /etc/shadow
sudo chattr +a /var/log/secure
lsattr /etc/passwd /etc/shadow /var/log/secure
```

Warnings:

- Immutable files cannot be modified, renamed, or deleted until the flag is removed.
- Package updates may fail if critical files are immutable.
- Attackers with full root may remove the flag unless additional controls exist.

Good use cases:

- Protecting selected configuration files from accidental change.
- Adding friction to log tampering.
- Guarding emergency break-glass artifacts during incident response.

### 3.8 ACLs Deep Dive

Access Control Lists extend beyond the traditional owner-group-other model.

ACLs are useful when several teams require different rights on shared paths without creating awkward group sprawl.

Core commands:

```bash
getfacl /srv/shared
setfacl -m u:alice:rwx /srv/shared
setfacl -m g:analysts:rx /srv/reports
setfacl -R -m d:g:developers:rwx /srv/devdrop
```

ACL concepts:

| Item | Meaning |
| --- | --- |
| access ACL | Permissions on the object itself |
| default ACL | Permissions inherited by new children in a directory |
| mask | Maximum effective ACL permission for named users and groups |

Example workflow:

```bash
mkdir -p /srv/project
chown root:project /srv/project
chmod 2770 /srv/project
setfacl -m g:auditors:rx /srv/project
setfacl -m d:g:project:rwx /srv/project
getfacl /srv/project
```

ACL cautions:

- ACLs can confuse troubleshooting when engineers only inspect `ls -l`.
- Backup and restore tools must preserve ACLs.
- Complex ACLs can become invisible privilege creep.

### 3.9 Mount Options

Mount options can substantially reduce local abuse opportunities.

Especially review:

- `nodev`
- `nosuid`
- `noexec`
- `ro`

Example `/etc/fstab` ideas:

```fstab
UUID=xxxx /tmp      ext4 defaults,nodev,nosuid,noexec 0 0
UUID=yyyy /var/tmp  ext4 defaults,nodev,nosuid,noexec 0 0
UUID=zzzz /home     ext4 defaults,nodev               0 0
```

Benefits:

- `nodev` blocks device file interpretation.
- `nosuid` disables SUID/SGID semantics on the mount.
- `noexec` reduces direct binary execution from that mount.
- `ro` protects static partitions from change.

Operational caveat:

Some applications break under aggressive `noexec` or `ro` policies.

Test carefully.

### 3.10 Temporary Directories and Race Conditions

Temporary files are a common source of insecure behavior.

Good practice:

- Use secure file creation APIs.
- Avoid predictable names.
- Avoid running privileged scripts that trust user-controlled temp paths.
- Review cron jobs and deployment scripts for unsafe symlink handling.

### 3.11 File Integrity Monitoring

File integrity monitoring detects unauthorized changes to system files, binaries, configs, and selected data.

Two well-known tools:

- AIDE
- Tripwire

#### AIDE

AIDE creates a baseline database of file metadata and cryptographic hashes.

Typical flow:

```bash
sudo aide --init
sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
sudo aide --check
```

What AIDE can watch:

- file permissions
- ownership
- inode changes
- timestamps
- hashes
- ACLs on supported setups

Recommended targets:

- `/bin`
- `/sbin`
- `/usr/bin`
- `/usr/sbin`
- `/etc`
- bootloader and kernel paths
- security tooling configs

Best practices:

- Store the baseline securely.
- Rebuild the baseline only after approved changes.
- Alert on deviations.
- Review reports with context from package updates.

#### Tripwire

Tripwire provides similar integrity checking with policy-based monitoring.

Considerations:

- Strong policy tuning is essential.
- Unreviewed noise leads to alert fatigue.
- Signing and secure policy management matter.

### 3.12 Detecting Dangerous Permission Patterns

Look for:

```bash
find /etc -type f -perm -002 -ls
find /usr/local/bin -type f -perm -4000 -ls
find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -ls
```

Interpretation:

- World-writable files in `/etc` are usually severe findings.
- Custom SUID files in `/usr/local/bin` require strong justification.
- World-writable directories without sticky bit invite abuse.

### 3.13 Backup Security

Backups can bypass file-system protections if they are readable to the wrong users.

Protect backups by:

- Encrypting backup archives.
- Restricting backup operator permissions.
- Limiting restore rights.
- Testing integrity and access controls on restored files.

### 3.14 Sensitive Files to Review

| Path | Why It Matters |
| --- | --- |
| /etc/passwd | Account definitions |
| /etc/shadow | Password hashes |
| /etc/sudoers | Privilege policy |
| /root/.ssh | Root remote access material |
| /home/*/.ssh | User key material |
| /etc/ssh/sshd_config | Remote access policy |
| /boot | Boot security and kernel artifacts |
| /var/log | Security and incident evidence |
| /etc/fstab | Mount security posture |

### 3.15 Practical Review Routine

A weekly permissions review might include:

1. Export SUID and SGID inventory.
2. Compare it with the known-good baseline.
3. Search for new world-writable paths.
4. Review ownership changes in application directories.
5. Confirm AIDE or Tripwire status.
6. Verify log directories remain protected.

### 3.16 Summary

File system security is not just about `chmod`.

It includes permissions, ownership, mount semantics, ACL discipline, integrity monitoring, and safe handling of sensitive paths.

Done well, it reduces both accidental exposure and attacker privilege escalation.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.3 File System Security Checklist

- Review world-writable directories.
- Confirm sticky bit on shared temp directories.
- Review SUID and SGID inventory.
- Review ownership drift in application paths.
- Review ACL usage and necessity.
- Confirm sensitive files are not overly readable.
- Review mount options for `/tmp` and other risky paths.
- Review immutable or append-only flags where used.
- Review AIDE or Tripwire status.
- Confirm backup archives are access-controlled.

### B.3 File Permission Commands

```bash
ls -l /path
stat /path
namei -om /path/to/file
find / -xdev -perm -4000 -type f -print
find / -xdev -perm -2000 -type f -print
find / -xdev -type d -perm -0002 -print
find / -xdev -type f -perm -0002 -print
umask
getfacl /path
setfacl -m u:alice:rwx /path
lsattr /path
chattr +i /path
```

### C.3 File System Security

26. What does the sticky bit do on a directory?
27. Why are world-writable directories risky?
28. Why are SUID binaries dangerous?
29. What is the difference between DAC and ACLs?
30. Why can ACLs become hard to troubleshoot?
31. What does `chattr +i` do?
32. What does `chattr +a` do?
33. Why are mount options like `nosuid` and `nodev` important?
34. What does `noexec` protect against, and what does it not prevent?
35. Why is file integrity monitoring valuable?
36. Why should backup permissions be reviewed?
37. Why are orphaned file owners a concern?
38. Why should private keys usually be mode `600`?
39. What is the purpose of `umask`?
40. Why are temporary files often security-sensitive?
