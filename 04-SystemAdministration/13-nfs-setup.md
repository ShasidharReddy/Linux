# NFS Setup

---

<a id="nfs-network-file-system"></a>
## 📂 NFS — Network File System

### What is NFS?
Network File System (NFS) allows a Linux server to export directories over the network so remote systems can mount them as if they were local filesystems.

Core ideas:
- The server exports one or more directories.
- Clients mount those exports onto local mount points.
- Access is controlled by export rules, host matching, and filesystem permissions.
- NFS relies on both network reachability and consistent identity mapping.
- Production deployments should treat NFS as shared infrastructure, not just a convenience mount.

Common use cases:
- Shared home directories for lab or enterprise users.
- Centralized application content such as media, reports, or build outputs.
- Shared storage for analytics or batch-processing nodes.
- Stateful data for container workloads through persistent volumes.
- Shared repositories for backups, ISO images, or software artifacts.

### NFS version overview

| Feature | NFSv3 | NFSv4 / NFSv4.1 / NFSv4.2 |
|---|---|---|
| State model | Mostly stateless | Stateful |
| Default transport | Commonly TCP, historically UDP | TCP |
| Port usage | Multiple services and rpcbind | Primarily TCP 2049 |
| Locking | Separate protocols | Integrated |
| Firewalling | More complex | Simpler |
| ACL behavior | More limited | Better integrated |
| Kerberos support | Possible but less common | Common for secure enterprise use |
| Recommendation | Legacy compatibility only | Preferred for new deployments |

Operational guidance:
- Prefer NFSv4 unless you have a compatibility requirement.
- Standardize UID and GID mapping across systems.
- Avoid using `no_root_squash` unless there is a documented business need.
- Keep exports explicit and least-privileged.
- Monitor latency and stale handle errors for early warning of storage problems.

### NFS request flow

```mermaid
graph LR
    A["Client requests mount"] --> B["Server validates export policy"]
    B --> C["Client mounts export"]
    C --> D["Application reads or writes data"]
    D --> E["Server enforces filesystem permissions"]
    E --> F["Data is committed to storage"]
```

### NFS server prerequisites
Before configuring an NFS server, verify these points:
- The server has a stable IP address or reliable DNS name.
- The exported filesystem has enough capacity and correct ownership.
- UID and GID values are consistent across clients.
- Firewall rules allow the required NFS traffic.
- SELinux or AppArmor policy is understood before production rollout.
- Backup and recovery expectations are documented.

Recommended directory layout:

```bash
sudo groupadd -f projectshare
sudo mkdir -p /srv/nfs/shared
sudo mkdir -p /srv/nfs/readonly
sudo mkdir -p /srv/nfs/projects
sudo chown nobody:nogroup /srv/nfs/shared
sudo chown nobody:nogroup /srv/nfs/readonly
sudo chown root:projectshare /srv/nfs/projects
sudo chmod 2775 /srv/nfs/projects
```

Notes:
- `/srv` is a sensible default for exported service data.
- Set-group-ID on collaborative directories helps keep a shared group.
- If applications require stricter ownership, assign service users explicitly instead of using `nobody`.

### NFS Server Setup (Ubuntu/Debian)
Install the server components:

```bash
sudo apt update
sudo apt install -y nfs-kernel-server
```

Create export directories:

```bash
sudo groupadd -f projectshare
sudo mkdir -p /srv/nfs/shared
sudo mkdir -p /srv/nfs/readonly
sudo mkdir -p /srv/nfs/projects
sudo chown nobody:nogroup /srv/nfs/shared
sudo chown nobody:nogroup /srv/nfs/readonly
sudo chown root:projectshare /srv/nfs/projects
sudo chmod 2770 /srv/nfs/projects
```

Example `/etc/exports`:

```bash
/srv/nfs/shared    192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/readonly  192.168.1.0/24(ro,sync,no_subtree_check,root_squash)
/srv/nfs/projects  192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
```

If you must allow remote root to act as root on the export, the requested form is:

```bash
/srv/nfs/shared  192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/readonly  192.168.1.0/24(ro,sync,no_subtree_check)
```

Production recommendation:
- Use `root_squash` by default.
- Reserve `no_root_squash` for controlled administrative workflows only.
- Restrict by subnet or host, never `*`, on production networks.

Apply the configuration:

```bash
sudo exportfs -ra
sudo systemctl enable --now nfs-kernel-server
sudo systemctl restart nfs-kernel-server
```

Verify:

```bash
sudo exportfs -v
showmount -e localhost
rpcinfo -p localhost
systemctl status nfs-kernel-server
```

### NFS Server Setup (RHEL/CentOS/Rocky/AlmaLinux)
Install the server package:

```bash
sudo dnf install -y nfs-utils
```

Enable the service:

