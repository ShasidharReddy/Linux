# 9. Container Security

## 9.1 Security Is Shared Responsibility

Containers can improve consistency and reduce some risks, but they do not automatically make workloads secure.
You must secure:

- Images
- Runtime configuration
- Host OS
- Registry access
- Supply chain
- Secrets
- Network exposure

## 9.2 Threat Categories

Common container security risks:

- Vulnerable base images
- Excessive runtime privileges
- Secrets baked into images
- Overexposed ports
- Untrusted registries
- Insecure CI/CD pipelines
- Weak image provenance
- Host kernel escape vulnerabilities

## 9.3 Root vs Rootless

### Root in Container

Running as root inside a container is common but risky.
Even if isolated, container root often has more power than necessary.

### Rootless Containers

Rootless mode reduces privilege by running daemon and containers without host root.
This can significantly improve safety, especially in multi-user systems.

## 9.4 Root vs Rootless Table

| Aspect | Rootful | Rootless |
|---|---|---|
| Privilege level | Higher | Lower |
| Compatibility | Broadest | Some limitations |
| Attack surface | Larger | Reduced |
| Ease of adoption | Often simpler initially | May need more setup |

## 9.5 Run as Non-Root User

Inside the image, create and use a non-root user whenever possible.

```dockerfile
RUN useradd -r -u 10001 appuser
USER appuser
```

If the app must bind to low ports, consider capabilities or use a reverse proxy rather than making the whole process root.

## 9.6 Read-Only Filesystems

A read-only root filesystem reduces runtime mutation opportunities.

```bash
docker run --read-only --tmpfs /tmp myapp:latest
```

Benefits:

- Limits persistence of malicious writes
- Encourages better filesystem design
- Prevents accidental in-container drift

## 9.7 Resource Limits as Security Controls

CPU, memory, and PID limits are not just performance controls.
They also reduce denial-of-service blast radius.

Example:

```bash
docker run --memory=512m --cpus=1 --pids-limit=200 myapp:latest
```

## 9.8 Drop Capabilities

Start from least privilege.

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp:latest
```

Avoid:

```bash
docker run --privileged myapp:latest
```

Unless you fully understand the implications and absolutely need it.

## 9.9 Privileged Containers

`--privileged` effectively disables many isolation protections.
It can grant access to devices and broad kernel functionality.
This should be exceptional, not normal.

## 9.10 seccomp Profiles

Docker applies a default seccomp profile in many environments.
Keep it enabled unless you must adjust it for a specific syscall requirement.

## 9.11 AppArmor/SELinux Profiles

Use host security modules as another containment layer.
Ensure your operational platform supports and enforces them consistently.

## 9.12 Image Scanning

Image scanning identifies known vulnerabilities in packages and OS layers.
Common tools:

- Trivy
- Snyk
- Grype
- Registry-native scanners

### Trivy example

```bash
trivy image myapp:latest
```

### Snyk example

```bash
snyk container test myapp:latest
```

## 9.13 Interpreting Scan Results

Do not blindly chase every CVE count.
Consider:

- Is the package actually present in runtime path?
- Is the vulnerable component reachable?
- Is there a fix available?
- What is the exploitability in your context?

Still, high and critical issues in active packages should be addressed quickly.

## 9.14 Base Image Hygiene

- Use official or trusted vendor images
- Rebuild frequently for security patches
- Pin versions/digests
- Remove unnecessary software
- Prefer smaller runtime images

## 9.15 Supply Chain Security

Important topics:

- Provenance
- SBOMs
- Signing
- Trusted builders
- Dependency pinning

## 9.16 Image Signing

Image signing helps verify authenticity and integrity.
Common ecosystem tools include:

- Cosign
- Notation

Example concept with Cosign:

```bash
cosign sign registry.example.com/team/myapp:1.2.3
```

Verification:

```bash
cosign verify registry.example.com/team/myapp:1.2.3
```

## 9.17 SBOMs

SBOM stands for Software Bill of Materials.
It lists components included in the image.
This helps with:

- Vulnerability management
- Compliance
- Incident response
- Dependency visibility

## 9.18 Secrets Management Principles

Never bake secrets into:

- Dockerfiles
- Image layers
- Git repositories
- Default environment files committed to source control

Prefer:

- Runtime secret injection
- Secret managers
- Short-lived credentials
- BuildKit secret mounts for build-time secrets

## 9.19 Runtime Secret Options

Options vary by platform:

- Docker Swarm secrets
- Compose secrets in supported scenarios
- Kubernetes Secrets with extra hardening
- External systems like Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager

## 9.20 Environment Variables and Secrets

Environment variables are convenient but have caveats:

- May appear in process metadata
- May leak through logs or diagnostics
- May persist in deployment definitions

Use them carefully for low/medium sensitivity config; prefer stronger secret mechanisms for high sensitivity data.

## 9.21 Network Hardening

- Publish only necessary ports
- Use private networks for internal traffic
- Restrict egress where possible
- Segment workloads by trust boundary
- Apply TLS for service communication as appropriate

## 9.22 Registry Security

- Enforce authentication and authorization
- Use least-privilege tokens
- Enable image scanning
- Require signed images where possible
- Audit push/pull activity

## 9.23 CI/CD Security for Containers

- Build from trusted sources
- Protect build secrets
- Use ephemeral runners when possible
- Scan images before push/deploy
- Sign release images
- Block deployment of untrusted or high-risk artifacts

## 9.24 Rootless Docker Notes

Rootless Docker can reduce daemon risk, but remember:

- It may have networking/storage differences
- Some privileged operations are unavailable
- Not every legacy workflow fits rootless cleanly

## 9.25 Distroless Security Trade-Off

Distroless images reduce attack surface because they exclude shells and package managers.
But debugging becomes harder.
Mitigation strategies:

- Use ephemeral debug containers
- Keep symbols/artifacts elsewhere
- Reproduce with a debug image variant if needed

## 9.26 File Permissions and Ownership

Harden writable paths.
Prefer:

- Minimal writable directories
- Correct ownership for app user
- Read-only mounts for config and code where possible

## 9.27 Securing the Host

Container security depends on host security.
Protect:

- Kernel patch levels
- SSH/admin access
- Audit logging
- Runtime versions
- Storage/network configurations
- Monitoring and alerting

## 9.28 Example Hardened Run Command

```bash
docker run -d \
  --name api \
  --user 10001:10001 \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt no-new-privileges:true \
  --pids-limit=200 \
  --memory=512m \
  --cpus=1 \
  -p 127.0.0.1:8080:8080 \
  myorg/api:1.2.3
