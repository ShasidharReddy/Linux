# Real-World Scenarios

## Scenario 1: Web server returning 502 Bad Gateway

### Symptoms

- NGINX returns 502.
- Users reach the load balancer but requests fail.
- Error logs mention upstream connection failure.

### Likely causes

- Upstream app is down.
- Wrong upstream port.
- Socket permission issue.
- App is timing out or crashing.
- SELinux blocks UNIX socket access.

### Step-by-step

1. Confirm the frontend status code.
2. Check local web server health.
3. Check upstream process status.
4. Check upstream port or socket.
5. Review error logs on both tiers.
6. Test backend locally with `curl`.
7. Check SELinux or permission issues.
8. Check resource pressure and OOM.
9. Fix root cause.
10. Retest end to end.

### Commands

```bash
systemctl status nginx --no-pager
journalctl -u nginx -b --no-pager | tail -100
curl -vk http://127.0.0.1:8080/health
ss -ltnp | grep 8080
journalctl -u myapp -b --no-pager | tail -100
ls -lZ /run/myapp.sock
```

### Resolution pattern

- If upstream is down, restore the app.
- If socket permissions are wrong, fix ownership or SELinux context.
- If upstream is overloaded, scale or tune timeouts.

---

## Scenario 2: SSH connection refused

### Symptoms

- `ssh: connect to host ... port 22: Connection refused`

### Likely causes

- `sshd` is not running.
- SSH listens on a different port.
- Firewall actively rejects traffic.

### Step-by-step

1. Confirm host reachability.
2. Check if TCP 22 is listening.
3. Check `sshd` service status.
4. Review `sshd` configuration.
5. Confirm firewall and security group rules.
6. Test locally from the host itself.

### Commands

```bash
ping -c 3 host
ss -ltnp | grep sshd
systemctl status sshd --no-pager
sshd -t
journalctl -u sshd -b --no-pager | tail -100
firewall-cmd --list-all
```

### Resolution pattern

- Start or fix `sshd`.
- Correct the configured port.
- Open the firewall path.

---

## Scenario 3: Disk 100% full

### Symptoms

- Writes fail.
- Services crash or become read-only.
- Package updates fail.

### Likely causes

- Log growth.
- Backup accumulation.
- Open deleted file.
- Container storage growth.

### Step-by-step

1. Check `df -h` and `df -i`.
2. Identify the full filesystem.
3. Use `du` to find largest directories.
4. Compare `du` and `df`.
5. Check `lsof +L1`.
6. Clear safe caches or rotate logs.
7. Restore service.
8. Prevent recurrence.

### Commands

```bash
df -hT
df -i
du -xhd1 /var | sort -h
lsof +L1
journalctl --disk-usage
```

### Resolution pattern

- Recover enough space safely.
- Restart process holding deleted files if needed.
- Add log retention or monitoring.

---

## Scenario 4: Cannot resolve DNS

### Symptoms

- `ping` to IP works.
- `ping hostname` fails.
- Applications time out on hostnames.

### Likely causes

- Bad `/etc/resolv.conf`.
- DNS server unreachable.
- Firewall blocking UDP or TCP 53.
- Search domain confusion.

### Step-by-step

1. Test by IP and hostname.
2. Inspect resolver state.
3. Query nameservers directly.
4. Check routing to DNS server.
5. Check firewall and local cache.
6. Validate search domains.

### Commands

```bash
getent hosts example.com
cat /etc/resolv.conf
resolvectl status
resolvectl query example.com
dig @8.8.8.8 example.com
ip route get 8.8.8.8
```

### Resolution pattern

- Correct nameserver entries.
- Restore network reachability.
- Fix resolver service.

---

## Scenario 5: Cron job not running

### Symptoms

- Scheduled task did not execute.
- No expected output files or emails.

### Likely causes

- Crond not running.
- Wrong crontab syntax.
- PATH issue.
- Script not executable.
- Wrong user context.
- Timezone confusion.

### Step-by-step

