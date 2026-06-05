# Log Management

---

Logs are the primary evidence trail for operations, failures, access, and system behavior.

## 8.1 Why logs matter
Logs help you:
- Diagnose incidents.
- Investigate security events.
- Audit changes and access.
- Understand performance trends.
- Satisfy compliance requirements.

## 8.2 Common log locations under /var/log
Examples:
- `/var/log/messages`
- `/var/log/syslog`
- `/var/log/auth.log`
- `/var/log/secure`
- `/var/log/kern.log`
- `/var/log/dmesg`
- `/var/log/cron`
- `/var/log/nginx/`
- `/var/log/httpd/`
- `/var/log/audit/`
- `/var/log/journal/`

Exact files vary by distribution.

## 8.3 journald
`journald` stores structured logs for systemd-managed systems.

Useful commands:

```bash
journalctl -b
journalctl -u sshd
journalctl -p warning
journalctl --disk-usage
journalctl --vacuum-time=7d
journalctl --vacuum-size=1G
```

Configuration file:

```text
/etc/systemd/journald.conf
```

Important settings:
- `Storage=`
- `SystemMaxUse=`
- `RuntimeMaxUse=`
- `Compress=`
- `ForwardToSyslog=`

Persistent logging is typically enabled by storing journals under `/var/log/journal/`.

## 8.4 syslog and rsyslog
Traditional syslog-based logging is still common.
`rsyslog` is a widely used implementation.

Service names often include:
- `rsyslog.service`
- `syslog-ng.service`

Typical config locations:
- `/etc/rsyslog.conf`
- `/etc/rsyslog.d/*.conf`

Example rule sending auth logs to a dedicated file:

```conf
authpriv.*    /var/log/auth-custom.log
```

Reload after changes:

```bash
sudo systemctl restart rsyslog
```

## 8.5 Classic log viewing tools
Use these tools constantly:

```bash
tail -f /var/log/syslog
tail -100 /var/log/messages
less /var/log/auth.log
grep -i error /var/log/nginx/error.log
awk '/Failed password/ {print}' /var/log/auth.log
```

Use compression-aware tools for rotated logs:

```bash
zgrep -i panic /var/log/messages.*.gz
zless /var/log/syslog.2.gz
```

## 8.6 logrotate
`logrotate` manages log rotation, compression, retention, and post-rotate actions.

Main config locations:
- `/etc/logrotate.conf`
- `/etc/logrotate.d/`

Example configuration:

```conf
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    missingok
    notifempty
    create 0640 myapp adm
    postrotate
        systemctl reload myapp >/dev/null 2>&1 || true
    endscript
}
```

Test a config:

```bash
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.conf
```

## 8.7 Centralized logging
Centralized logging is strongly recommended for production.

Benefits:
- Long-term retention.
- Faster incident triage.
- Cross-host correlation.
- Better security monitoring.
- Reduced risk when a host is lost.

Common stacks:
- rsyslog to central syslog server.
- ELK or Elastic Stack.
- Graylog.
- Loki and Grafana.
- Splunk.
- Cloud-native logging platforms.

## 8.8 Logging design principles
- Log in structured formats where possible.
- Synchronize time with NTP.
- Protect logs from unauthorized modification.
- Retain logs based on policy.
- Avoid logging secrets.
- Separate application, access, and audit logs.
- Ensure central forwarding on critical systems.

## 8.9 Common log analysis patterns
Authentication failures:

```bash
grep "Failed password" /var/log/auth.log
journalctl -u sshd | grep "Failed password"
```

Kernel disk errors:

```bash
dmesg | grep -i -E "error|fail|xfs|ext4|nvme|sda"
```

Service crash loops:

```bash
journalctl -u myapp --since -1h
systemctl status myapp
```

OOM kills:

```bash
journalctl -k | grep -i oom
dmesg | grep -i "killed process"
```

## 8.10 Disk usage and retention control
Check journal disk use:

```bash
journalctl --disk-usage
```

Vacuum journal size:

```bash
sudo journalctl --vacuum-size=500M
```

Find large logs:

```bash
find /var/log -type f -exec du -h {} + | sort -h | tail -20
```

## 8.11 Security-sensitive logs
Pay special attention to:
- SSH authentication logs.
- sudo usage.
- auditd logs.
- kernel security events.
- firewall logs.
- SELinux or AppArmor denials.

Typical paths:
- `/var/log/auth.log`
- `/var/log/secure`
- `/var/log/audit/audit.log`

## 8.12 Log management best practices
- Enable persistent journaling where useful.
- Rotate logs before disks fill.
- Forward critical logs centrally.
- Use retention policies.
- Monitor logs for failed auth and service crashes.
- Restrict permissions on sensitive logs.
- Time sync all hosts.
- Test post-rotate hooks.

---

## 8.6 Logging commands reference
- `journalctl`
- `journalctl -b`
- `journalctl -k`
- `journalctl -u sshd`
- `journalctl -p warning`
- `journalctl -f`
- `journalctl --disk-usage`
- `journalctl --vacuum-time=7d`
- `tail -f /var/log/syslog`
- `tail -100 /var/log/messages`
- `grep -i error /var/log/nginx/error.log`
- `zgrep -i panic /var/log/messages.*.gz`
- `logrotate -d /etc/logrotate.conf`
- `logrotate -f /etc/logrotate.conf`
- `ausearch -m avc -ts recent`
- `aureport -au`

---

## B.6 Logging quick reminders
- Centralize logs for important hosts.
- Enable persistent journals where needed.
- Rotate logs before the disk fills.
- Keep time synchronized.
- Separate noisy debug logs from security-relevant logs.
- Avoid logging secrets and tokens.
- Limit permissions on audit and auth logs.
- Use `journalctl --since` to narrow incident windows.
- Search rotated logs with `zgrep`.
- Test `logrotate` changes with debug mode.
- Verify application reload behavior after rotation.
- Use structured logging where possible.
- Preserve logs during incident response.
- Document retention policies.
- Monitor for repeated authentication failures.

---

### Logging inspection examples
```bash
journalctl --list-boots
journalctl -u sshd -S today
awk '{print $1,$2,$3,$11}' /var/log/nginx/access.log | head
find /var/log -type f -size +100M -ls
```

---

## B.29 More logging examples
```bash
journalctl --since "1 hour ago" --until now
journalctl -u nginx -g error
grep -R "CRON" /var/log 2>/dev/null | tail
find /var/log -type f -mtime -7 -ls | head
```
- Search by time window first when investigating incidents.
- Narrow by unit, priority, or keyword to reduce noise.
- Establish log retention to satisfy operational and compliance needs.
- Separate application verbosity controls from platform log retention.
- Excessive debug logging can create its own outage through disk pressure.
- Structured fields make centralized search much easier.
- Ensure time zones are consistent across log pipelines.
- Preserve security logs long enough for investigations.
- Verify forwarded logs actually arrive at the destination.
- Periodically test log pipeline failure scenarios.
