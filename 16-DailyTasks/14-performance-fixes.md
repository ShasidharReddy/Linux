# Performance Quick Fixes

This guide covers fast operational checks and common mitigations for CPU, memory, disk, and system slowness.

## 14.1 Identifying resource hogs

### CPU hogs

```bash
top -o %CPU
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -20
pidstat -u 1 5
```

### Memory hogs

```bash
top -o %MEM
ps -eo pid,user,%mem,%cpu,rss,cmd --sort=-%mem | head -20
smem -rtk | head -20
```

### Disk I/O hogs

```bash
iotop -oPa
iostat -xz 1 5
pidstat -d 1 5
```

## 14.2 Killing runaway processes

### Graceful terminate first

```bash
kill -TERM 12345
```

### Force kill if necessary

```bash
kill -KILL 12345
```

### Kill by user

```bash
sudo pkill -u baduser
```

### Renice busy process

```bash
sudo renice +10 -p 12345
```

## 14.3 Clearing caches carefully

### Sync before dropping page cache

```bash
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

> Use cache dropping only for testing or special troubleshooting, not as routine tuning.

## 14.4 Quick tuning

### Swappiness

```bash
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10
```

### File limits

```bash
ulimit -n
cat /proc/$(pgrep -n nginx)/limits | grep 'open files'
```

### Temporary file descriptor raise via systemd override

```ini
[Service]
LimitNOFILE=65535
```

### Sysctl for connection backlog

```bash
sudo sysctl net.core.somaxconn=4096
sudo sysctl net.ipv4.tcp_max_syn_backlog=8192
```

## 14.5 Connection limit adjustments

### Check current connections by state

```bash
ss -s
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c | sort -nr
```

### Top remote IPs on port 443

```bash
ss -ant '( sport = :443 )' | awk 'NR>1 {print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -20
```

### Adjust ephemeral port range

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
sudo sysctl net.ipv4.ip_local_port_range='10240 65000'
```

## 14.6 Quick service tuning examples

### Nginx worker file limits

```bash
nginx -T | grep -E 'worker_processes|worker_connections'
```

### PostgreSQL activity snapshot

```bash
sudo -u postgres psql -c "select pid, usename, state, wait_event_type, wait_event, query from pg_stat_activity order by state;"
```

### MySQL process list

```bash
mysql -e 'show full processlist'
```

## 14.7 Performance one-liners

```bash
vmstat 1 10
```

```bash
sar -n TCP,ETCP 1 5
```

```bash
dmesg | grep -Ei 'oom|throttl|blocked for more than|hung task'
```

---
