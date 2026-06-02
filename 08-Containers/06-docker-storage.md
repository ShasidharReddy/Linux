# 6. Docker Volumes & Storage

## 6.1 Why Storage Needs Special Attention

Container writable layers are not ideal for persistent data.
If a container is removed, its writable layer typically disappears.

Persistent storage should usually live in:

- Named volumes
- Bind mounts
- External storage systems

## 6.2 Storage Types in Docker

| Type | Managed By | Best For |
|---|---|---|
| Named volume | Docker | Persistent app data |
| Bind mount | Host filesystem | Dev workflows, specific host file integration |
| tmpfs | Memory | Sensitive or temporary runtime data |

## 6.3 Named Volumes

Named volumes are Docker-managed storage objects.
They are generally the preferred choice for persistent container data.

Create and use:

```bash
docker volume create pgdata
```

```bash
docker run -d --name db -v pgdata:/var/lib/postgresql/data postgres:16
```

## 6.4 Benefits of Named Volumes

- Managed by Docker
- Easier backup/migration than ad hoc writable layers
- Less tied to exact host path layout
- Better portability than many bind-mount setups

## 6.5 Bind Mounts

Bind mounts map a host path into a container.

Example:

```bash
docker run -v $(pwd):/app -w /app node:22 npm test
```

Common use cases:

- Local development
- Mounting configuration files
- Sharing source code with tools running in containers

## 6.6 Bind Mount Risks

- Ties workload to specific host path structure
- Permissions mismatches can cause failures
- Container may modify host files
- Can reduce portability and reproducibility

## 6.7 tmpfs Mounts

A tmpfs mount stores data in memory.
It is useful for:

- Temporary files
- Sensitive data that should not hit disk
- Performance-sensitive ephemeral scratch data

Example:

```bash
docker run --tmpfs /run:rw,noexec,nosuid,size=64m myapp:latest
```

## 6.8 Volume Syntax Variants

Short syntax examples:

```bash
docker run -v myvol:/data myapp:latest
```

```bash
docker run -v /host/path:/data:ro myapp:latest
```

Long syntax example:

```bash
docker run --mount type=volume,src=myvol,dst=/data myapp:latest
```

Long syntax is often clearer and less error-prone.

## 6.9 Read-Only Mounts

Use read-only mounts when writes are unnecessary.

```bash
docker run --mount type=bind,src=$(pwd)/config,dst=/app/config,readonly myapp:latest
```

## 6.10 Volume Inspection

```bash
docker volume ls
```

```bash
docker volume inspect pgdata
```

## 6.11 Backing Up a Volume

Example using a temporary helper container:

```bash
docker run --rm \
  -v pgdata:/source:ro \
  -v $(pwd):/backup \
  alpine sh -c 'cd /source && tar czf /backup/pgdata-backup.tar.gz .'
```

## 6.12 Restoring a Volume

```bash
docker run --rm \
  -v pgdata:/target \
  -v $(pwd):/backup \
  alpine sh -c 'cd /target && tar xzf /backup/pgdata-backup.tar.gz'
```

## 6.13 Volume Drivers

Docker supports plugins/drivers for specialized storage backends.
These may integrate with:

- NFS
- Cloud block storage
- Distributed filesystems
- Vendor-specific CSI-like solutions in broader ecosystems

## 6.14 Storage Driver vs Volume Driver

These are different concepts.

| Term | Meaning |
|---|---|
| Storage driver | How image/container layers are stored, such as `overlay2` |
| Volume driver | How Docker-managed volumes are provisioned/accessed |

## 6.15 Overlay2 and Writable Layers

The container writable layer is fine for transient changes but not ideal for durable application data.
Reasons include:

- Performance considerations
- Lifecycle coupling to container
- Harder backup semantics

## 6.16 Database Storage Guidance

For databases:

- Use named volumes or dedicated host/cloud storage
- Know your fsync and durability assumptions
- Monitor free space and I/O behavior
- Back up independently of the container lifecycle

## 6.17 File Ownership and Permissions

Common source of problems:

- Host path owned by one UID/GID
- Container process runs as another UID/GID

Fixes may include:

- Matching container UID/GID
- Pre-chowning host directories
- Using named volumes where possible
- Using user namespace or rootless-aware setups carefully