```bash
sudo systemctl enable --now nfs-server
```

Use the same `/etc/exports` structure:

```bash
/srv/nfs/shared    192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/readonly  192.168.1.0/24(ro,sync,no_subtree_check,root_squash)
/srv/nfs/projects  192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
```

Reload exports:

```bash
sudo exportfs -ra
sudo exportfs -v
```

Firewall rules:

```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
```

SELinux considerations on RHEL-family systems:

```bash
sudo setsebool -P nfs_export_all_rw 1
sudo restorecon -Rv /srv/nfs
```

Only enable `nfs_export_all_rw` when it matches your policy. If you export read-only data, consider `nfs_export_all_ro` instead.

### Understanding `/etc/exports` options

| Option | Meaning | Typical use |
|---|---|---|
| `rw` | Read-write export | Shared data and collaborative directories |
| `ro` | Read-only export | Software repositories and reference data |
| `sync` | Reply after data is committed | Safer default |
| `async` | Reply before full commit | Higher performance with higher risk |
| `root_squash` | Map client root to anonymous user | Secure default |
| `no_root_squash` | Preserve client root privileges | Rare, tightly controlled use only |
| `no_subtree_check` | Disable subtree verification | Common and simpler |
| `fsid=0` | NFSv4 pseudo-root | NFSv4 namespace design |
| `sec=krb5p` | Kerberos with privacy | High-security environments |

Example NFSv4 pseudo-root layout:

```bash
/srv/nfs          192.168.1.0/24(ro,fsid=0,sync,no_subtree_check,root_squash)
/srv/nfs/shared   192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/projects 192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
```

This lets clients mount from the NFSv4 namespace root instead of separate exports.

### NFS identity mapping and permissions
NFS permission problems are often identity problems.

Best practices:
- Keep the same UID and GID values for shared users on all nodes.
- Use centralized identity such as LDAP or FreeIPA for larger estates.
- Avoid relying only on usernames; the kernel cares about numeric IDs.
- Validate directory ownership before blaming NFS itself.
- Use ACLs only when your team understands how they interact with exports and applications.

Quick verification:

```bash
id appuser
getent passwd appuser
getent group projectshare
ls -ld /srv/nfs/projects
```

### NFS client setup
Install client utilities:

```bash
sudo apt update
sudo apt install -y nfs-common      # Ubuntu/Debian
```

```bash
sudo dnf install -y nfs-utils       # RHEL/CentOS/Rocky/AlmaLinux
```

Discover exports:

```bash
showmount -e server-ip
rpcinfo -p server-ip
```

Create a mount point and mount manually:

```bash
sudo mkdir -p /mnt/nfs
sudo mount -t nfs server-ip:/srv/nfs/shared /mnt/nfs
```

Force NFSv4 explicitly:

```bash
sudo mount -t nfs -o vers=4 server-ip:/shared /mnt/nfs
```

Confirm the mount:

```bash
mount | grep nfs
findmnt /mnt/nfs
df -h /mnt/nfs
touch /mnt/nfs/client-write-test
ls -l /mnt/nfs/client-write-test
```

### Persistent mounts with `/etc/fstab`
A persistent mount survives reboot and is the usual choice for stable infrastructure.

Example entry:

```fstab
server-ip:/srv/nfs/shared  /mnt/nfs  nfs  defaults,_netdev  0  0
```

Recommended production variant:

```fstab
server-ip:/srv/nfs/shared  /mnt/nfs  nfs  vers=4,hard,timeo=600,retrans=2,_netdev,nofail  0  0
```

Option guidance:
- `_netdev` delays mount handling until network is available.
- `nofail` prevents boot failure if the NFS server is temporarily unavailable.
- `hard` is safer for data integrity because operations keep retrying.
- `timeo` and `retrans` control retry behavior.

Test before rebooting:

```bash
sudo umount /mnt/nfs
sudo mount -a
findmnt /mnt/nfs
```

### AutoFS for on-demand mounting
AutoFS is useful when mounts should appear only when accessed.

Install AutoFS:

```bash
sudo apt install -y autofs     # Ubuntu/Debian
sudo dnf install -y autofs     # RHEL-family
```

Add a master map to `/etc/auto.master`:

```bash
/mnt/nfs  /etc/auto.nfs  --timeout=60
```

Create `/etc/auto.nfs`:

```bash
shared   -fstype=nfs4,rw,hard   server-ip:/srv/nfs/shared
readonly -fstype=nfs4,ro,hard   server-ip:/srv/nfs/readonly
projects -fstype=nfs4,rw,hard   server-ip:/srv/nfs/projects
```

Start the service:

```bash
sudo systemctl enable --now autofs
sudo systemctl restart autofs
```

Test:

```bash
ls /mnt/nfs/shared
findmnt | grep /mnt/nfs
```

