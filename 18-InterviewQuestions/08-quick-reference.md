# Linux Interview Quick Reference

This guide collects file locations, command packs, ports, signals, exit codes, and final review checklists.

## Important File Locations

| Path | Purpose |
|---|---|
| `/etc/passwd` | User account definitions |
| `/etc/shadow` | Password hashes and aging information |
| `/etc/group` | Group definitions |
| `/etc/gshadow` | Group security information |
| `/etc/sudoers` | Sudo policy |
| `/etc/ssh/sshd_config` | OpenSSH server configuration |
| `/etc/hosts` | Static hostname mappings |
| `/etc/resolv.conf` | DNS resolver configuration |
| `/etc/fstab` | Static mount definitions |
| `/etc/crontab` | System-wide cron entries |
| `/etc/sysctl.conf` | Kernel parameter configuration |
| `/etc/sysctl.d/` | Drop-in kernel tuning files |
| `/etc/pam.d/` | PAM service configuration |
| `/etc/systemd/system/` | Local systemd unit files |
| `/usr/lib/systemd/system/` | Packaged systemd unit files |
| `/var/log/` | Traditional log directory |
| `/var/spool/cron/` | User cron spools on some systems |
| `/var/lib/` | Persistent application and service state |
| `/boot/` | Kernel, initramfs, bootloader files |
| `/proc/` | Virtual kernel and process info |
| `/sys/` | Virtual device and kernel interfaces |
| `/dev/` | Device nodes |
| `/home/` | User home directories |
| `/root/` | Root user home |
| `/tmp/` | Temporary files |
| `/run/` | Runtime state files and sockets |

---

## Essential Commands Cheat Sheet

| Area | Commands |
|---|---|
| Navigation | `pwd`, `cd`, `ls`, `tree` |
| Files | `touch`, `cp`, `mv`, `rm`, `mkdir`, `find` |
| Viewing | `cat`, `less`, `head`, `tail`, `tail -f` |
| Text processing | `grep`, `awk`, `sed`, `sort`, `cut`, `uniq`, `wc` |
| Permissions | `chmod`, `chown`, `chgrp`, `umask`, `getfacl`, `setfacl` |
| Users | `id`, `who`, `w`, `useradd`, `usermod`, `passwd` |
| Processes | `ps`, `top`, `htop`, `pgrep`, `kill`, `nice`, `renice` |
| Services | `systemctl`, `journalctl`, `loginctl` |
| Networking | `ip`, `ss`, `ping`, `dig`, `curl`, `nc`, `traceroute`, `tcpdump` |
| Storage | `df`, `du`, `lsblk`, `blkid`, `mount`, `umount`, `findmnt` |
| LVM | `pvs`, `vgs`, `lvs`, `lvextend`, `lvreduce` |
| Packages | `apt`, `dpkg`, `dnf`, `yum`, `rpm`, `zypper` |
| Scheduling | `crontab`, `at`, `systemctl list-timers` |
| Performance | `vmstat`, `iostat`, `sar`, `strace`, `perf` |
| Security | `sudo`, `getenforce`, `sestatus`, `firewall-cmd`, `nft`, `iptables` |
| Archives | `tar`, `gzip`, `gunzip`, `zip`, `unzip` |

---

## Common Port Numbers