1. Verify cron service status.
2. Check system and user crontabs.
3. Check cron logs.
4. Run the job manually as the target user.
5. Use absolute paths in commands.
6. Confirm environment assumptions.

### Commands

```bash
systemctl status cron --no-pager || systemctl status crond --no-pager
crontab -l
ls -l /etc/cron.*
journalctl -u cron -b --no-pager | tail -100
sudo -u targetuser /bin/bash -lc '/path/to/script'
```

### Resolution pattern

- Fix syntax or paths.
- Ensure cron daemon is running.
- Redirect output for debugging.

---

## Scenario 6: Docker container keeps restarting

### Symptoms

- Container in restart loop.
- Service unavailable intermittently.

### Likely causes

- Process exits immediately.
- Health check fails.
- OOM kill.
- Missing environment variable.
- Dependency unavailable.

### Step-by-step

1. Inspect restart policy.
2. Read container logs.
3. Inspect exit code.
4. Check memory limits and OOM messages.
5. Run image interactively if safe.
6. Confirm mounts and secrets exist.

### Commands

```bash
docker ps -a
docker logs --tail 200 container
docker inspect container --format '{{.State.ExitCode}} {{.State.OOMKilled}} {{.HostConfig.RestartPolicy.Name}}'
docker stats --no-stream
journalctl -k --no-pager | grep -i oom
```

### Resolution pattern

- Fix app startup failure.
- Adjust health checks or resources.
- Provide missing configuration.

---

## Scenario 7: High load average with low CPU

### Symptoms

- `uptime` shows high load.
- CPU usage is not very high.
- System feels stuck.

### Likely causes

- Disk I/O stalls.
- NFS hang.
- Blocked tasks in D state.
- Swap thrashing.

### Step-by-step

1. Check `vmstat`.
2. Check for D-state tasks.
3. Check disk latency with `iostat`.
4. Check kernel logs.
5. Check remote mounts.

### Commands

```bash
uptime
vmstat 1 5
iostat -xz 1 5
ps -eo pid,stat,wchan:32,comm | awk '$2 ~ /D/'
mount | egrep 'nfs|cifs'
journalctl -k --no-pager | tail -100
```

### Resolution pattern

- Restore storage path.
- Unblock hung mount.
- Fix I/O bottleneck.

---

## Scenario 8: Service fails with permission denied

### Symptoms

- Unit exits immediately.
- Logs show `permission denied`.

### Likely causes

- File mode issue.
- Parent directory execute bit missing.
- SELinux denial.
- Capability missing.

### Step-by-step

1. Identify exact path.
2. Check with `namei -l`.
3. Check service user and groups.
4. Check ACLs.
5. Check SELinux or AppArmor.
6. Fix least privilege path.

### Commands

```bash
systemctl status myservice --no-pager
journalctl -u myservice -b --no-pager | tail -100
namei -l /path/to/file
id serviceuser
getfacl /path/to/file
ls -Z /path/to/file
ausearch -m AVC -ts recent 2>/dev/null
```

### Resolution pattern

- Correct ownership or mode.
- Restore proper SELinux context.
- Add required capability if appropriate.

---

## Scenario 9: Application cannot bind to port 80

### Symptoms

- App starts fine on 8080 but not 80.

### Likely causes

- Port already in use.
- Missing privileges or capability.
- SELinux policy.

### Step-by-step

1. Check whether port 80 is occupied.
2. Check whether the app runs as non-root.
3. Use file capability if desired.
4. Check firewall and SELinux.

### Commands

```bash
ss -ltnp '( sport = :80 )'
getcap /usr/local/bin/myapp
setcap 'cap_net_bind_service=+ep' /usr/local/bin/myapp
journalctl -u myapp -b --no-pager | tail -100
```

### Resolution pattern

- Free the port.
- Grant `cap_net_bind_service`.
- Update policy if SELinux blocks.

---

## Scenario 10: System boots to emergency mode after fstab change

### Symptoms

- Boot stops in emergency mode.
- Logs mention mount failure.

### Likely causes

- Bad UUID.
- Missing mount point.
- Network mount without `_netdev`.
- Invalid mount options.