### Common mount option choices

| Option | Meaning | When to use |
|---|---|---|
| `hard` | Retry indefinitely | Data that must not silently fail |
| `soft` | Return I/O errors sooner | Read-mostly or less critical workloads |
| `timeo=10` | Lower timeout | Faster failure detection on unstable links |
| `rsize=1048576` | Read transfer size | Large sequential read workloads |
| `wsize=1048576` | Write transfer size | Large sequential write workloads |
| `noatime` | Reduce access time writes | Read-heavy workloads |
| `vers=4.1` | Pin protocol version | Standardization and compatibility control |

Guidance:
- Use `hard` for databases only if the application is designed for network storage failure semantics.
- Avoid `soft` on write-heavy applications unless the risk of partial failures is acceptable.
- Tune `rsize` and `wsize` only after measuring.

### NFS security practices
- Use `root_squash` unless a controlled exception exists.
- Limit exports to specific subnets or hostnames.
- Prefer NFSv4 with Kerberos in regulated environments.
- Treat exported directories like shared trust boundaries.
- Keep host firewalls enabled and explicit.
- Log export changes through configuration management or change control.
- Separate read-only from read-write exports.
- Do not export broad system paths such as `/` or `/home` without policy review.

### Example: shared engineering repository
Server-side preparation:

```bash
sudo groupadd projectshare
sudo mkdir -p /srv/nfs/projects/release-artifacts
sudo chown root:projectshare /srv/nfs/projects/release-artifacts
sudo chmod 2775 /srv/nfs/projects/release-artifacts
```

Export entry:

```bash
/srv/nfs/projects  192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
```

Client-side mount:

```bash
sudo mkdir -p /srv/projects
sudo mount -t nfs -o vers=4 server-ip:/srv/nfs/projects /srv/projects
```

Validation:

```bash
touch /srv/projects/release-artifacts/testfile
ls -ld /srv/projects/release-artifacts
```

### NFS troubleshooting

#### `mount.nfs: access denied by server`
Likely causes:
- Client IP does not match the export rule.
- The export was not reloaded after editing `/etc/exports`.
- Hostname resolution differs from what the server expects.
- Firewall rules block NFS-related services.

Checks:

```bash
sudo exportfs -v
showmount -e server-ip
getent hosts client-hostname
sudo journalctl -u nfs-server -u nfs-kernel-server --since -30m
```

#### Stale NFS handle
This commonly happens when the exported directory or underlying inode changed unexpectedly.

Recovery steps:

```bash
sudo umount -f /mnt/nfs
sudo mount /mnt/nfs
```

If that fails:
- Confirm the path still exists on the server.
- Confirm the filesystem backing the export is mounted.
- Check whether a storage failover or restore changed the underlying object.

#### NFS mount hangs
Possible causes:
- Server unreachable.
- Storage latency on the NFS server.
- DNS delay or reverse lookup issues.
- Packet filtering on intermediate devices.

Useful checks:

```bash
ping -c 4 server-ip
rpcinfo -p server-ip
sudo ss -tanp | grep :2049
sudo dmesg | tail -50
```

For less critical mounts you can use a shorter timeout profile:

```fstab
server-ip:/srv/nfs/shared  /mnt/nfs  nfs  vers=4,soft,timeo=10,retrans=3,_netdev  0  0
```

Use that pattern only when the application can tolerate I/O errors.

#### Permission denied on write
Check these layers in order:
- Export mode is `rw`, not `ro`.
- Server directory ownership is correct.
- UID and GID mapping is consistent.
- SELinux or AppArmor is not blocking access.
- The application user actually has write permission on the directory.

#### Slow performance
Investigate:
- Server disk latency.
- Network packet loss or congestion.
- Small I/O patterns from the application.
- Overly strict synchronous write behavior for the workload.
- Inefficient client mount options.

Basic measurement commands:

```bash
iostat -xz 1 5
sar -n DEV 1 5
nfsstat -m
nfsiostat 1 5
```

### NFS operational checklist
- Confirm exports with `exportfs -v` after every change.
- Validate client access from an allowed and a denied host.
- Document UID and GID expectations.
- Keep the exported storage backed up.
- Monitor disk latency and free space on the server.
- Use configuration management for `/etc/exports` and mount units.
- Test reboot behavior for both server and clients.
- Review whether AutoFS is better than static mounts for edge clients.

### NFS quick reference

```bash
# Server
sudo apt install nfs-kernel-server
sudo dnf install nfs-utils
sudo exportfs -ra
sudo exportfs -v
showmount -e localhost

# Client
sudo apt install nfs-common
sudo dnf install nfs-utils
showmount -e server-ip
sudo mount -t nfs server-ip:/srv/nfs/shared /mnt/nfs
findmnt /mnt/nfs
nfsstat -m
```
