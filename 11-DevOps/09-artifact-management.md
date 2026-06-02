# Artifact Management

[Back to guide index](README.md)

### 9.1 Why artifact management matters
Artifacts are the outputs of build pipelines. Without a managed artifact repository or registry, builds are harder to reproduce and releases are harder to audit.

### 9.2 Artifact types
- Application packages
- Binary tarballs
- Maven packages
- npm packages
- Python wheels
- Helm charts
- Container images
- SBOMs

### 9.3 JFrog Artifactory overview
Artifactory supports many repository types and enterprise promotion patterns.

Use cases:
- Central package management.
- Proxying upstream dependencies.
- Build info and traceability.
- Promotion across lifecycle stages.

### 9.4 Sonatype Nexus overview
Nexus Repository supports hosted, proxy, and group repositories for multiple package formats.

### 9.5 Container registries
Common options:
- Harbor
- Amazon ECR
- Google GCR or Artifact Registry
- Azure ACR
- Docker Hub
- GitHub Container Registry

### 9.6 Registry practices
- Use immutable tags when possible.
- Track digests.
- Enforce vulnerability scanning.
- Control retention.
- Sign artifacts.

### 9.7 Harbor overview
Harbor adds enterprise features like role-based access, replication, scanning, and content trust.

### 9.8 Image naming example
```text
registry.example.com/platform/payments-api:1.4.2
registry.example.com/platform/payments-api@sha256:abcd...
```

### 9.9 Docker login and push
```bash
echo "$REGISTRY_PASSWORD" | docker login registry.example.com -u "$REGISTRY_USER" --password-stdin
docker build -t registry.example.com/team/app:1.0.0 .
docker push registry.example.com/team/app:1.0.0
```

### 9.10 Helm chart repositories
Helm charts can be stored in:
- Artifactory
- Nexus
- Harbor
- OCI registries

Example OCI push:
```bash
helm package charts/myapp
helm push myapp-1.0.0.tgz oci://registry.example.com/helm
```

### 9.11 Dependency proxy benefits
A proxy cache:
- Reduces internet dependency.
- Improves speed.
- Helps during upstream outages.
- Supports compliance review.

### 9.12 Promotion models
| Model | Description |
|---|---|
| Copy | Duplicate artifact between repos |
| Move | Promote artifact to next stage repo |
| Metadata tag | Mark artifact state without copying |
| Digest pinning | Promote by immutable digest |

### 9.13 SBOM and provenance
Modern artifact management often includes:
- SBOM generation.
- Signature storage.
- Provenance attestations.
- Scan reports.

### 9.14 Linux storage considerations for artifact repos
- Fast disk for metadata-heavy operations.
- Retention cleanup.
- Backup strategy.
- TLS and certificate management.

### 9.15 Artifact repository checklist
- Access controls defined.
- Upstream proxies configured.
- Retention enforced.
- Vulnerability scanning enabled.
- Backup and restore tested.

---
