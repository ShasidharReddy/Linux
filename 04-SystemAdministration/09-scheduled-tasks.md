# Scheduled Tasks

---

Task scheduling on Linux can be handled by cron, anacron, at, and systemd timers.

## 7.1 cron basics

Cron runs commands on a schedule.

System-wide files:
- `/etc/crontab`
- `/etc/cron.d/`
- `/etc/cron.hourly/`
- `/etc/cron.daily/`
- `/etc/cron.weekly/`
- `/etc/cron.monthly/`

Per-user crontab:

```bash
crontab -e
crontab -l
crontab -r
```

## 7.2 Cron expression format

Standard five fields:

```text
* * * * * command
- - - - -
| | | | |
| | | | +--- day of week
| | | +----- month
| | +------- day of month
| +--------- hour
+----------- minute
```

## 7.3 Cron expression cheat sheet

| Expression | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Hourly at minute 0 |
| `0 0 * * *` | Daily at midnight |
| `30 2 * * *` | Daily at 02:30 |
| `0 3 * * 0` | Weekly on Sunday at 03:00 |
| `0 4 1 * *` | Monthly on day 1 at 04:00 |
| `15 1 1 1 *` | Every January 1st at 01:15 |
| `0 9 * * 1-5` | Weekdays at 09:00 |
| `0 0 1,15 * *` | On day 1 and 15 each month |

## 7.4 Cron examples

Daily backup:

```cron
0 2 * * * /usr/local/bin/backup.sh
```

Cleanup every 10 minutes:

```cron
*/10 * * * * /usr/local/bin/cleanup-temp.sh
```

Write output to a log:

```cron
15 1 * * * /usr/local/bin/report.sh >> /var/log/report.log 2>&1
```

Best practice:
Use full paths in cron jobs.
Cron runs with a limited environment.

## 7.5 Environment in cron

Cron does not run with your interactive shell environment.
Set what you need explicitly.

Example:

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=ops@example.com

0 1 * * * /usr/local/bin/job.sh
```

## 7.6 anacron

`anacron` is useful for systems that may not be powered on continuously.
It ensures periodic jobs are eventually run.

Common on laptops and some non-24x7 systems.

Config file:

```text
/etc/anacrontab
```

Example line:

```text
1   5   cron.daily   run-parts /etc/cron.daily
```

Meaning:
- Period in days.
- Delay in minutes.
- Job identifier.
- Command.

## 7.7 at and batch

Use `at` for one-time future execution.

Examples:

```bash
echo "/usr/local/bin/restart-app.sh" | at 23:30
atq
atrm 2
```

Use `batch` to run when system load is lower.

```bash
echo "/usr/local/bin/report.sh" | batch
```

## 7.8 systemd timers as an alternative

Timers are often preferred on systemd systems.
They offer:
- Better logging.
- Better dependency management.
- Better status visibility.
- Catch-up behavior.

Example timer schedule expressions:
- `OnCalendar=daily`
- `OnCalendar=Mon *-*-* 02:00:00`
- `OnBootSec=15min`
- `OnUnitActiveSec=1h`

## 7.9 Example systemd timer setup

Service file:

```ini
[Unit]
Description=Rotate application cache

[Service]
Type=oneshot
ExecStart=/usr/local/bin/cache-rotate.sh
```

Timer file:

```ini
[Unit]
Description=Run cache rotation every hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

Commands:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now cache-rotate.timer
systemctl list-timers --all
```

## 7.10 Troubleshooting scheduled jobs

Cron troubleshooting checklist:
- Check service status for cron daemon.
- Check syntax.
- Check executable permissions.
- Check path issues.
- Check environment variables.
- Check log redirection.
- Check mail output if configured.
- Check whether the host was powered off at run time.

Helpful checks:

```bash
systemctl status cron
systemctl status crond
grep CRON /var/log/syslog
journalctl -u cron
journalctl -u crond
```

## 7.11 Scheduled task best practices

- Use absolute paths.
- Redirect output intentionally.
- Make scripts idempotent.
- Add locking to prevent overlap.
- Monitor failures.
- Prefer timers for modern systemd environments.
- Avoid hidden dependencies on shell startup files.
- Document schedules and purpose.

---

## 13.7 Scheduling commands reference

- `crontab -e`
- `crontab -l`
- `crontab -r`
- `systemctl status cron`
- `systemctl status crond`
- `journalctl -u cron`
- `journalctl -u crond`
- `atq`
- `atrm <jobid>`
- `echo "/usr/local/bin/task.sh" | at 22:00`
- `systemctl enable --now myjob.timer`
- `systemctl list-timers --all`

---

## B.7 Scheduling quick reminders
- Use absolute paths in cron.
- Cron does not source your shell profile by default.
- Redirect output intentionally.
- Add locking to avoid overlapping jobs.
- Prefer timers when you want better visibility and missed-run handling.
- Use `at` for one-time tasks instead of hacky sleep loops.
- Review root crontab entries on inherited systems.
- Keep job definitions in version control where possible.
- Make scheduled scripts idempotent.
- Capture failure exit codes in logs or monitoring.
- Test jobs manually before scheduling them.
- Check local time zone assumptions.
- Consider daylight saving impacts on clock-based schedules.
- Prefer event-driven automation when appropriate.
- Remove obsolete tasks when systems change.

---

### Scheduling inspection examples
```bash
crontab -l
ls -l /etc/cron.d
systemctl list-timers --all
atq
```

---

## B.30 More scheduling examples
```bash
flock -n /var/lock/myjob.lock /usr/local/bin/myjob.sh
systemd-run --on-calendar="tomorrow 03:00" /usr/local/bin/oneoff.sh
systemctl cat backup.timer
```
- `flock` helps prevent overlapping runs.
- `systemd-run` is useful for transient scheduled or ad hoc jobs.
- Review timers and services together because they work as a pair.
- Cron jobs should usually log somewhere intentional.
- Scheduled jobs should be idempotent and safe to retry.
- Detect long-running jobs before they stack up.
- Use random delay on distributed fleets to avoid thundering herd behavior.
- Keep time synchronization healthy across all nodes.
- Prefer clear names for timer and cron tasks.
- Remove old schedule definitions after service decommissioning.
