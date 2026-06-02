# Database Security

Authentication, authorization, encryption, backup protection, and security review guidance across database engines.
# 9. Database Security

## 9.1 Security principles

Core ideas:

- Least privilege.
- Defense in depth.
- Encrypted transport.
- Auditable access.
- Secret rotation.
- Backup protection.

## 9.2 Authentication methods

Common methods by engine:

| Engine | Methods |
|---|---|
| MySQL | Native password, caching_sha2_password, LDAP, PAM, TLS certs depending on edition/ecosystem |
| PostgreSQL | peer, scram-sha-256, md5 legacy, cert, LDAP, GSSAPI |
| MongoDB | SCRAM, x.509, LDAP/Kerberos integrations |
| Redis | ACLs, passwords, TLS cert support |
| Elasticsearch | Native users, SSO integrations, TLS certs |

## 9.3 Authorization design

Best practices:

- Separate admin, app, read-only, and backup roles.
- Avoid shared superuser credentials.
- Use roles/groups rather than direct grants when possible.
- Rotate credentials periodically.

## 9.4 Encryption in transit

Use SSL/TLS for:

- Client-to-server traffic.
- Replication traffic.
- Cluster-internal node communication.
- Backup transport.

General Linux tasks:

- Install CA chain.
- Deploy server certificates with correct ownership.
- Restrict file permissions.
- Rotate before expiration.

## 9.5 Encryption at rest

Options include:

- LUKS/dm-crypt volume encryption.
- Filesystem-level encryption where appropriate.
- Engine-level tablespace encryption.
- Cloud block storage encryption.

Practical note:

- At-rest encryption helps with disk theft and some decommissioning risks.
- It does not replace access controls on running systems.

## 9.6 SQL injection prevention

Mandatory application practices:

- Use parameterized queries.
- Avoid string concatenation for SQL.
- Validate input strictly.
- Limit application account privileges.

Bad example:

```python
query = "SELECT * FROM users WHERE email = '" + email + "'"
```

Good example:

```python
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

## 9.7 Audit logging

Log and retain:

- Authentication success and failures.
- Permission changes.
- Schema changes.
- Backup and restore actions.
- Failover actions.
- Administrative sessions.

## 9.8 Backup encryption

Backups should usually be encrypted:

- Before leaving the host.
- At rest in backup storage.
- In transit to backup destinations.

Example with GPG:

```bash
gpg --encrypt --recipient dba-team@example.com appdb.sql.gz
```

Example with OpenSSL symmetric encryption:

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -in appdb.sql.gz -out appdb.sql.gz.enc
```

## 9.9 Secret management

Prefer:

- Vault-based secret injection.
- Kubernetes secrets with external secret management.
- Systemd credentials or protected files.

Avoid:

- Hardcoding credentials in scripts.
- Storing secrets in world-readable configs.
- Reusing one admin password everywhere.

## 9.10 Host hardening for database servers

- Restrict SSH access.
- Keep packages patched.
- Enable firewall rules.
- Disable unnecessary services.
- Use centralized logging.
- Monitor integrity and drift.

## 9.11 Compliance considerations

Depending on industry requirements, document controls for:

- Data retention.
- Audit trails.
- Encryption.
- Access review.
- Backup retention and destruction.
- Incident handling.

---

---

# 20. Database Security Deep Dive

## 20.1 TLS deployment checklist

- CA chain deployed.
- Correct SAN/CN values.
- Key permissions restricted.
- Client verification policy decided.
- Rotation procedure documented.

## 20.2 Least-privilege role examples

Example service account needs only:

- CONNECT on target database.
- USAGE on target schema.
- SELECT/INSERT/UPDATE/DELETE on required tables.
- EXECUTE only on required procedures.

Avoid granting:

- SUPERUSER.
- Global wildcard privileges.
- Schema modification unless required.

## 20.3 Network segmentation

Recommended pattern:

- Clients connect from app subnets only.
- Admin access comes from bastion or VPN.
- Replication traffic isolated when possible.
- Backup endpoints restricted.

## 20.4 Audit review process

At least periodically:

- Review failed login spikes.
- Review privilege changes.
- Review superuser actions.
- Review unusual export patterns.
- Review dormant accounts.

## 20.5 Data masking and non-production safety

Before restoring production data to lower environments:

- Mask PII.
- Remove secrets.
- Rotate keys and tokens.
- Restrict access.

## 20.6 Decommissioning storage safely

When retiring disks or volumes:

- Follow media sanitization policy.
- Verify encryption key destruction where applicable.
- Remove from asset inventory.

## 20.7 Shared responsibility note

In managed services, some controls are handled by provider, but you still own:

- Access management.
- Query security.
- Secret management.
- Backup policy validation.
- Application security.

---

---