| Port | Protocol | Typical Use |
|---|---|---|
| 20/21 | TCP | FTP |
| 22 | TCP | SSH/SCP/SFTP |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP |
| 69 | UDP | TFTP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 123 | UDP | NTP |
| 143 | TCP | IMAP |
| 161/162 | UDP | SNMP |
| 389 | TCP/UDP | LDAP |
| 443 | TCP | HTTPS |
| 465 | TCP | SMTPS |
| 514 | UDP | Syslog |
| 587 | TCP | SMTP submission |
| 636 | TCP | LDAPS |
| 993 | TCP | IMAPS |
| 995 | TCP | POP3S |
| 1433 | TCP | Microsoft SQL Server |
| 1521 | TCP | Oracle DB |
| 2049 | TCP/UDP | NFS |
| 2379 | TCP | etcd client |
| 2380 | TCP | etcd peer |
| 3000 | TCP | Common web app port |
| 3306 | TCP | MySQL/MariaDB |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 5601 | TCP | Kibana |
| 5672 | TCP | RabbitMQ |
| 6379 | TCP | Redis |
| 6443 | TCP | Kubernetes API server |
| 8080 | TCP | Alternative HTTP/app port |
| 8443 | TCP | Alternative HTTPS/app port |
| 9090 | TCP | Prometheus |
| 9200 | TCP | Elasticsearch/OpenSearch |
| 9300 | TCP | Elasticsearch transport |
| 10250 | TCP | Kubelet |

---

## Common Signal Numbers

| Signal | Number | Meaning |
|---|---:|---|
| SIGHUP | 1 | Hangup or reload request |
| SIGINT | 2 | Interrupt from keyboard |
| SIGQUIT | 3 | Quit |
| SIGILL | 4 | Illegal instruction |
| SIGTRAP | 5 | Trace/breakpoint trap |
| SIGABRT | 6 | Abort |
| SIGBUS | 7 | Bus error |
| SIGFPE | 8 | Floating-point exception |
| SIGKILL | 9 | Force kill, cannot be caught |
| SIGUSR1 | 10 | User-defined signal 1 |
| SIGSEGV | 11 | Segmentation fault |
| SIGUSR2 | 12 | User-defined signal 2 |
| SIGPIPE | 13 | Broken pipe |
| SIGALRM | 14 | Alarm clock |
| SIGTERM | 15 | Graceful termination request |
| SIGCHLD | 17 | Child stopped or exited |
| SIGCONT | 18 | Continue if stopped |
| SIGSTOP | 19 | Stop, cannot be caught |
| SIGTSTP | 20 | Terminal stop |
| SIGTTIN | 21 | Background read from TTY |
| SIGTTOU | 22 | Background write to TTY |

Note: Signal numbering can vary slightly across Unix variants, but the above is standard enough for Linux interview reference.

---

## Common Exit Codes

| Exit Code | Meaning |
|---|---|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of shell builtin or incorrect usage |
| 126 | Command found but not executable |
| 127 | Command not found |
| 128 | Invalid exit argument |
| 128+n | Fatal error due to signal `n` |
| 130 | Script terminated by `Ctrl+C` / SIGINT |
| 137 | Killed by SIGKILL, often OOM or forced kill |
| 139 | Segmentation fault / SIGSEGV |
| 143 | Terminated by SIGTERM |
| 255 | Exit status out of range or general failure in some tools |

Example:
```bash
ls /does-not-exist
echo $?
```

---

## Troubleshooting Command Packs

### Disk Investigation Pack
```bash
df -h
df -i
du -xhd1 /var | sort -h
find /var -xdev -type f -size +500M -ls
lsof | grep deleted
```

### CPU and Memory Investigation Pack
```bash
uptime
top -b -n 1 | head -20
free -h
vmstat 1 5
ps -eo pid,%cpu,%mem,cmd --sort=-%cpu | head
ps -eo pid,%mem,rss,cmd --sort=-%mem | head
```

### Network Investigation Pack
```bash
ip addr
ip route
ss -tulnp
ss -s
ping -c 4 8.8.8.8
dig example.com
curl -Iv https://example.com
```

### Service Failure Investigation Pack
```bash
systemctl status myservice
journalctl -u myservice -n 100 --no-pager
systemctl cat myservice
systemctl show myservice | grep -E 'ExecStart|Environment|After|Requires'
```

---

## Linux Interview Preparation Tips

