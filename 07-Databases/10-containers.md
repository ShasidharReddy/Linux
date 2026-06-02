# Databases in Containers

Container, Compose, Kubernetes, StatefulSet, operator, and backup guidance for containerized databases.
# 10. Database in Containers

## 10.1 When containers make sense

Containers are excellent for:

- Development environments.
- CI integration testing.
- Standardized deployment artifacts.
- Some production workloads with mature storage and operations.

## 10.2 Docker basics

### MySQL example

```bash
docker run -d \
  --name mysql8 \
  -e MYSQL_ROOT_PASSWORD=StrongPassword \
  -p 3306:3306 \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

### PostgreSQL example

```bash
docker run -d \
  --name pg16 \
  -e POSTGRES_PASSWORD=StrongPassword \
  -p 5432:5432 \
  -v pg_data:/var/lib/postgresql/data \
  postgres:16
```

## 10.3 Docker Compose for development

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: StrongPassword
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"
    command: ["redis-server", "--appendonly", "yes"]

volumes:
  pg_data:
```

## 10.4 Production concerns in containers

- Persistent storage class quality.
- Backup integration.
- Node failure behavior.
- Pod disruption budgets.
- Resource requests and limits.
- Startup and readiness probes.

## 10.5 Kubernetes StatefulSets

StatefulSets help databases by providing:

- Stable pod identities.
- Stable network identities.
- Ordered startup and termination.
- Persistent volume claims per replica.

Example outline:

- Headless service.
- StatefulSet.
- PVC templates.
- Secrets and config maps.
- Backup sidecars or external jobs.

## 10.6 Persistent volumes

Critical questions:

- What is the storage latency?
- Does it guarantee write ordering?
- What happens on node failure?
- Are snapshots crash-consistent or application-consistent?

## 10.7 Operators

### CloudNativePG

A mature PostgreSQL operator for Kubernetes.

Features:

- Backup automation.
- HA orchestration.
- Declarative cluster management.

### MySQL Operator

Oracle and ecosystem variants exist for automating MySQL cluster deployments.

### MongoDB Operator

Operators can automate replica sets, sharding, backups, and upgrades depending on the implementation.

## 10.8 Container security notes

- Run with minimal privileges.
- Use secrets, not env vars where possible.
- Encrypt storage classes when available.
- Limit network policies.
- Avoid running databases in arbitrary ephemeral pods without persistence.

## 10.9 Backup in containerized environments

Approaches:

- Logical backups via Kubernetes CronJobs.
- Physical backups to object storage.
- Operator-managed backups.
- Volume snapshots with application consistency coordination.

---

---
