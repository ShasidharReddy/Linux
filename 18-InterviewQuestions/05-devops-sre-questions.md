# DevOps and SRE Linux Interview Questions

This guide collects Linux interview questions tailored to DevOps, CI/CD, containers, Kubernetes, and SRE work.

## Q181: Why is Linux foundational in DevOps and SRE roles?
**Answer:** Most cloud infrastructure, containers, CI runners, build agents, Kubernetes nodes, monitoring stacks, and production services run on Linux. DevOps and SRE engineers need Linux fluency because deployments, observability, security, automation, and incident response all depend on it.

Core reasons:
- Dominant server OS
- Strong automation interfaces
- Rich networking and process controls
- Native container ecosystem
- Flexible observability tooling

Example commands:
```bash
uname -a
cat /etc/os-release
systemctl list-units --type=service | head
```

---

## Q182: How does Linux knowledge help in CI/CD pipelines?
**Answer:** CI/CD pipelines often execute on Linux-based workers where jobs perform source checkout, package installation, testing, artifact creation, container builds, and deployments.

Important Linux skills for CI/CD:
- Shell scripting
- Package management
- File permissions
- Environment variable handling
- Service startup/troubleshooting
- Network debugging for artifact registries and APIs

Example commands:
```bash
env | sort
chmod +x build.sh
bash -x build.sh
curl -I https://registry.example.com
```

---

## Q183: What Linux troubleshooting is important for container platforms?
**Answer:** Container platforms still depend on Linux fundamentals. When pods or containers fail, the root cause may involve cgroups, namespaces, overlay storage, DNS, iptables/nftables, systemd, disk pressure, or process/resource limits.

Example commands:
```bash
lsns
systemd-cgls
df -h
ss -tulnp
journalctl -u containerd -n 100 || journalctl -u docker -n 100
```

---

## Q184: What are the Linux-level causes of Kubernetes pod restarts?
**Answer:** Pod restarts may be triggered by application issues, but Linux-level contributors include OOM kills, node pressure, file system issues, DNS failures, permission problems, and container runtime errors.

What to check:
- Node memory pressure and OOM logs
- Kubelet/runtime logs
- Disk usage and inode usage
- Cgroup limits

Example commands:
```bash
journalctl -u kubelet -n 100 || true
journalctl -u containerd -n 100 || true
journalctl -k | grep -i oom
free -h
df -h
df -i
```

---

## Q185: Why do SREs care about logs, metrics, and traces on Linux hosts?
**Answer:** Linux hosts are the substrate where signals originate. Application behavior is only part of the picture; host-level metrics and logs explain CPU saturation, memory pressure, network errors, file system saturation, and service crashes.

Three pillars in host context:
- Logs explain events and errors
- Metrics show resource trends
- Traces help understand request latency across components

Example commands:
```bash
journalctl -p err -n 50
top -b -n 1 | head
ss -s
```

---

## Q186: How do Linux file permissions affect CI/CD deployments?
**Answer:** Incorrect ownership or execute bits can break artifact extraction, startup scripts, SSH-based deployments, and secret handling.

Common issues:
- Script missing execute permission
- Wrong owner on deployment directory
- Private key too permissive
- Shared directories not writable by service account

Example commands:
```bash
ls -l deploy.sh
chmod 755 deploy.sh
chmod 600 ~/.ssh/id_ed25519
chown -R appuser:appgroup /opt/myapp
```

---

## Q187: How does Linux support observability agents?
**Answer:** Monitoring and logging agents on Linux collect process, CPU, memory, disk, journal, network, and service data. They often integrate with systemd, `/proc`, cgroups, kernel counters, and sometimes eBPF.

Operational considerations:
- Agent resource usage
- Access permissions
- Journal and log file access
- Network egress to observability backends

Example commands:
```bash
systemctl status node_exporter || true
systemctl status fluent-bit || true
journalctl -u node_exporter -n 50 || true
ss -tulnp
```

---

## Q188: What Linux checks help when a CI runner is slow?
**Answer:** Slow runners may suffer from CPU contention, memory pressure, saturated disks, registry/network slowness, or workspace cleanup issues.

Step-by-step:
1. Check CPU and memory
2. Check disk usage and I/O latency
3. Check network path to source/artifact registries
4. Review runner service logs

Example commands:
```bash
uptime
free -h
iostat -xz 1 3
df -h
curl -I https://github.com
journalctl -u gitlab-runner -n 100 || journalctl -u actions.runner.* -n 100 || true
```

---

## Q189: How do Linux concepts relate to Infrastructure as Code tools like Ansible?
**Answer:** Ansible automates Linux tasks by codifying operations such as package installs, file edits, service management, users/groups, firewall changes, and templated configuration.

Linux knowledge matters because you must understand what the automation is changing.

Example commands:
```bash
ansible localhost -m setup || true
ansible-playbook site.yml --check || true
systemctl status nginx
```

---

## Q190: What Linux topics matter for Docker image optimization?
**Answer:** Efficient Linux-based images depend on package selection, file system layering, permissions, process model, signal handling, and minimal runtime dependencies.