1. Practice in a VM or cloud instance.
2. Learn the reasoning behind commands, not just syntax.
3. Be able to explain trade-offs.
4. When answering scenario questions, use a structured troubleshooting methodology.
5. Mention logs, metrics, and recent changes.
6. Be explicit about safety in production.
7. Know the difference between a quick fix and root cause.
8. Explain how to verify success after every action.

A strong interview answer usually includes:
- Problem framing
- Likely causes
- Validation steps
- Safe remediation
- Follow-up prevention measures

```mermaid
graph TD
    A["Interview scenario"] --> B["Clarify symptoms and impact"]
    B --> C["Check recent changes"]
    C --> D["Gather evidence with commands"]
    D --> E["Form hypothesis"]
    E --> F["Apply safe fix"]
    F --> G["Verify recovery"]
    G --> H["Prevent recurrence"]
```

---

## Final Review Checklist

Use this checklist before a Linux interview:

- I can explain the Linux file system hierarchy.
- I can manage permissions, ownership, users, and groups.
- I can use pipes, redirection, grep, awk, and find effectively.
- I can inspect processes, logs, services, and ports.
- I can troubleshoot SSH, DNS, storage, and performance issues.
- I understand systemd, cron, and shell scripting basics.
- I know LVM, mounts, `/etc/fstab`, and file system basics.
- I understand load average, memory pressure, page cache, and swap.
- I can discuss containers using namespaces and cgroups.
- I can handle scenario questions with clear step-by-step logic.
- I can connect Linux concepts to DevOps, CI/CD, Kubernetes, and SRE work.

---

## Additional Practice Prompts

Use these prompts to extend your preparation:

1. Explain how Linux handles a TCP connection from SYN to application accept.
2. Compare ext4 and XFS for a database workload.
3. Describe how a reverse proxy communicates with an upstream service.
4. Explain what happens when a process writes to a file on a full file system.
5. Design a Linux hardening baseline for an internet-facing VM.
6. Troubleshoot why a container can reach the internet but not another pod.
7. Explain how systemd restarts a crashing service.
8. Investigate a sudden spike in context switches.
9. Explain the impact of a bad DNS resolver on a web app.
10. Describe how you would automate Linux patching safely.

---

## Closing Note

Linux interview success comes from combining command knowledge with systems thinking. The best answers show that you can:
- Operate safely in production
- Troubleshoot methodically
- Understand internals well enough to explain behavior
- Automate repeatable work
- Balance speed, reliability, and security

Study deeply, practice repeatedly, and always verify your assumptions with data.

---

### 🧪 Practice Labs & Hands-On Platforms

#### Linux Practice Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| KillerCoda | https://killercoda.com/ | Browser-based Linux and Kubernetes scenarios with instant sandbox environments. | Yes |
| OverTheWire Bandit | https://overthewire.org/wargames/bandit/ | Step-by-step Linux and shell fundamentals through SSH-based wargames. | Yes |
| OverTheWire Natas | https://overthewire.org/wargames/natas/ | Web and Linux challenge path that builds command-line problem-solving skills. | Yes |
| Hack The Box | https://www.hackthebox.com/ | Realistic Linux-heavy offensive and defensive labs for advanced practice. | Freemium |
| TryHackMe | https://tryhackme.com/ | Guided Linux, security, and enumeration rooms for beginners through intermediate users. | Freemium |
| Linux Survival | https://linuxsurvival.com/ | Interactive browser tutorial for core Linux command-line usage. | Yes |
| Webminal | https://www.webminal.org/ | Online terminal and Linux tutorial environment with guided exercises. | Freemium |
| JSLinux | https://bellard.org/jslinux/ | Run lightweight Linux systems directly in the browser for quick experiments. | Yes |
| Copy.sh | https://copy.sh/ | In-browser x86 emulator for booting Linux images without local setup. | Yes |
| Katacoda (archived) | https://www.katacoda.com/ | Historical interactive scenario platform; useful as a reference, but no longer actively maintained. | Archived |
| Linux Journey | https://linuxjourney.com/ | Structured Linux lessons that pair well with hands-on terminal practice. | Yes |
| edX Linux courses | https://www.edx.org/learn/linux | University and vendor-backed Linux courses with labs and guided exercises. | Freemium |
| Linux Foundation training | https://training.linuxfoundation.org/ | Official Linux Foundation training catalog for admin, cloud, and Kubernetes skills. | Paid |
| Codecademy Linux | https://www.codecademy.com/learn/learn-the-command-line | Interactive command-line practice for Linux newcomers. | Freemium |
| LinuxCommand.org | https://linuxcommand.org/ | Command-line lessons and exercises focused on practical shell usage. | Yes |
| Red Hat Developer Sandbox | https://developers.redhat.com/developer-sandbox | Free Red Hat sandbox environments for OpenShift, containers, and Linux-adjacent practice. | Yes |

