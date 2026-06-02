# Container Security

Containers change packaging and deployment, but they do not eliminate Linux security fundamentals.

Container security depends heavily on kernel isolation features, runtime configuration, image hygiene, and least privilege.

### 12.1 Core Isolation Primitives

Linux container isolation typically uses:

- namespaces
- cgroups
- capabilities
- seccomp
- LSMs such as SELinux or AppArmor

### 12.2 Namespaces

Namespaces isolate views of system resources.

Common namespace types:

| Namespace | Purpose |
| --- | --- |
| pid | Isolate process IDs |
| net | Isolate network stack |
| mnt | Isolate mount points |
| uts | Isolate hostname and domain name |
| ipc | Isolate IPC resources |
| user | Isolate users and groups |
| cgroup | Isolate cgroup view |

Security benefit:

A process inside a container sees a constrained environment rather than the full host perspective.

### 12.3 cgroups

Control groups limit and account for resources.

They are important for:

- CPU limits
- memory limits
- I/O control
- process limits

Security relevance:

- reduce noisy-neighbor damage
- constrain resource exhaustion
- support safer multi-tenant workloads

### 12.4 Linux Capabilities

Capabilities break root privileges into smaller units.

Examples:

- `CAP_NET_BIND_SERVICE`
- `CAP_SYS_ADMIN`
- `CAP_SYS_PTRACE`
- `CAP_CHOWN`

Guidance:

- drop all unnecessary capabilities
- treat `CAP_SYS_ADMIN` as especially dangerous
- review runtime defaults rather than assuming they are safe enough

### 12.5 seccomp

`seccomp` filters system calls available to a process.

Security value:

- reduce kernel attack surface
- block dangerous syscalls not needed by the workload

Best practice:

- use the default runtime seccomp profile as a starting point
- customize carefully for sensitive services
- avoid privileged containers that effectively bypass seccomp value

### 12.6 Rootless Containers

Rootless containers reduce risk by avoiding a root-equivalent daemon or container process path on the host.

Benefits:

- lower host compromise impact in some scenarios
- better fit for developer and CI environments
- fewer direct root-level assumptions

Limitations:

- some networking and storage features may behave differently
- operational tuning may be required

### 12.7 Image Security

Secure images should be:

- minimal
- regularly rebuilt
- scanned for vulnerabilities
- free of embedded secrets
- based on trusted sources

Common guidance:

- prefer distroless or minimal base images where suitable
- remove package managers from runtime images if not needed
- pin and verify dependencies
- sign images when supply-chain tooling supports it

### 12.8 Read-Only Filesystems and Volumes

Many containers do not need write access to the root filesystem.

Helpful controls:

- read-only root filesystem
- dedicated writable volumes for required paths
- `noexec` or other mount restrictions where feasible

### 12.9 Runtime Hardening Checklist

- run as non-root user
- drop unnecessary capabilities
- enable seccomp
- use AppArmor or SELinux confinement
- avoid `--privileged`
- avoid host PID and host network unless required
- mount secrets carefully
- use read-only rootfs where possible

### 12.10 Kubernetes and Orchestrated Environments

Even if this guide is Linux-centric, remember that orchestrators add more layers.

Review:

- pod security settings
- admission controls
- RBAC
- network policies
- secret management
- node hardening

### 12.11 Container Escape Risk

Container escape is less likely when:

- the kernel is patched
- runtime isolation is enabled
- capabilities are minimized
- privileged mode is avoided
- host mounts are limited
- SELinux or AppArmor policy is active

### 12.12 Summary

Container security is Linux security with additional abstraction layers.

Understand namespaces, cgroups, capabilities, seccomp, image hygiene, and rootless operation to keep containers from becoming a false sense of isolation.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.12 Container Security Checklist

- Run containers as non-root where possible.
- Drop unnecessary capabilities.
- Use seccomp profiles.
- Use SELinux or AppArmor confinement.
- Avoid privileged mode.
- Avoid host namespace sharing unless required.
- Use minimal images.
- Scan images regularly.
- Use read-only root filesystems where possible.
- Manage secrets outside image layers.

### B.12 Container Security Commands

```bash
podman ps
docker ps
docker inspect container_id
podman inspect container_id
cat /proc/1/status | grep Cap
```

### C.12 Container Security

131. What are namespaces used for in containers?
132. What are cgroups used for?
133. Why should capabilities be minimized?
134. Why is `CAP_SYS_ADMIN` considered dangerous?
135. What does seccomp do?
136. Why is `--privileged` risky?
137. Why do rootless containers reduce some risks?
138. Why should images be minimal?
139. Why should secrets not be baked into images?
140. Why should host namespace sharing be rare?