Best practices:
- Use smaller base images when appropriate
- Remove package caches
- Run as non-root where possible
- Use exec form entrypoints
- Keep layers clean and purposeful

Example commands:
```bash
docker history image:tag || true
docker inspect image:tag || true
```

---

## Q191: How do Linux signals matter in containers and orchestration?
**Answer:** Proper signal handling is critical for graceful shutdown during rolling deployments, autoscaling, and restarts. PID 1 behavior inside containers matters because signal forwarding and zombie reaping can differ from normal processes.

Important signals:
- `SIGTERM` for graceful stop
- `SIGKILL` if timeout exceeded
- `SIGHUP` for reload in some apps

Example commands:
```bash
ps -p 1 -o pid,comm,args
kill -TERM 1234
docker stop container_id || true
```

---

## Q192: What Linux checks help when Kubernetes DNS appears broken?
**Answer:** At the Linux level, verify node connectivity, resolver behavior, firewall policy, and DNS pod/service reachability.

Example commands:
```bash
cat /etc/resolv.conf
ss -lunp | grep ':53 '
iptables -L -n -v || nft list ruleset
ping -c 2 kube-dns.kube-system.svc.cluster.local || true
```

Also inspect CoreDNS and node-local DNS components from the platform side.

---

## Q193: How does Linux support blue-green or rolling deployments?
**Answer:** Linux hosts support deployment patterns through process isolation, service managers, reverse proxies, load balancers, health checks, and symlink-based releases.

Common implementation details:
- New app version starts on a new port/path
- Health checks validate readiness
- Proxy switches traffic gradually or atomically
- Old version drains and stops gracefully

Example commands:
```bash
systemctl status myapp-blue myapp-green || true
curl -f http://127.0.0.1:8081/health || true
nginx -t
ln -sfn /opt/releases/2025-01-10 /opt/myapp/current
```

---

## Q194: What host-level Linux security controls matter in cloud environments?
**Answer:** Even in cloud-native environments, host security remains critical.

Key controls:
- Patch management
- Minimal packages
- SSH hardening or agent-based access
- Firewalling
- SELinux/AppArmor
- Audit logs
- Non-root workloads
- Secure secret injection

Example commands:
```bash
ss -tulnp
sudo firewall-cmd --list-all || true
getenforce || true
sudo -l
```

---

## Q195: How do you debug a failing systemd service in an automated deployment?
**Answer:** Deployment automation often fails at the final service startup step. Linux-side debugging should validate the unit file, environment file, permissions, config syntax, working directory, and dependencies.

Example commands:
```bash
systemctl status myapp
journalctl -u myapp -n 100 --no-pager
systemctl cat myapp
systemctl show myapp | grep -E 'ExecStart|WorkingDirectory|EnvironmentFile'
```

---

## Q196: What Linux skills help with incident response in SRE?
**Answer:** Strong SRE response depends on fast host-level triage.

Critical skills:
- Read logs with `journalctl`
- Inspect processes with `ps`, `top`, `ss`, `lsof`
- Check resources with `df`, `free`, `vmstat`, `iostat`
- Validate networking with `ip`, `dig`, `curl`, `tcpdump`
- Read service state with `systemctl`

Example commands:
```bash
journalctl -p warning -n 100
ps -eo pid,%cpu,%mem,cmd --sort=-%cpu | head
ss -tulnp
free -h
df -h
```

---

## Q197: What Linux host issues commonly impact Terraform or provisioning workflows?
**Answer:** Provisioning steps can fail because of package repository issues, DNS failures, missing permissions, cloud-init errors, time drift, blocked outbound access, or incorrect SSH setup.

Example commands:
```bash
cloud-init status || true
journalctl -u cloud-init -n 100 || true
curl -I https://registry.terraform.io || true
ssh -vvv user@host || true
```

---

## Q198: How do Linux concepts relate to Kubernetes node reliability?
**Answer:** Kubernetes nodes are Linux systems. Node reliability depends on stable kernel behavior, container runtime health, storage space, cgroup correctness, time sync, DNS, and resource isolation.

Example commands:
```bash
systemctl status kubelet
systemctl status containerd || systemctl status docker
free -h
df -h
journalctl -u kubelet -n 100
```

---

## Q199: What Linux-level checks help with monitoring alert fatigue?
**Answer:** Host-level context helps distinguish real incidents from noisy symptoms.

Approach:
1. Correlate alerts with system state
2. Identify repeated false positives due to transient spikes
3. Improve thresholds and saturation indicators
4. Add dependency-aware alerting

Example commands:
```bash
uptime
free -h
df -h
ss -s
journalctl --since '1 hour ago' -p warning
```

---

## Q200: What should a DevOps/SRE engineer know deeply about Linux?
**Answer:** DevOps/SRE roles require more than command memorization. You should deeply understand how Linux handles processes, systemd, networking, filesystems, permissions, resource limits, containers, observability, and incident response.

A strong engineer can:
- Debug under pressure
- Automate safely
- Explain trade-offs clearly
- Correlate host-level signals with service health
- Prevent recurrence with better engineering controls

Example commands:
```bash
systemctl list-units --type=service | head
journalctl -b -p err
ip route
ss -s
df -h
free -h
```

---