#### Kubernetes Practice Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| KillerCoda Kubernetes | https://killercoda.com/kubernetes | Interactive Kubernetes scenarios for cluster, kubectl, and troubleshooting practice. | Yes |
| Play with Kubernetes | https://labs.play-with-k8s.com/ | Free browser-based playground for spinning up short-lived Kubernetes clusters. | Yes |
| Kubernetes.io tutorials | https://kubernetes.io/docs/tutorials/ | Official tutorials covering basics, StatefulSets, services, and cluster concepts. | Yes |
| KodeKloud | https://kodekloud.com/ | Guided Kubernetes labs and certification-focused hands-on tracks. | Freemium |
| Instruqt | https://instruqt.com/ | Vendor-built cloud-native scenarios for Kubernetes and platform engineering workflows. | Freemium |
| Kubernetes The Hard Way | https://github.com/kelseyhightower/kubernetes-the-hard-way | Deep manual cluster build that teaches core Kubernetes internals end to end. | Yes |
| KillerShell / killer.sh | https://killer.sh/ | Exam-style CKA, CKAD, and CKS simulators with timed practice environments. | Paid |
| Katacoda Kubernetes (archived) | https://www.katacoda.com/ | Historical Kubernetes scenarios that are now mostly retained as archived references. | Archived |
| K8s Academy | https://k8sacademy.com/ | Free community Kubernetes learning path with guided cluster topics. | Yes |
| Collabnix | https://collabnix.com/ | Hands-on Kubernetes and container labs, walkthroughs, and curated challenge content. | Yes |
| LFS258 | https://training.linuxfoundation.org/training/kubernetes-fundamentals/ | Linux Foundation Kubernetes fundamentals course aligned to enterprise basics. | Paid |

#### Docker Practice Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| Play with Docker | https://labs.play-with-docker.com/ | Launch Docker playgrounds in the browser for image, networking, and compose practice. | Yes |
| Docker Labs | https://dockerlabs.collabnix.com/ | Collabnix Docker workshop labs covering containers, images, networking, and orchestration basics. | Yes |
| KillerCoda Docker | https://killercoda.com/ | Browser-based Docker labs for container build, run, and troubleshooting tasks. | Yes |
| Instruqt Docker | https://instruqt.com/ | Interactive Docker scenarios created by vendors and training providers. | Freemium |

#### Networking Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| GNS3 | https://www.gns3.com/ | Powerful network emulation platform for routers, switches, and Linux appliances. | Yes |
| Cisco Packet Tracer | https://www.netacad.com/courses/packet-tracer | Beginner-friendly network simulation for routing, switching, and topology practice. | Yes |
| EVE-NG | https://www.eve-ng.net/ | Multi-vendor network emulation platform for advanced lab topologies. | Freemium |
| OverTheWire | https://overthewire.org/wargames/ | Wargames that reinforce Linux networking, SSH, and troubleshooting fundamentals. | Yes |