### Step-by-step

1. Enter rescue shell.
2. Review `journalctl -xb`.
3. Compare `fstab` with `blkid`.
4. Comment out suspect entry.
5. Test with `mount -a`.
6. Reboot after validation.

### Commands

```bash
journalctl -xb --no-pager
blkid
cat /etc/fstab
mount -a
```

### Resolution pattern

- Correct `fstab`.
- Add `nofail` or `_netdev` where appropriate.

---

## Scenario 11: OOM killer terminates database process

### Symptoms

- Database restarts.
- Kernel log shows `Killed process`.

### Likely causes

- Mis-sized memory settings.
- Host overcommitted.
- Another workload consumed memory.
- Cgroup memory cap too low.

### Step-by-step

1. Confirm OOM in kernel logs.
2. Identify victim and memory footprint.
3. Review database memory config.
4. Check co-located workloads.
5. Reduce pressure or increase limits.
6. Add monitoring for memory headroom.

### Commands

```bash
journalctl -k --no-pager | grep -i 'killed process\|out of memory'
free -h
ps -eo pid,user,comm,rss,%mem --sort=-rss | head -20
systemctl show dbservice -p MemoryMax
```

### Resolution pattern

- Tune DB buffers.
- Isolate noisy neighbors.
- Adjust memory limits.

---

## Scenario 12: Filesystem remounts read-only

### Symptoms

- Apps fail with write errors.
- Root filesystem becomes read-only.

### Likely causes

- Filesystem corruption.
- Underlying I/O errors.
- Storage path instability.

### Step-by-step

1. Confirm mount state.
2. Review kernel logs.
3. Determine if hardware is failing.
4. Schedule offline repair.
5. Boot rescue environment if needed.

### Commands

```bash
mount | grep ' / '
journalctl -k --no-pager | tail -200
smartctl -a /dev/sdX
```

### Resolution pattern

- Repair filesystem offline.
- Replace failing storage if necessary.

---

## Scenario 13: Package update fails with GPG key error

### Symptoms

- `apt update` or `dnf makecache` fails.

### Likely causes

- Missing repository key.
- Wrong key.
- Expired metadata.
- Incorrect system time.

### Step-by-step

1. Verify system clock.
2. Confirm repository URL.
3. Install correct key from trusted source.
4. Refresh metadata.
5. Retry update.

### Commands

```bash
date
chronyc tracking 2>/dev/null || timedatectl
apt update
rpm -qa gpg-pubkey
dnf clean all
```

### Resolution pattern

- Correct time.
- Install trusted key.
- Fix repository configuration.

---

## Scenario 14: Kernel upgrade leaves system unbootable

### Symptoms

- New kernel fails.
- Old kernel may still work.

### Likely causes

- Broken initramfs.
- Driver mismatch.
- `/boot` space issue.
- Bad kernel parameters.

### Step-by-step

1. Boot previous kernel from GRUB.
2. Verify current system state.
3. Rebuild initramfs for failing kernel.
4. Check `/boot` free space.
5. Review last kernel install logs.

### Commands

```bash
df -h /boot
ls -lh /boot
journalctl -b -1 --no-pager
update-initramfs -u -k all || dracut -f --regenerate-all
```

### Resolution pattern

- Keep known-good kernel.
- Rebuild initramfs.
- Remove broken parameters.

---

## Scenario 15: NFS mount hangs processes

### Symptoms

- Commands touching mount path hang.
- Load average rises.
- Tasks enter D state.

### Likely causes

- NFS server unreachable.
- Network path issue.
- Hard-mounted share not responding.

### Step-by-step

1. Identify affected mount.
2. Check server reachability.
3. Check mount options.
4. Review network and NFS client logs.
5. Consider lazy unmount or recovery based on impact.

### Commands

```bash
mount | grep nfs
showmount -e nfs-server
ping -c 3 nfs-server
journalctl -k --no-pager | grep -i nfs
ps -eo pid,stat,wchan:32,comm | awk '$2 ~ /D/'
```

### Resolution pattern

