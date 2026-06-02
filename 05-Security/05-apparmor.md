# AppArmor

AppArmor is another Linux mandatory access control framework.

It is especially common on Ubuntu and SUSE-based environments.

Unlike SELinux, which heavily uses labels and types, AppArmor focuses on path-based profiles that define what applications can access.

### 5.1 AppArmor Concepts

Key components:

- profiles
- complain mode
- enforce mode
- abstractions
- hats and subprofiles in advanced use cases

Modes:

| Mode | Meaning |
| --- | --- |
| enforce | Violations are blocked |
| complain | Violations are allowed but logged |
| disable | Profile is not applied |

Useful commands:

```bash
sudo aa-status
sudo apparmor_status
```

### 5.2 Why AppArmor Matters

AppArmor can significantly limit damage if a service is compromised.

Examples:

- Restrict a web service to specific files.
- Prevent unexpected shell execution.
- Limit network, mount, or capability usage.
- Reduce data exposure from daemon compromise.

### 5.3 Profile Discovery

Profiles are commonly stored in:

- `/etc/apparmor.d/`

Inspect loaded profiles:

```bash
sudo aa-status
ls -1 /etc/apparmor.d/
```

You may see profiles for:

- `sshd`
- `ntpd`
- `tcpdump`
- browsers
- container runtimes
- custom local services

### 5.4 Profile Structure

A typical profile defines allowed file access, capabilities, network usage, signals, and includes.

Example simplified idea:

```text
/usr/sbin/nginx {
  #include <abstractions/base>
  /usr/sbin/nginx mr,
  /etc/nginx/** r,
  /var/log/nginx/** rw,
  /var/www/** r,
  capability net_bind_service,
  network inet stream,
}
```

Interpretation:

- `r` means read.
- `w` means write.
- `m` often relates to memory map execute/read behavior.
- Rules can apply recursively using globs.

### 5.5 Managing Modes

Switch a profile to complain mode:

```bash
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
```

Switch back to enforce mode:

```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
```

Disable a profile:

```bash
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx
```

Re-enable and load:

```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo systemctl reload apparmor
```

### 5.6 aa-genprof

`aa-genprof` helps create or refine profiles interactively.

Typical workflow:

1. Start `aa-genprof` for a target binary.
2. Exercise the application normally.
3. Review logged events.
4. Accept or refine the proposed rules.

Example:

```bash
sudo aa-genprof /usr/local/bin/myapp
```

Best use case:

- New in-house services without a mature profile.

### 5.7 aa-logprof

`aa-logprof` reads AppArmor logs and suggests profile updates.

Example:

```bash
sudo aa-logprof
```

Best practices:

- Reproduce real application behavior before accepting rules.
- Reject overly broad paths.
- Prefer explicit access over recursive write access when possible.

### 5.8 Creating a Custom Profile

Example process:

1. Identify the executable path.
2. Start with a minimal profile.
3. Include standard abstractions.
4. Allow only required config, runtime, data, and log paths.
5. Allow only required capabilities.
6. Test in complain mode.
7. Move to enforce mode.

Simple custom profile example:

```text
#include <tunables/global>

/usr/local/bin/backup-sync {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  /usr/local/bin/backup-sync mr,
  /etc/backup-sync.conf r,
  /var/log/backup-sync.log rw,
  /srv/backup-source/** r,
  /srv/backup-target/** rw,

  capability dac_read_search,
  network inet stream,
}
```

### 5.9 Reading Logs

AppArmor denials may appear in:

- `journalctl`
- kernel logs
- audit logs on some systems

Examples:

```bash
journalctl -k | grep -i apparmor
journalctl -xe | grep DENIED
```

When analyzing denials, ask:

- Is the app trying to access an intended path?
- Is the path too broad?
- Would allowing it violate least privilege?
- Is the application design itself problematic?

### 5.10 Common AppArmor Rule Types

| Rule Type | Example Purpose |
| --- | --- |
| File path access | Allow config reads or log writes |
| Capability rules | Permit specific kernel capabilities |
| Network rules | Allow socket use by family and type |
| Signal rules | Control process signaling |
| Mount rules | Restrict mount operations |
| ptrace rules | Control debugging and inspection |

### 5.11 Abstractions and Reuse

AppArmor ships with abstractions to reduce repetitive rule writing.

Examples may include:

- `abstractions/base`
- `abstractions/nameservice`
- `abstractions/ssl_certs`
- `abstractions/user-tmp`

Benefits:

- faster profile creation
- more maintainable policies
- fewer missed common access needs

Caution:

Over-including abstractions can widen access more than necessary.

### 5.12 AppArmor vs SELinux

| Area | AppArmor | SELinux |
| --- | --- | --- |
| Model | Path-based | Label-based |
| Operational feel | Often simpler initially | Often more powerful and granular |
| Common environments | Ubuntu, SUSE | RHEL, Fedora, many enterprise systems |
| Persistence model | Profile file driven | Policy and labeling driven |

Neither should be casually disabled.

Use the security framework that fits your distribution and operating model.

### 5.13 AppArmor Hardening Workflow

1. Confirm AppArmor is active.
2. Inventory loaded profiles.
3. Identify unconfined network-facing services.
4. Apply vendor profiles where available.
5. Build custom profiles for local apps.
6. Test in complain mode.
7. Enforce in production.
8. Review logs after deployments.

### 5.14 Common Pitfalls

- Leaving important custom services unconfined.
- Accepting all suggested rules from `aa-logprof`.
- Using recursive write rules where read-only rules would work.
- Forgetting to reload profiles after changes.
- Treating complain mode as a permanent state.

### 5.15 Example Service Review Questions

- Which executable should the profile attach to?
- Which directories must be readable?
- Which files must be writable?
- Does the service need network access?
- Does it need shell execution or child processes?
- Does it need special capabilities?
- Can logs reveal unexpected attempted access?

### 5.16 Summary

AppArmor provides practical process confinement through profiles.

When profiles are minimal, tested, and enforced, AppArmor becomes a strong control against service compromise and lateral access.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.5 AppArmor Checklist

- Confirm AppArmor is active.
- Inventory loaded profiles.
- Identify unconfined network-facing services.
- Move test profiles from complain to enforce mode.
- Review denial logs after application updates.
- Remove broad path rules.
- Reload changed profiles.
- Version-control custom profiles.
- Review capability usage in profiles.
- Confirm abstractions are not overused.

### B.5 AppArmor Commands

```bash
aa-status
apparmor_status
aa-complain /etc/apparmor.d/profile
aa-enforce /etc/apparmor.d/profile
aa-disable /etc/apparmor.d/profile
aa-genprof /usr/local/bin/myapp
aa-logprof
systemctl reload apparmor
```

### C.5 AppArmor

56. How is AppArmor different from SELinux at a high level?
57. What is complain mode?
58. What is enforce mode?
59. Why is path-based confinement operationally attractive?
60. What does `aa-genprof` help with?
61. What does `aa-logprof` do?
62. Why can broad recursive write rules be dangerous?
63. Why should unconfined services be reviewed?
64. Why must profiles be reloaded after change?
65. Why should complain mode not be permanent for sensitive services?
