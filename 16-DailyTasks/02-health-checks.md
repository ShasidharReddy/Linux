# System Health Checks

This guide covers repeatable daily health verification, monitoring checks, and quick operational diagnostics.

## 2.1 Daily health check objectives

- Detect CPU saturation.
- Detect memory pressure.
- Detect disk exhaustion.
- Detect service failures.
- Detect unusual reboots.
- Detect filesystem errors.
- Detect network issues.
- Detect security anomalies.

## 2.2 Quick dashboard commands

### Overall snapshot

```bash
date; hostname; uptime; who -b
```

### Load and CPU

```bash
uptime
cat /proc/loadavg
mpstat -P ALL 1 3
sar -u 1 3
```

### Memory

```bash
free -h
vmstat 1 5
sar -r 1 3
cat /proc/meminfo | egrep 'MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree'
```

### Disk

```bash
df -hT
lsblk -f
findmnt -lo TARGET,SOURCE,FSTYPE,OPTIONS
```

### Top processes

```bash
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head -20
ps -eo pid,ppid,user,%mem,%cpu,cmd --sort=-%mem | head -20
```

### Failed services

```bash
systemctl --failed
```

### Recent critical logs

```bash
journalctl -p 3 -xb --no-pager | tail -100
```

### Reboots and uptime history

```bash
last reboot | head
who -b
uptime -p
```

## 2.3 Daily health check script

```bash
#!/usr/bin/env bash
set -euo pipefail

HOST=$(hostname -f 2>/dev/null || hostname)
NOW=$(date '+%F %T %Z')

section() {
  echo
  echo "===== $1 ====="
}

echo "Health Check Report"
echo "Host: $HOST"
echo "Generated: $NOW"

section "Uptime"
uptime
who -b || true
last reboot | head -5 || true

section "CPU and Load"
cat /proc/loadavg
command -v mpstat >/dev/null && mpstat -P ALL 1 1 || true
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -10

section "Memory"
free -h
vmstat 1 3
ps -eo pid,user,%mem,%cpu,cmd --sort=-%mem | head -10

section "Disk"
df -hT
for mount in / /var /home; do
  [ -d "$mount" ] && du -xhd1 "$mount" 2>/dev/null | sort -h | tail -20
done

section "Inodes"
df -ih

section "Services"
systemctl --failed || true
systemctl list-units --type=service --state=running | head -40 || true

section "Network"
ip -br a
ip route
ss -tulpn | head -100

section "Logs"
journalctl -p warning -n 50 --no-pager || true
journalctl -p err -n 50 --no-pager || true

section "Filesystem errors"
dmesg --level=err,warn | tail -50 || true

section "Security"
last -a | head -20 || true
lastb -a | head -20 || true

section "Summary thresholds"
ROOT_USE=$(df --output=pcent / | tail -1 | tr -dc '0-9')
MEM_AVAIL=$(awk '/MemAvailable/ {print int($2/1024)}' /proc/meminfo)
echo "Root filesystem used: ${ROOT_USE}%"
echo "Available memory: ${MEM_AVAIL} MB"
if [ "$ROOT_USE" -ge 90 ]; then
  echo "ALERT: Root filesystem above 90%"
fi
if [ "$MEM_AVAIL" -le 512 ]; then
  echo "ALERT: Available memory below 512 MB"
fi
```

## 2.4 Health check with email alert example

```bash
#!/usr/bin/env bash
set -euo pipefail

REPORT=$(/usr/local/bin/daily-health-check.sh)
ALERTS=$(printf '%s
' "$REPORT" | grep '^ALERT:' || true)

if [ -n "$ALERTS" ]; then
  printf '%s

%s
' "Daily health check alerts" "$REPORT" | mail -s "[ALERT] $(hostname) daily health" ops@example.com
fi
```

## 2.5 Automated health monitoring pointers

- Prometheus Node Exporter for metrics.
- Alertmanager for notifications.
- Grafana for dashboards.
- Monit for simple process checks.
- systemd watchdog for service supervision.
- Ping checks for availability.
- Blackbox Exporter for HTTP, TCP, and DNS.

## 2.6 Uptime and availability tracking

### Local uptime tracking

```bash
uptime -s
systemd-analyze
who -b
```

### External availability check examples

```bash
curl -fsS -o /dev/null -w '%{http_code} %{time_total}
' https://example.com/healthz
```

```bash
for i in {1..5}; do date; curl -fsS -o /dev/null -w '%{http_code} %{time_total}
' https://example.com/healthz; sleep 5; done
```

### Ping loss quick test

```bash
ping -c 10 8.8.8.8
```

## 2.7 System status dashboard commands

```bash
watch -n 2 'date; echo; uptime; echo; free -h; echo; df -h / /var; echo; systemctl --failed; echo; ss -s'
```

```bash
watch -n 2 'ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -15'
```

```bash
watch -n 5 'journalctl -p err -n 20 --no-pager'
```

## 2.8 Health check interpretation tips

- High load with low CPU usage often means I/O wait or blocked tasks.
- High memory usage is not always bad if cache is reclaimable.
- Disk alerts require checking both space and inodes.
- Repeated service restarts indicate crash loops or bad dependencies.
- Kernel warnings in `dmesg` may point to hardware or driver issues.
- Many `TIME_WAIT` sockets may be normal for busy web servers.

## 2.9 Real-world health check one-liners

```bash
journalctl --since today -p warning --no-pager | tail -100
```

```bash
ps -eo state,pid,ppid,cmd | awk '$1 ~ /D/ {print}'
```

```bash
df -hP | awk 'NR==1 || int($5+0) > 80'
```

```bash
for s in sshd nginx docker; do systemctl is-active --quiet "$s" || echo "$s is not active"; done
```

---
