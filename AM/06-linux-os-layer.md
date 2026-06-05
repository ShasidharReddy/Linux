# Question 6: OS Layer — Linux Logic

Infrastructure becomes maintainable when the team understands Linux as a platform, not just as a place to paste commands. This chapter frames the operating system layer the way production operators actually use it.

## Linux as a Platform — Mental Model

- hardware provides CPU, RAM, storage, and NICs
- the kernel arbitrates devices, scheduling, memory, networking, and security hooks
- system services form the platform behavior operators interact with
- user space holds applications, shells, package managers, and tooling

## Linux architecture layers

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Hardware] --> B[Kernel]
    B --> C[System Services]
    C --> D[User Space]
    D --> E[Applications and Operators]
~~~

## Service / Process Model (systemd)

### Unit types you will use most

| Unit | Purpose | Example |
|------|---------|---------|
| service | long-running daemon or one-shot action | `nginx.service` |
| socket | socket activation trigger | `docker.socket` |
| timer | scheduled execution | `logrotate.timer` |
| mount | filesystem mount | `var-lib-containers.mount` |
| target | grouping and boot stage | `multi-user.target` |

### Lifecycle states

- loaded
- active
- inactive
- failed
- activating / deactivating

### Common commands

~~~bash
systemctl status nginx
systemctl list-units --type=service --state=running
systemctl enable --now chronyd
journalctl -u sshd -b
journalctl -xe
~~~

## systemd boot sequence

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[firmware] --> B[bootloader]
    B --> C[kernel + initramfs]
    C --> D[systemd]
    D --> E[basic.target]
    E --> F[multi-user.target]
    F --> G[application services]
~~~

## Example custom unit file

`/etc/systemd/system/inventory-api.service`

~~~ini
[Unit]
Description=Inventory API
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service

[Service]
User=inventory
Group=inventory
WorkingDirectory=/srv/inventory-api
Environment=APP_ENV=production
ExecStart=/usr/local/bin/inventory-api --config /etc/inventory-api/config.yaml
Restart=on-failure
RestartSec=5
MemoryMax=1G
CPUQuota=200%
LimitNOFILE=65535
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/inventory-api /var/log/inventory-api

[Install]
WantedBy=multi-user.target
~~~

### Why these directives matter

- `After=` controls startup order
- `Requires=` expresses a hard dependency
- `Wants=` expresses a softer preference
- `MemoryMax=` and `CPUQuota=` stop one service from consuming the whole host
- sandboxing directives reduce blast radius

## Socket activation note

Use socket activation when:

- a service is infrequently used
- fast boot matters more than persistent warm processes
- you want systemd to hold the listening socket until the app starts

## Package management

### DNF/YUM

~~~bash
dnf repolist
dnf module list
dnf group list
dnf install -y nginx
dnf update -y
~~~

### APT

~~~bash
apt update
apt policy nginx
apt install -y nginx
apt full-upgrade -y
~~~

## Package management decision notes

| Family | Strong Fit | Operational Notes |
|--------|------------|-------------------|
| DNF/YUM | Rocky, Alma, RHEL | modules, easy repo mirroring, SELinux-native ecosystem |
| APT | Debian, Ubuntu | large package ecosystem, common in container and K8s environments |

## Local repository mirror

Why use one:

- reproducibility during maintenance windows
- faster patching
- less exposure to upstream outage or compromise
- cleaner approval of package sources

### RHEL-family local mirror example

~~~bash
dnf install -y createrepo_c httpd reposync
reposync -p /srv/repos/rocky9-baseos --download-metadata --repoid=baseos
createrepo_c /srv/repos/rocky9-baseos/baseos
~~~

### Debian-family mirror example

~~~bash
apt install -y apt-mirror nginx
vim /etc/apt/mirror.list
apt-mirror
~~~

## Filesystem Hierarchy Standard

| Path | Purpose |
|------|---------|
| `/etc` | configuration |
| `/var` | variable data, logs, spool |
| `/opt` | third-party packaged software |
| `/srv` | service-owned data |
| `/home` | human user data |
| `/tmp` | temporary runtime data |

## Partition strategy for production

A practical baseline:

- `/` modest but not tiny
- `/var` separate for app and package activity
- `/var/log` separate to prevent logs filling root
- `/tmp` separate or tmpfs based on policy
- `/home` often small on servers
- `/opt` or `/srv` separate when apps write large data there

### LVM example

~~~bash
pvcreate /dev/sdb /dev/sdc
vgcreate vg_system /dev/sdb /dev/sdc
lvcreate -L 50G -n lv_var vg_system
lvcreate -L 20G -n lv_varlog vg_system
mkfs.xfs /dev/vg_system/lv_var
mkfs.xfs /dev/vg_system/lv_varlog
~~~

## XFS vs ext4

| Filesystem | Choose When | Trade-off |
|------------|-------------|-----------|
| XFS | large filesystems, modern RHEL defaults, online growth | cannot shrink easily |
| ext4 | simple general-purpose Linux hosts | fewer large-scale tuning features |

## Networking stack

### Common Linux approaches

| Tooling | Strength |
|--------|----------|
| NetworkManager | common on enterprise Linux, integrates with nmcli |
| systemd-networkd | lean and declarative on modern systemd hosts |
| `/etc/network/interfaces` | direct and common in Proxmox/Debian virtualization contexts |

### Diagnostics commands

~~~bash
ip -br addr
ip route
ss -tulpn
nmcli device status
ethtool eno1
nft list ruleset
~~~

## Networking relationships

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    NIC[Physical NIC] --> BOND[Bond]
    BOND --> VLAN[VLAN subinterfaces]
    VLAN --> BRIDGE[Linux bridge]
    BRIDGE --> VM[VMs or containers]
