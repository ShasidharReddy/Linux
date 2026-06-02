# Package Issues

## 9.1 Common package problems

- Broken dependencies.
- Repository metadata errors.
- GPG signature failures.
- Held packages.
- Version conflicts.
- Interrupted package transaction.
- File conflicts between packages.

## 9.2 First commands

Debian or Ubuntu:

```bash
apt update
apt install -f
apt-cache policy pkgname
dpkg --audit
```

RHEL family:

```bash
dnf repolist
dnf check
rpm -Va
```

## 9.3 Repository issues

Common causes:

- DNS failure.
- TLS or certificate issue.
- Proxy misconfiguration.
- Mirror unavailable.
- Incorrect repository URL.
- Subscription or entitlement issue.

## 9.4 GPG key failures

Symptoms:

- Repository metadata not trusted.
- Package signatures rejected.

Check:

- correct key installed.
- repo metadata signed by expected key.
- system time accurate.

## 9.5 Broken dependency chains

Debian tools:

```bash
apt --fix-broken install
dpkg --configure -a
```

RHEL tools:

```bash
dnf repoquery --unsatisfied
rpm -q --whatrequires pkgname
```

## 9.6 Held packages and pinning

Debian:

```bash
apt-mark showhold
apt-cache policy pkgname
```

Version pinning can cause confusing partial upgrades.

## 9.7 File conflicts

RPM example checks:

```bash
rpm -qf /path/to/file
rpm -Va
```

## 9.8 Interrupted updates

Symptoms:

- Lock files remain.
- package database inconsistent.
- postinstall scripts incomplete.

Debian recovery:

```bash
dpkg --configure -a
apt install -f
```

RHEL recovery:

```bash
rpm --rebuilddb
dnf check
```

## 9.9 Kernel package caution

- Keep at least one known-good kernel.
- Verify `/boot` has space before updates.
- Regenerate bootloader config if needed.

## 9.10 Verify package ownership

Debian:

```bash
dpkg -S /path/to/file
```

RHEL:

```bash
rpm -qf /path/to/file
```

## 9.11 Package issue checklist

- Verify repo reachability.
- Verify GPG trust.
- Verify time sync.
- Check held packages.
- Repair broken dependencies.
- Audit package DB integrity.
- Confirm enough disk space in `/` and `/boot`.

---
