# Auditing & Compliance

Auditing and compliance provide visibility and evidence.

Hardening without auditability leaves you blind to drift, misuse, and control failure.

### 9.1 auditd Overview

`auditd` records security-relevant events from the Linux Audit Framework.

It is useful for:

- command execution tracking
- file access monitoring
- privilege use
- identity changes
- policy verification

Check service state:

```bash
sudo systemctl status auditd
sudo auditctl -s
```

### 9.2 Key auditd Components

| Component | Purpose |
| --- | --- |
| auditd | Daemon collecting audit events |
| auditctl | Manage runtime rules |
| augenrules | Load persistent rule files |
| ausearch | Search audit logs |
| aureport | Generate summary reports |

### 9.3 Example Audit Rules

Common rule targets:

- `/etc/passwd`
- `/etc/shadow`
- `/etc/sudoers`
- `/etc/ssh/sshd_config`
- privileged command execution
- module loading
- time changes

Example ideas:

```bash
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k scope
-w /etc/ssh/sshd_config -p wa -k sshd_config
-a always,exit -F arch=b64 -S execve -k exec
-a always,exit -F arch=b64 -S sethostname -S adjtimex -k time-change
```

Be selective.

Too many noisy rules can overwhelm storage and analysis.

### 9.4 ausearch

`ausearch` helps query audit logs efficiently.

Examples:

```bash
sudo ausearch -k identity
sudo ausearch -m USER_LOGIN -ts today
sudo ausearch -m avc -ts recent
sudo ausearch -ua 1001 -ts yesterday
```

Questions it can answer:

- Who modified a sensitive file?
- Which user logged in recently?
- Which command executed at a given time?
- Which SELinux denials occurred?

### 9.5 aureport

`aureport` summarizes audit activity.

Examples:

```bash
sudo aureport -au
sudo aureport -x
sudo aureport -l
sudo aureport -f
```

Use cases:

- summarize authentication results
- list executed commands
- review file access reports
- detect spikes or anomalies

### 9.6 Protecting Audit Logs

Audit data is valuable evidence.

Recommendations:

- forward logs centrally
- restrict local access
- protect storage from tampering
- monitor audit daemon failures
- ensure clocks are synchronized

### 9.7 CIS Benchmarks

CIS Benchmarks provide prescriptive configuration guidance for operating systems and applications.

Benefits:

- standardized baseline
- useful for audit preparation
- good starting point for policy discussions

Cautions:

- not every recommendation fits every workload
- some settings trade usability for control
- exceptions should be documented and approved

### 9.8 OpenSCAP

OpenSCAP can evaluate systems against security content such as SCAP benchmarks.

It is often used for:

- baseline assessment
- compliance reporting
- remediation guidance

High-level workflow:

1. select the correct content for the OS version
2. run an evaluation
3. review failed rules
4. remediate carefully
5. re-scan

### 9.9 Lynis

Lynis is a widely used security auditing tool for Unix-like systems.

It checks:

- system hardening state
- services
- auth settings
- file permissions
- kernel settings
- networking posture

Operational use:

- baseline new hosts
- compare results over time
- use findings to prioritize hardening work

### 9.10 Compliance Is Not Security

Compliance can help structure control implementation.

It does not guarantee actual safety.

Examples:

- A compliant server with stale exceptions may still be exploitable.
- A passing benchmark can hide insecure custom applications.
- Audit evidence without operational review has limited value.

### 9.11 Recommended Audit Topics

- authentication and login events
- sudo usage
- user and group changes
- kernel module operations
- system time changes
- firewall rule changes
- SSH configuration changes
- integrity tool results

### 9.12 Example Weekly Audit Review

1. Review failed and successful privileged logins.
2. Review sudo execution summaries.
3. Review identity file changes.
4. Review SELinux or AppArmor denials.
5. Review new listening ports.
6. Review benchmark drift or scan findings.

### 9.13 Summary

Auditing and compliance controls make hardening measurable.

Use `auditd`, `ausearch`, `aureport`, benchmark frameworks, and targeted reviews to turn configuration into verifiable security posture.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.9 Auditing Checklist

- Confirm auditd is active.
- Review audit rule coverage.
- Review audit disk space settings.
- Review `ausearch` reports for identity changes.
- Review `aureport` command summaries.
- Forward logs centrally.
- Review benchmark findings.
- Review compliance exceptions.
- Confirm clocks are synchronized.
- Protect audit logs from tampering.

### B.9 Auditing Commands

```bash
systemctl status auditd
auditctl -s
ausearch -k identity
ausearch -m USER_LOGIN -ts today
aureport -au
aureport -x
aureport -f
```

### C.9 Auditing and Compliance

101. What does auditd record?
102. Why should audit rules focus on high-value events?
103. What is the difference between `ausearch` and `aureport`?
104. Why should audit logs be forwarded centrally?
105. What is the value of CIS Benchmarks?
106. Why is compliance not the same as security?
107. What does OpenSCAP help with?
108. Why are benchmark exceptions inevitable in some environments?
109. Why must compliance findings be reviewed in operational context?
110. Why do synchronized clocks matter in audit analysis?
