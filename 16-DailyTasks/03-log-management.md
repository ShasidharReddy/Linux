# Log Management and Analysis

This guide covers log review workflows, rotation, cleanup, and high-value analysis commands.

## 3.1 Log locations to know

### Common files

- `/var/log/syslog`
- `/var/log/messages`
- `/var/log/auth.log`
- `/var/log/secure`
- `/var/log/kern.log`
- `/var/log/dmesg`
- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`
- `/var/log/httpd/access_log`
- `/var/log/httpd/error_log`
- `/var/log/mysql/error.log`
- `/var/log/postgresql/`
- `/var/log/audit/audit.log`

### Quick listing

```bash
sudo find /var/log -maxdepth 2 -type f | sort
```

## 3.2 Using `journalctl`

### Current boot logs

```bash
journalctl -xb --no-pager
```

### Follow logs live

```bash
journalctl -f
journalctl -u nginx -f
```

### Show last hour

```bash
journalctl --since '1 hour ago' --no-pager
```

### Show errors only

```bash
journalctl -p err --since today --no-pager
```

### Show logs for a service

```bash
journalctl -u sshd --no-pager
journalctl -u docker --since yesterday --no-pager
```

### Show previous boot logs

```bash
journalctl -b -1 --no-pager
```

## 3.3 Grep patterns for error hunting

```bash
sudo grep -RiE 'error|fail|fatal|panic|segfault|denied|timeout|refused' /var/log | head -200
```

```bash
journalctl --since today --no-pager | grep -Ei 'oom|killed process|segfault|i/o error|read-only file system'
```

### Useful patterns

- `error`
- `failed`
- `fatal`
- `panic`
- `exception`
- `segfault`
- `oom`
- `out of memory`
- `denied`
- `permission denied`
- `connection refused`
- `timed out`
- `reset by peer`
- `disk full`
- `read-only file system`

## 3.4 Log rotation with `logrotate`

### Example application rotation config

```conf
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 myapp myapp
    sharedscripts
    postrotate
        systemctl reload myapp >/dev/null 2>&1 || true
    endscript
}
```

### Test `logrotate`

```bash
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.conf
```

### Inspect rotation status

```bash
sudo cat /var/lib/logrotate/status
```

## 3.5 Centralized logging setup ideas

- rsyslog forwarding.
- syslog-ng forwarding.
- Filebeat to Elasticsearch.
- Fluent Bit to Loki or OpenSearch.
- journald remote forwarding.
- Cloud-native agents for cloud VMs.

### rsyslog forwarding example

```conf
*.* action(type="omfwd" target="loghub.example.com" port="514" protocol="tcp")
```

### Restart rsyslog

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog --no-pager
```

## 3.6 Real-world log analysis one-liners

### Top 20 IPs hitting Nginx access log

```bash
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -20
```

### Top 20 URLs

```bash
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -20
```

### Count HTTP status codes

```bash
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -nr
```

### Show 5xx responses

```bash
awk '$9 ~ /^5/' /var/log/nginx/access.log | tail -50
```

### Find auth failures

```bash
sudo grep -Ei 'failed password|authentication failure|invalid user' /var/log/auth.log /var/log/secure 2>/dev/null
```

### Top invalid usernames attempted

```bash
sudo grep -h 'Invalid user' /var/log/auth.log /var/log/secure 2>/dev/null | awk '{print $(NF-2)}' | sort | uniq -c | sort -nr | head
```

### Find OOM kills

```bash
journalctl -k --since yesterday --no-pager | grep -Ei 'killed process|out of memory|oom'
```

### Detect disk I/O errors

```bash
journalctl -k --since today --no-pager | grep -Ei 'I/O error|buffer I/O error|blk_update_request|EXT4-fs error|XFS'
```

### Extract slow queries from MySQL log

```bash
grep -Ei 'Query_time|^# Time|^SET timestamp|^[A-Z]' /var/log/mysql/mysql-slow.log | tail -100
```

### Count SSH login successes by user

```bash
sudo grep 'Accepted ' /var/log/auth.log /var/log/secure 2>/dev/null | awk '{print $(NF-5)}' | sort | uniq -c | sort -nr
```

### Show journald disk usage

```bash
journalctl --disk-usage
```

## 3.7 Log retention planning

| Log type | Typical retention | Notes |
|---|---|---|
| Security logs | 90 to 365 days | Compliance-driven |
| Syslog | 14 to 30 days | Depends on disk |
| App logs | 7 to 30 days | Ship centrally |
| Access logs | 14 to 90 days | Depends on traffic |
| Audit logs | 180+ days | Often immutable storage |

## 3.8 Journald maintenance

```bash
sudo journalctl --vacuum-time=14d
sudo journalctl --vacuum-size=1G
sudo journalctl --verify
```

## 3.9 Log troubleshooting workflow

- Confirm the affected host and time range.
- Identify the impacted service.
- Check application logs.
- Check service logs with `journalctl -u`.
- Check kernel logs.
- Correlate with deployment time.
- Correlate with config changes.
- Correlate with traffic spikes.
- Capture evidence before cleanup.

---
