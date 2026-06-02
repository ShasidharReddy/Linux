# Databases on Linux


---

## 🎬 Database Operations — Animated Workflow

```mermaid
graph TD
    subgraph QUERY["📊 Query Lifecycle"]
        direction TB
        Q1["📝 SQL Query"]:::query --> Q2["🔍 Parser"]:::parse
        Q2 --> Q3["📋 Query Planner"]:::plan
        Q3 --> Q4["⚡ Optimizer"]:::optimize
        Q4 --> Q5["🔧 Executor"]:::exec
        Q5 --> Q6["💾 Storage Engine"]:::storage
        Q6 --> Q7["📤 Result Set"]:::result
    end

    subgraph REPLICATION["🔄 Replication Flow"]
        direction LR
        M["🟢 Primary"]:::primary --> |"WAL/Binlog"| R1["🔵 Replica 1"]:::replica
        M --> |"WAL/Binlog"| R2["🔵 Replica 2"]:::replica
        R1 --> |"Read Queries"| APP1["⚙️ App"]:::app
        R2 --> |"Read Queries"| APP2["⚙️ App"]:::app
        APP1 --> |"Write Queries"| M
    end

    subgraph BACKUP_DB["💾 Backup Strategy"]
        direction LR
        BK1["📊 pg_dump / mysqldump"]:::dump --> BK2["🗜️ Compress"]:::compress
        BK2 --> BK3["☁️ Remote Storage"]:::remote
        BK3 --> BK4["✅ Verified"]:::verify
    end

    classDef query fill:#2196F3,stroke:#1565C0,color:#fff
    classDef parse fill:#FF9800,stroke:#E65100,color:#fff
    classDef plan fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef optimize fill:#e94560,stroke:#b71c1c,color:#fff
    classDef exec fill:#00BCD4,stroke:#00838F,color:#fff
    classDef storage fill:#607D8B,stroke:#37474F,color:#fff
    classDef result fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef primary fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
    classDef replica fill:#2196F3,stroke:#1565C0,color:#fff
    classDef app fill:#FF9800,stroke:#E65100,color:#fff
    classDef dump fill:#0f3460,stroke:#16213e,color:#fff
    classDef compress fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef remote fill:#00BCD4,stroke:#00838F,color:#fff
    classDef verify fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:2px
```

---

This guide has been split into smaller, topic-focused files for easier navigation.

## Table of Contents

1. [Fundamentals](01-fundamentals.md) — RDBMS vs NoSQL, ACID, CAP, architecture patterns, and stack-selection examples
2. [MySQL / MariaDB](02-mysql-mariadb.md) — Install, config, users, replication, tuning, backup, HA, and command cookbook
3. [PostgreSQL](03-postgresql.md) — Install, config, replication, tuning, backup, extensions, HA, and command cookbook
4. [MongoDB](04-mongodb.md) — Install, CRUD, replica sets, sharding, backup, and operations guidance
5. [Redis](05-redis.md) — Install, data types, persistence, replication, Sentinel, Cluster, and operations guidance
6. [Elasticsearch](06-elasticsearch.md) — Install, indexes, mapping, queries, cluster operations, snapshots, and ELK/EFK guidance
7. [SQLite](07-sqlite.md) — When to use it, CLI basics, backup, WAL mode, and performance tips
8. [Administration](08-administration.md) — Monitoring, pooling, migrations, backups, performance, Linux playbooks, and operational references
9. [Security](09-security.md) — Auth, encryption, secret handling, SQL injection prevention, and audit guidance
10. [Containers](10-containers.md) — Docker, Compose, Kubernetes StatefulSets, operators, and container backup concerns
11. [Troubleshooting](11-troubleshooting.md) — Connection issues, slow queries, replication lag, corruption recovery, and runbook summaries
