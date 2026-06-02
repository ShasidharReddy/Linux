# Cron Job Management

This guide covers cron creation, debugging, validation, and systemd timer alternatives.

## 13.1 Creating and editing cron jobs

### Edit current user crontab

```bash
crontab -e
```

### List current user crontab

```bash
crontab -l
```

### Edit root crontab

```bash
sudo crontab -e
sudo crontab -l
```

### System-wide cron files

- `/etc/crontab`
- `/etc/cron.d/`
- `/etc/cron.daily/`
- `/etc/cron.hourly/`
- `/etc/cron.weekly/`
- `/etc/cron.monthly/`

## 13.2 Cron expression examples

| Expression | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Hourly |
| `0 2 * * *` | Daily at 02:00 |
| `30 2 * * 0` | Weekly Sunday 02:30 |
| `0 1 1 * *` | First day of month 01:00 |
| `15 9 * * 1-5` | Weekdays 09:15 |

### Example jobs

```cron
*/5 * * * * /usr/local/bin/check-disk.sh
0 2 * * * /usr/local/bin/db-backup.sh >> /var/log/db-backup.log 2>&1
15 3 * * 0 /usr/local/bin/log-cleanup.sh
```

## 13.3 Cron job debugging

### Check cron service

```bash
systemctl status cron --no-pager || systemctl status crond --no-pager
```

### Review cron logs

```bash
journalctl -u cron --since today --no-pager || journalctl -u crond --since today --no-pager
```

### Environment issues

- Cron uses a minimal environment.
- Use full paths to commands.
- Set `PATH` explicitly.
- Redirect output to log files.
- Confirm script is executable.
- Confirm shebang is correct.

### Example robust cron entry

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=ops@example.com

0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Test script as cron would run it

```bash
env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin /bin/bash -c '/usr/local/bin/backup.sh'
```

## 13.4 systemd timers as an alternative

### Example service unit

```ini
[Unit]
Description=Run backup script

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

### Example timer unit

```ini
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Enable timer

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers --all
```

## 13.5 Common scheduled tasks

- Health checks.
- Backups.
- Log rotation validation.
- Certificate renewal.
- Temp file cleanup.
- Report generation.
- Security updates.
- Database vacuum or analyze jobs.

## 13.6 Cron one-liners

```bash
for u in root appuser deploy; do echo "== $u =="; sudo crontab -u "$u" -l 2>/dev/null; done
```

```bash
grep -R '' /etc/cron.d /etc/cron.daily /etc/cron.hourly 2>/dev/null
```

---