~~~

## Permission & security model

### User classes

- **service accounts**: non-login, app-owned, minimal privileges
- **human accounts**: named users, MFA or key-backed auth, audited sudo
- **automation accounts**: narrowly scoped and documented

### sudo policy guidance

- humans should usually enter a password for privileged actions
- automation can use controlled `NOPASSWD` for exactly defined commands
- use `/etc/sudoers.d/` drop-ins, never edit the main file blindly

### sudoers example

~~~bash
Cmnd_Alias AUTOMATION = /usr/bin/systemctl restart *, /usr/bin/dnf *, /usr/bin/apt *
%ops-admins ALL=(ALL) ALL
ansible ALL=(ALL) NOPASSWD: AUTOMATION
~~~

### File permission reminders

- review SUID/SGID binaries regularly
- keep sticky bit on `/tmp`
- use ACLs when Unix mode bits are not expressive enough

~~~bash
find / -perm /6000 -type f 2>/dev/null
getfacl /srv/appdata
setfacl -m u:appuser:rwx /srv/appdata
~~~

## SELinux and PAM

### SELinux building blocks

- contexts label files and processes
- booleans tune common behaviors safely
- audit logs explain denials

~~~bash
ls -Z /var/www/html
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect on
ausearch -m avc -ts recent
~~~

### PAM controls

- `faillock` for lockout
- `pam_limits` for resource ceilings
- MFA modules where required

## Keeping config repeatable

- Ansible should be the source of truth for host configuration
- use `etckeeper` so `/etc` changes have history
- prefer immutable rebuilds for cattle-like servers
- make infra changes go through PR review and automation

### etckeeper example

~~~bash
dnf install -y etckeeper git || apt install -y etckeeper git
etckeeper init
etckeeper commit "Baseline OS config"
~~~

## Config management workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[Git PR] --> B[Review and Merge]
    B --> C[CI Lint and Syntax Check]
    C --> D[Ansible Apply]
    D --> E[Drift Check and Monitoring]
~~~

## Patch management workflow

1. stage updates in non-production
2. validate applications and agents
3. snapshot or ensure rollback path exists
4. patch one host or one VM group at a time
5. verify service health and logs
6. update the golden image baseline when the patch set is approved

## Common mistakes

- treating systemd as a black box
- storing application data in random paths instead of FHS-aligned locations
- allowing direct manual edits on many hosts without version control
- disabling SELinux because an app needs a quick fix
- using shared admin accounts instead of named access

## Exit criteria

You are ready for [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) when:

- service management is standardized with systemd
- package source policy is documented
- filesystem layout and LVM policy are defined
- host config changes flow through automation and version control


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | systemd as standard service manager | predictable dependency model and controls | ad hoc init scripts rejected |
| ADR-02 | local package mirror | repeatability and controlled patch windows | direct Internet repos only rejected |
| ADR-03 | separated filesystems with LVM | easier growth and log containment | monolithic root filesystem rejected |
| ADR-04 | SELinux/AppArmor left enabled | stronger host isolation | disabling MAC controls rejected |

### Integration points

- Hardening inputs come from [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md).
- Host firewall policy ties back to [03-firewall-and-security.md](./03-firewall-and-security.md).
- Container hosts and Kubernetes nodes in [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) and [09-kubernetes-deployment.md](./09-kubernetes-deployment.md) inherit this baseline.

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

- Consider live patching only for fleets where reboot windows are genuinely expensive; it is not a substitute for regular maintenance.
- Use OpenSCAP/OSCAP profiles in CI to validate image compliance before promotion.
- DR pattern: keep repo mirrors, automation repos, and secrets replicated so hosts can be rebuilt in another site.
- Compliance impact: SOC 2 wants evidence of change control, PCI often requires stronger file-integrity and account controls, HIPAA emphasizes auditability and access review.
- Capacity alerts: `/var` or `/var/log` >80%, package mirror sync failure, SELinux disabled, too many failed sudo/PAM auths, drift detected in critical files.
- Auto-remediation can rotate logs, restart non-critical services, or re-apply a baseline role; kernel and auth changes stay human-approved.

## Building & Deployment Runbook

1. Select the supported OS family and lock the major version in the image pipeline.
2. Build repo access and baseline packages:
   ~~~bash
   dnf repolist || apt update
   dnf install -y chrony lvm2 audit etckeeper || apt install -y chrony lvm2 auditd etckeeper
   ~~~
3. Apply filesystem and mount policy, then validate:
   ~~~bash
   lsblk
   vgs
   lvs
   findmnt / /var /var/log /tmp /opt
   ~~~
4. Enforce service, PAM, and SELinux/AppArmor policy with Ansible.
5. Run compliance validation:
   ~~~bash
   oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_server_l1 /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
   sestatus
   auditctl -s
   ~~~
6. Validation gate: patch one canary host, confirm apps and agents, then roll through the rest of the ring.

### Common mistakes to avoid during build

- Choosing a distro based only on engineer preference and ignoring lifecycle/support needs.
- No training plan for SELinux/AppArmor and then disabling controls during the first incident.
- Ignoring the cost of downtime while arguing over subscription fees.
- Letting package sources drift across environments.


## Cross-references

- VM hardening inputs: [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md)
- Security controls: [03-firewall-and-security.md](./03-firewall-and-security.md)
- Related repo references: [../Physical-Setup/02-os-installation-and-hardening.md](../Physical-Setup/02-os-installation-and-hardening.md), [../Virtual-Setup/03-docker-fundamentals.md](../Virtual-Setup/03-docker-fundamentals.md)