## 6.18 Mount Propagation and Recursive Options

Most users do not need advanced mount propagation settings, but they matter for:

- Nested containers
- Kubernetes hostPath edge cases
- System-level tooling

## 6.19 Volume Lifecycle

Named volumes outlive containers unless explicitly removed.
This is valuable but can also create stale-data surprises.

Useful cleanup:

```bash
docker volume prune
```

Use carefully.

## 6.20 Example: Nginx with Read-Only Static Content

```bash
docker run -d \
  --name site \
  -p 8080:80 \
  --mount type=bind,src=$(pwd)/public,dst=/usr/share/nginx/html,readonly \
  nginx:stable
```

## 6.21 Example: PostgreSQL with Named Volume

```bash
docker run -d \
  --name pg \
  -e POSTGRES_PASSWORD=secret \
  --mount type=volume,src=pgdata,dst=/var/lib/postgresql/data \
  postgres:16
```

## 6.22 Example: tmpfs for Temporary Secrets Material

```bash
docker run -d \
  --name secure-app \
  --read-only \
  --tmpfs /run/secrets:rw,noexec,nosuid,size=16m \
  myapp:latest
```

## 6.23 Storage Performance Considerations

Performance varies by:

- Host filesystem
- Storage driver
- Bind vs volume choice
- OS platform
- Virtualization layer

On macOS/Windows with desktop virtualization, bind mounts can be significantly slower than native Linux.

## 6.24 Data Management Best Practices

- Separate code, config, and data clearly
- Back up volumes explicitly
- Avoid storing mutable data in image layers
- Use read-only mounts where possible
- Document data paths in runbooks

## 6.25 Summary

Persistent data should be treated as a first-class operational concern.
Named volumes are usually the cleanest default, bind mounts are powerful but host-coupled, and tmpfs is excellent for ephemeral or sensitive scratch data.

---

## Appendix A.6 Storage Quick Reference

- Named volumes for persistent data
- Bind mounts for dev or specific integration
- tmpfs for ephemeral sensitive data

---

## B.6 Storage Q&A

### Q101. Why is container writable layer poor for persistent data?
A101. It is ephemeral and tightly coupled to container lifecycle.

### Q102. What is a named volume?
A102. Docker-managed persistent storage.

### Q103. What is a bind mount?
A103. A direct mount of a host path into the container.

### Q104. What is tmpfs?
A104. An in-memory filesystem mount.

### Q105. When are bind mounts especially useful?
A105. Local development and host-file integration.

### Q106. Why can bind mounts cause permission issues?
A106. Host UID/GID ownership may not match the container user.

### Q107. What command creates a named volume?
A107. `docker volume create <name>`.

### Q108. What command lists volumes?
A108. `docker volume ls`.

### Q109. What command inspects a volume?
A109. `docker volume inspect <name>`.

### Q110. Why use read-only mounts?
A110. To reduce accidental or malicious modification.

### Q111. What is the difference between storage driver and volume driver?
A111. Storage driver handles image/container layers; volume driver handles managed volumes.

### Q112. Why are named volumes often better than bind mounts in production?
A112. They are less host-path-coupled and easier to manage consistently.

### Q113. Why are tmpfs mounts good for secrets material?
A113. Data stays in memory instead of being written to disk.

### Q114. Why should volume backup be explicit?
A114. Containers are easy to redeploy, but data loss is permanent.

### Q115. Can named volumes outlive containers?
A115. Yes.

### Q116. What is a common mistake with `docker compose down -v`?
A116. Accidentally deleting persistent data volumes.

### Q117. Why can storage performance differ across platforms?
A117. Filesystem drivers and virtualization layers vary, especially on desktop OSes.

### Q118. Why should data paths be documented?
A118. For operations, backup, restore, and incident response.

### Q119. Why might a non-root container fail to write to a mounted path?
A119. Ownership/permission mismatch.

### Q120. What is a good default for database storage in Docker?
A120. Named volumes or dedicated external storage.

---

## F.2 Mount Type Summary

| Type | Typical Usage |
|---|---|
| Named volume | Persistent app data |
| Bind mount | Dev and host path integration |
| tmpfs | Ephemeral memory-backed data |