- Restore path to server.
- Revisit mount options.
- Prevent boot hangs with `_netdev` and appropriate automounting.

---

## Scenario 16: DNS works locally but app still cannot connect

### Symptoms

- `dig` succeeds.
- App times out on outbound HTTPS.

### Likely causes

- Proxy misconfiguration.
- Firewall blocks egress.
- MTU or TLS issue.
- IPv6 preference problem.

### Step-by-step

1. Test with `curl -v`.
2. Check proxy environment variables.
3. Test by IPv4 explicitly.
4. Inspect firewall rules.
5. Check path MTU.
6. Inspect TLS handshake.

### Commands

```bash
env | grep -i proxy
curl -4vk https://api.example.com/
openssl s_client -connect api.example.com:443 -servername api.example.com
tracepath api.example.com
nft list ruleset
```

### Resolution pattern

- Fix proxy settings.
- Open egress path.
- Adjust MTU or TLS trust.

---

## Scenario 17: High system CPU caused by interrupt storm

### Symptoms

- `%sy` high.
- One CPU core heavily loaded.
- Network performance unstable.

### Likely causes

- NIC interrupt storm.
- Driver issue.
- RX/TX imbalance.

### Step-by-step

1. Check `mpstat` per CPU.
2. Watch `/proc/interrupts`.
3. Check NIC counters.
4. Review driver and firmware logs.
5. Rebalance IRQs or investigate hardware.

### Commands

```bash
mpstat -P ALL 1 5
watch -n1 'cat /proc/interrupts'
ethtool -S eth0 | head -100
journalctl -k --no-pager | grep -i -E 'eth0|ixgbe|mlx|irq|interrupt'
```

### Resolution pattern

- Update driver or firmware.
- Correct affinity or RSS settings.
- Replace failing hardware.

---

## Scenario 18: Login fails after password reset on SELinux system

### Symptoms

- Password was changed.
- Login still fails.

### Likely causes

- SELinux contexts on auth files are wrong.
- Root reset done from rescue without relabel.

### Step-by-step

1. Review auth logs.
2. Check SELinux mode.
3. Restore contexts or trigger relabel.
4. Reboot if full relabel is required.

### Commands

```bash
journalctl -b --no-pager | grep -i -E 'selinux|pam|authentication'
ls -Z /etc/shadow /etc/passwd
touch /.autorelabel
```

### Resolution pattern

- Restore proper labels.
- Reboot and test login.

---

## Scenario 19: RAID array degraded after disk replacement

### Symptoms

- RAID marked degraded.
- Replacement disk visible.
- Rebuild not started.

### Likely causes

- Partition table not copied.
- Wrong member size.
- Disk not added back to array.

### Step-by-step

1. Inspect array detail.
2. Compare old and new partition layout.
3. Partition replacement disk correctly.
4. Add device back to array.
5. Monitor rebuild.

### Commands

```bash
cat /proc/mdstat
mdadm --detail /dev/md0
lsblk -f
mdadm /dev/md0 --add /dev/sdX1
watch -n5 cat /proc/mdstat
```

### Resolution pattern

- Recreate matching partition layout.
- Add member and monitor rebuild.

---

## Scenario 20: Server reachable but website very slow

### Symptoms

- Ping is fine.
- TCP connect works.
- Page generation is slow.

### Likely causes

- Slow database.
- Disk latency.
- App thread pool exhaustion.
- DNS or upstream timeout.
- Swap thrashing.

### Step-by-step

1. Measure frontend and backend latency separately.
2. Check app logs for slow requests.
3. Check DB response time.
4. Check CPU, memory, disk, and network metrics.
5. Identify bottleneck with USE method.
6. Apply mitigation and verify improvement.

### Commands

```bash
curl -w 'connect=%{time_connect} starttransfer=%{time_starttransfer} total=%{time_total}\n' -o /dev/null -s https://site
pidstat -dur 1 5
iostat -xz 1 5
vmstat 1 5
journalctl -u app -b --no-pager | tail -100
```

### Resolution pattern

- Fix the actual bottleneck.
- Do not assume high load means CPU.

---
