# Log Analysis

## 10.1 Common log locations

| Path | Typical content |
|---|---|
| `/var/log/messages` | General system messages on many distros |
| `/var/log/syslog` | General syslog on Debian family |
| `/var/log/auth.log` | Authentication events |
| `/var/log/secure` | Auth events on RHEL family |
| `/var/log/kern.log` | Kernel messages on some systems |
| `/var/log/dmesg` | Boot kernel ring buffer snapshot |
| `/var/log/nginx/` | NGINX access and error logs |
| `/var/log/httpd/` | Apache logs |
| `/var/log/mysql/` | MySQL or MariaDB logs |
| `/var/log/audit/` | SELinux and audit data |

## 10.2 `journalctl` essentials

Examples:

```bash
journalctl -b --no-pager
journalctl -b -1 --no-pager
journalctl -u sshd --since '1 hour ago' --no-pager
journalctl -p err..alert --since today --no-pager
journalctl -k --no-pager
journalctl -f
```

## 10.3 High-value journal filters

- by boot: `-b`
- by service: `-u name`
- by priority: `-p err..alert`
- by time: `--since`, `--until`
- by kernel only: `-k`
- by executable or PID with `SYSTEMD_UNIT` or `_PID` fields

## 10.4 Correlation mindset

Correlate logs by:

- timestamp.
- hostname.
- request ID.
- transaction ID.
- PID or unit.
- user session.
- source IP.

## 10.5 grep examples

```bash
grep -i 'error' /var/log/syslog | tail -50
grep -Rni 'permission denied' /var/log
```

## 10.6 awk examples

Count repeated messages:

```bash
awk '{print $5}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head
```

Summarize status codes:

```bash
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -nr
```

## 10.7 Common log patterns

Pattern | Interpretation
---|---
`Out of memory` | Memory pressure or limit hit
`segfault` | Application crash or bad library
`permission denied` | Access control failure
`No space left on device` | Blocks or inodes exhausted
`Connection refused` | No listener or active reject
`timed out` | Slow dependency or dropped packets
`I/O error` | Device, path, or filesystem issue
`AVC denied` | SELinux blocked operation

## 10.8 Kernel logs matter

Always inspect kernel messages when symptoms are low-level.

Examples:

```bash
journalctl -k --since '2 hours ago' --no-pager
dmesg -T | tail -200
```

## 10.9 Time alignment

Beware:

- UTC vs local time.
- clock drift.
- daylight saving changes.
- rotated logs.

## 10.10 Access log plus error log pairing

For web systems:

- Use access logs to find failed requests.
- Use timestamps to align error logs.
- Match upstream latency and status codes.

## 10.11 Audit logs

For security or SELinux:

```bash
ausearch -m AVC -ts recent
aureport -au
```

## 10.12 Structured logs

If JSON logs exist, prefer `jq`.

Example:

```bash
jq -r '.level + " " + .message' app.log | head
```

## 10.13 Log rotation pitfalls

Check:

- file keeps growing after rotation due to open fd.
- rotation frequency too low.
- compression delayed.
- service lacks `USR1` or reopen signal.

## 10.14 Journal disk usage

Check:

```bash
journalctl --disk-usage
```

Vacuum examples:

```bash
journalctl --vacuum-time=14d
journalctl --vacuum-size=500M
```

## 10.15 Practical log workflow

1. Define time window.
2. Identify affected service.
3. Pull error logs.
4. Pull kernel logs.
5. Correlate with metrics.
6. Correlate with changes.
7. Confirm the first bad event.

---
