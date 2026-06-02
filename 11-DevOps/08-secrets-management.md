# Secrets Management

[Back to guide index](README.md)

### 8.1 Why secrets management matters
DevOps teams handle credentials for cloud APIs, databases, registries, CI systems, and application runtimes. Poor secret hygiene is one of the fastest ways to create severe incidents.

### 8.2 Secret types
- Passwords
- API keys
- TLS private keys
- Tokens
- SSH keys
- Encryption keys
- Certificates

### 8.3 Environment variables vs files
| Method | Pros | Cons |
|---|---|---|
| Environment variables | Easy injection, common in containers | Can leak in process listings or crash dumps |
| Mounted files | Good for multiline certs/keys | File permission management required |
| Secret manager API | Dynamic and auditable | Requires runtime integration |

### 8.4 HashiCorp Vault overview
Vault provides secret storage, dynamic credentials, encryption as a service, and leasing.

Core concepts:
- Auth methods
- Policies
- Secrets engines
- Leases
- Audit logs

### 8.5 Vault use cases
- Dynamic database credentials.
- PKI issuance.
- Cloud IAM secrets.
- Transit encryption.
- Kubernetes auth.

### 8.6 Vault CLI basics
```bash
vault login
vault kv put secret/app username=app password='supersecret'
vault kv get secret/app
```

### 8.7 SOPS overview
SOPS encrypts YAML, JSON, ENV, and other files using KMS, GPG, or age.

Benefits:
- Encrypted values live in Git.
- Teams can review config safely.
- Works well with GitOps.

### 8.8 SOPS example workflow
```bash
sops secrets.enc.yaml
sops -d secrets.enc.yaml
```

### 8.9 Sealed Secrets for Kubernetes
Sealed Secrets lets you store encrypted Kubernetes secrets in Git and decrypt them only in the target cluster.

Workflow:
1. Create secret manifest.
2. Seal it with controller public key.
3. Commit sealed secret.
4. Controller decrypts in cluster.

### 8.10 Kubernetes secret handling guidance
- Enable encryption at rest.
- Restrict RBAC access.
- Avoid mounting broad secrets into many pods.
- Rotate secrets automatically where possible.
- Prefer external secret operators for cloud secret stores.

### 8.11 External Secrets patterns
Popular integrations:
- AWS Secrets Manager
- AWS SSM Parameter Store
- Azure Key Vault
- Google Secret Manager
- Vault

### 8.12 Secret rotation strategies
- Scheduled rotation.
- Rotation on personnel change.
- Rotation on suspected exposure.
- Dynamic short-lived credentials where possible.

### 8.13 Secret scanning
Tools and practices:
- Pre-commit scanning.
- CI secret scanning.
- Repository scanning.
- DLP rules.

### 8.14 Linux file permission example for secrets
```bash
install -m 600 /dev/null /etc/myapp/secret.env
chown myapp:myapp /etc/myapp/secret.env
```

### 8.15 systemd environment file example
```ini
[Service]
EnvironmentFile=/etc/myapp/secret.env
ExecStart=/opt/myapp/bin/server
```

### 8.16 Secret lifecycle checklist
- Store centrally.
- Audit access.
- Rotate regularly.
- Remove stale secrets.
- Redact from logs.
- Scan repos continuously.

---