```

## 9.29 `no-new-privileges`

This security option prevents processes from gaining new privileges via mechanisms like setuid binaries.
It is a valuable hardening flag.

## 9.30 Security Checklist

- Use trusted, minimal base images
- Rebuild often
- Run as non-root
- Use rootless where feasible
- Drop capabilities
- Keep seccomp enabled
- Use LSM profiles
- Set read-only rootfs when possible
- Limit CPU/memory/pids
- Scan images
- Sign release artifacts
- Manage secrets safely
- Harden host OS and registry

## 9.31 Summary

Container security requires layered defenses across build, image, runtime, host, and supply chain.
Least privilege, image hygiene, and strong secret handling are foundational.

---

## Appendix A.7 Security Quick Reference

- Least privilege
- Non-root users
- Read-only root filesystem
- Capability dropping
- Scanning and signing

---

## B.9 Security Q&A

### Q161. Why run as non-root?
A161. To reduce privilege and blast radius.

### Q162. What does `--read-only` do?
A162. Makes the container root filesystem read-only.

### Q163. Why use `--tmpfs /tmp` with read-only rootfs?
A163. Many apps still need a writable temporary directory.

### Q164. What does `--cap-drop=ALL` do?
A164. Removes all Linux capabilities by default.

### Q165. Why is `--privileged` risky?
A165. It effectively disables many isolation protections.

### Q166. What does `no-new-privileges` help prevent?
A166. Gaining extra privileges via exec/setuid-style behavior.

### Q167. What is image scanning for?
A167. Detecting known vulnerabilities in image contents.

### Q168. Name two common image scanners.
A168. Trivy and Snyk.

### Q169. What is image signing for?
A169. Verifying authenticity and integrity of images.

### Q170. What is an SBOM?
A170. Software Bill of Materials.

### Q171. Why are secrets in Dockerfiles bad?
A171. They can persist in layers and leak widely.

### Q172. Are environment variables always safe for secrets?
A172. No.

### Q173. What is rootless mode's main security benefit?
A173. Reduced host-root exposure.

### Q174. Why harden the registry too?
A174. Because it is part of the software supply chain.

### Q175. Why should images be rebuilt frequently?
A175. To pick up security patches.

### Q176. What is a trusted base image?
A176. An image from a reputable, maintained source.

### Q177. Why pin digests for critical workloads?
A177. To guarantee exact image content.

### Q178. Why use minimal runtime images?
A178. Reduced attack surface and smaller vulnerability footprint.

### Q179. Why does host security still matter with containers?
A179. Containers share the host kernel and depend on host controls.

### Q180. What is least privilege in container context?
A180. Granting only the permissions, resources, and access the workload truly needs.

---

## F.3 Security Flag Summary

| Option | Purpose |
|---|---|
| `--user` | Run as specified UID/GID |
| `--read-only` | Read-only root filesystem |
| `--tmpfs` | Writable temp path in memory |
| `--cap-drop=ALL` | Remove capabilities |
| `--cap-add` | Add only required capabilities |
| `--security-opt no-new-privileges:true` | Prevent gaining new privileges |
| `--pids-limit` | Limit process count |
| `--memory` | Limit memory |
| `--cpus` | Limit CPU |