#### Security Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| Hack The Box | https://www.hackthebox.com/ | Realistic security labs with strong Linux enumeration and privilege-escalation content. | Freemium |
| TryHackMe | https://tryhackme.com/ | Guided rooms for Linux, SOC, web, and offensive security practice. | Freemium |
| PentesterLab | https://pentesterlab.com/ | Web exploitation labs that strengthen Linux, HTTP, and application security workflows. | Freemium |
| VulnHub | https://www.vulnhub.com/ | Downloadable vulnerable VMs for self-hosted Linux and network security practice. | Yes |
| DVWA | https://github.com/digininja/DVWA | Deliberately Vulnerable Web Application for hands-on web security testing. | Yes |
| OWASP Juice Shop | https://owasp.org/www-project-juice-shop/ | Modern insecure web app packed with guided security challenges. | Yes |
| CyberDefenders | https://cyberdefenders.org/ | Blue-team and DFIR labs for SOC, incident response, and log analysis practice. | Freemium |

#### CI/CD and DevOps Labs

| Platform | URL | Description | Free? |
|---|---|---|---|
| GitLab CI tutorials | https://docs.gitlab.com/ci/ | Official tutorials for pipelines, runners, variables, and deployment automation. | Yes |
| GitHub Actions Learning Lab / GitHub Skills | https://skills.github.com/ | GitHub's current interactive learning platform for Actions and workflow automation. | Yes |
| Jenkins tutorials | https://www.jenkins.io/doc/tutorials/ | Official Jenkins tutorials for jobs, pipelines, and automation server setup. | Yes |
| Instruqt | https://instruqt.com/ | Scenario-based DevOps labs spanning CI/CD, platform engineering, and cloud-native tooling. | Freemium |

#### Cloud Practice

| Platform | URL | Description | Free? |
|---|---|---|---|
| AWS Free Tier | https://aws.amazon.com/free/ | Entry point for practicing EC2, IAM, S3, networking, and cloud automation. | Free tier |
| Azure Free Account | https://azure.microsoft.com/free/ | Free credits and always-free services for Azure fundamentals practice. | Free tier |
| GCP Free Tier | https://cloud.google.com/free | Free Google Cloud services and credits for compute, storage, and networking labs. | Free tier |
| LocalStack | https://www.localstack.cloud/ | Local AWS cloud emulator for practicing automation without full cloud spend. | Freemium |
| Cloud Academy | https://cloudacademy.com/ | Structured cloud labs and learning paths for AWS, Azure, and GCP. | Paid |
| A Cloud Guru | https://acloudguru.com/ | Cloud training and labs, now part of Pluralsight Cloud Guru. | Paid |

#### Certification Exam Practice

| Platform | URL | Description | Free? |
|---|---|---|---|
| KillerShell / killer.sh | https://killer.sh/ | Best-known exam simulator for CNCF Kubernetes certifications. | Paid |
| KodeKloud | https://kodekloud.com/ | Certification-aligned labs for Linux, Kubernetes, Ansible, and DevOps tracks. | Freemium |
| Whizlabs | https://www.whizlabs.com/ | Practice tests and hands-on labs for Linux, cloud, and DevOps certifications. | Paid |
| Linux Foundation exams | https://training.linuxfoundation.org/certification/ | Official certification catalog for LFCS, CKA, CKAD, CKS, and related exams. | Paid |
| CompTIA Linux+ | https://www.comptia.org/certifications/linux | Vendor-neutral Linux certification overview with exam objectives and prep guidance. | Paid |

```mermaid
flowchart TD
    A[Linux basics] --> B[Linux command-line labs]
    B --> C[Networking fundamentals]
    B --> D[Docker practice]
    B --> E[Security basics]
    D --> F[Kubernetes labs]
    C --> F
    E --> G[Security challenge labs]
    F --> H[CI/CD and DevOps labs]
    H --> I[Cloud practice]
    F --> J[Certification simulators]
    G --> J
    I --> J
```
