# Database Administration

← Back to [12-database-essentials.md](./12-database-essentials.md)

Backup, restore, monitoring, replication, security, and day-2 database operations.

---

## 5. Database Administration Essentials

Administration starts long before the first incident and continues long after the first deployment.

### 5.1 Creating databases, users, and permissions

#### 5.1.1 MySQL / MariaDB

```sql
CREATE DATABASE appdb;
CREATE USER 'appuser'@'10.10.%' IDENTIFIED BY 'StrongPass!';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, ALTER
ON appdb.* TO 'appuser'@'10.10.%';
FLUSH PRIVILEGES;
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

#### 5.1.2 PostgreSQL

```sql
CREATE DATABASE appdb;
CREATE ROLE appuser WITH LOGIN PASSWORD 'StrongPass!';
GRANT CONNECT ON DATABASE appdb TO appuser;
\c appdb
GRANT USAGE ON SCHEMA public TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO appuser;
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

#### 5.1.3 MongoDB

```javascript
use appdb
db.createUser({
  user: "appuser",
  pwd: "StrongPass!",
  roles: [
    { role: "readWrite", db: "appdb" }
  ]
})
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

#### 5.1.4 Redis

```bash
ACL SETUSER appuser on >StrongPass! ~app:* +@read +@write +@hash +@string
ACL LIST
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

#### 5.1.5 Elasticsearch

```bash
curl -u elastic:password -X PUT https://localhost:9200/app-logs \
  -H "Content-Type: application/json" \
  -d '{
    "settings": {"number_of_shards": 3, "number_of_replicas": 1}
  }'

# Create a role and user with Kibana or the security APIs
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

#### 5.1.6 SQLite

```bash
sqlite3 app.db <<'SQL'
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
SQL
```

- Grant only what the application needs.
- Keep admin accounts separate from service accounts.
- Document ownership of each database, schema, or collection.

### 5.2 Backup and restore by engine

#### 5.2.1 MySQL / MariaDB backup

```bash
mysqldump --single-transaction --routines --triggers appdb > appdb.sql
# Compressed backup
mysqldump --single-transaction appdb | gzip > appdb.sql.gz
```

##### Restore

```bash
mysql -e "CREATE DATABASE IF NOT EXISTS appdb;"
mysql appdb < appdb.sql
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

#### 5.2.2 PostgreSQL backup

```bash
pg_dump -Fc appdb > appdb.dump
pg_dump -Fd appdb -f appdb_dir_backup
```

##### Restore

```bash
createdb appdb_restore
pg_restore -d appdb_restore appdb.dump
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

#### 5.2.3 MongoDB backup

```bash
mongodump --uri "mongodb://backup:password@localhost:27017/admin" --archive=appdb.archive --gzip
```

##### Restore

```bash
mongorestore --uri "mongodb://restore:password@localhost:27017/admin" --archive=appdb.archive --gzip --nsInclude="appdb.*"
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

#### 5.2.4 Redis backup

```bash
redis-cli BGSAVE
# Copy dump.rdb and/or appendonly.aof from the data directory after save completes
```

##### Restore

```bash
sudo systemctl stop redis-server
# Replace dump.rdb or appendonly.aof in the Redis data directory
sudo systemctl start redis-server
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

#### 5.2.5 Elasticsearch backup

```bash
curl -u elastic:password -X PUT https://localhost:9200/_snapshot/fsrepo \
  -H "Content-Type: application/json" \
  -d '{"type":"fs","settings":{"location":"/var/backups/elasticsearch"}}'

curl -u elastic:password -X PUT https://localhost:9200/_snapshot/fsrepo/snap-001?wait_for_completion=true
```

##### Restore

```bash
curl -u elastic:password -X POST https://localhost:9200/_snapshot/fsrepo/snap-001/_restore \
  -H "Content-Type: application/json" \
  -d '{"indices":"app-logs"}'
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

#### 5.2.6 SQLite backup

```bash
sqlite3 app.db ".backup app-backup.db"
```

##### Restore

```bash
cp app-backup.db app.db
```

**Backup note:** test restores regularly; a backup that cannot be restored is only a theory.

### 5.3 Monitoring and performance essentials

Monitor the host, the database, and the workload together.

| Category | Examples |
|---|---|
| Host metrics | CPU, memory, disk IOPS, latency, network retransmits |
| Database metrics | Connections, buffer usage, locks, replication lag, query latency |
| Business metrics | Orders per minute, login latency, failed checkouts, queue depth |
| Protection metrics | Backup success rate, restore test age, certificate expiry, audit log volume |

#### 5.3.1 MySQL / MariaDB monitoring commands

```bash
mysqladmin ping
mysqladmin extended-status
mysql -e "SHOW PROCESSLIST;"
mysql -e "SHOW ENGINE INNODB STATUS\G"
```

#### 5.3.2 PostgreSQL monitoring commands

```bash
pg_isready
psql -c "SELECT version();"
psql -c "SELECT * FROM pg_stat_activity;"
psql -c "SELECT * FROM pg_stat_replication;"
```

#### 5.3.3 MongoDB monitoring commands

```bash
mongosh --eval "db.serverStatus()"
mongosh --eval "db.adminCommand({ replSetGetStatus: 1 })"
mongosh --eval "db.currentOp()"
```

#### 5.3.4 Redis monitoring commands

```bash
redis-cli INFO memory
redis-cli INFO stats
redis-cli LATENCY DOCTOR
redis-cli SLOWLOG GET 10
```

#### 5.3.5 Elasticsearch monitoring commands

```bash
curl -u elastic:password https://localhost:9200/_cluster/health?pretty
curl -u elastic:password https://localhost:9200/_cat/nodes?v
curl -u elastic:password https://localhost:9200/_cat/shards?v
```

#### 5.3.6 SQLite monitoring commands

```bash
sqlite3 app.db "PRAGMA integrity_check;"
sqlite3 app.db "EXPLAIN QUERY PLAN SELECT * FROM users WHERE username = 'alice';"
```

### 5.4 Replication setup basics

#### 5.4.1 MySQL / MariaDB

1. Enable binary logging.
2. Create a replication user.
3. Take a consistent snapshot or use clone tooling.
4. Point replicas at the primary using GTID when possible.
5. Monitor replica lag and IO/SQL thread health.

#### 5.4.2 PostgreSQL

1. Set `wal_level = replica`.
2. Increase `max_wal_senders` and `max_replication_slots` as required.
3. Use `pg_basebackup` to seed a standby.
4. Create a replication role.
5. Monitor replay lag and replication slots.

#### 5.4.3 MongoDB

1. Deploy at least three voting members.
2. Initialize with `rs.initiate()`.
3. Add members with `rs.add()`.
4. Set priorities so the correct node becomes primary.
5. Monitor oplog window and replication lag.

#### 5.4.4 Redis

1. Configure `replicaof <primary-host> <primary-port>` on replicas.
2. Secure replication traffic with ACLs and TLS.
3. Use Sentinel for primary election and service discovery.
4. Use Redis Cluster when partitioning across shards is required.

#### 5.4.5 Elasticsearch

1. Replication happens through primary and replica shards.
2. Every index can define replica count.
3. Cluster health depends on shard placement and quorum-capable master nodes.
4. Snapshots remain essential because replicas are not backups.

#### 5.4.6 SQLite

1. SQLite has no native built-in multi-node replication like server databases.
2. HA is usually implemented at the application, filesystem, or synchronization layer.
3. When HA becomes important, teams often graduate to PostgreSQL or MySQL.

### 5.5 High availability patterns

#### 5.5.1 MySQL / MariaDB

- Primary plus one or more replicas for read scaling.
- Semi-synchronous replication when stronger durability is required.
- Managed failover using Orchestrator, MHA, or external automation.

#### 5.5.2 PostgreSQL

- Primary plus streaming replicas.
- Patroni or repmgr for automated failover.
- Synchronous standby for critical write durability when latency budget allows.

#### 5.5.3 MongoDB

- Replica set with one primary and multiple secondaries.
- Add an arbiter only when absolutely necessary; data-bearing nodes are preferred.
- Sharding adds routers and config servers for very large scale-out needs.

#### 5.5.4 Redis

- Primary plus replica plus Sentinel for small HA setups.
- Redis Cluster for sharded HA.
- Managed services are popular because stateful failover is operationally sensitive.

#### 5.5.5 Elasticsearch

- At least three master-eligible nodes for quorum in production.
- Multiple data nodes with replica shards for resilience.
- Hot-warm-cold architectures for cost-aware log retention.

#### 5.5.6 SQLite

- Use app-level failover to a replicated file if absolutely necessary.
- Keep writes small and transactional.
- Prefer server databases for multi-user HA workloads.

### 5.6 HA architecture diagrams

#### Relational primary / replica with pooler
```mermaid
flowchart TD
    Apps[Application pods] --> Pool[Connection pooler]
    Pool --> Primary[(Primary RDBMS)]
    Primary --> Replica1[(Replica 1)]
    Primary --> Replica2[(Replica 2)]
    Monitor[Monitoring] --> Primary
    Backup[Backup job] --> Primary
    VIP[Failover VIP / DNS] --> Pool
```

#### MongoDB replica set
```mermaid
flowchart LR
    C1[Client] --> M1[(Primary)]
    C1 --> M2[(Secondary)]
    C1 --> M3[(Secondary)]
    M1 -->|Oplog| M2
    M1 -->|Oplog| M3
    A[Arbiter optional] -. votes .-> M1
    A -. votes .-> M2
    A -. votes .-> M3
```

#### Distributed SQL / search cluster quorum pattern
```mermaid
flowchart LR
    App[Applications] --> LB[Load balancer]
    LB --> N1[Node 1]
    LB --> N2[Node 2]
    LB --> N3[Node 3]
    N1 --- N2
    N2 --- N3
    N1 --- N3
    Q[Quorum required for leadership] -.-> N2
```

### 5.7 Performance tuning principles

- Measure before tuning: capture latency, throughput, concurrency, and plan details.
- Add indexes for read paths, but remember that every index adds write overhead.
- Right-size memory and avoid swapping.
- Use fast storage for WAL, redo logs, transaction logs, and hot working sets.
- Prefer prepared statements and bounded result sets.
- Archive or partition cold data rather than forcing hot tables or collections to grow forever.
- Rehearse failover and restore procedures so performance tuning does not break resilience.

### 5.8 Backup strategy design

1. Define recovery point objective (RPO) and recovery time objective (RTO).
2. Choose logical, physical, snapshot, or API-driven backups depending on engine and scale.
3. Store backups off-host and off-zone.
4. Encrypt backups at rest and in transit.
5. Automate retention and rotation.
6. Test restore weekly or monthly based on criticality.
7. Document the exact restore steps and ownership.

### 5.9 Capacity planning

- Project data growth per day, per week, and per retention period.
- Track connection counts and peak concurrency.
- Estimate index growth separately from base data growth.
- Watch compaction, vacuum, or segment merge behavior because overhead can exceed raw data size.
- Reserve room for backups, replicas, and temporary maintenance operations.

---

## 6. Database Security

### 6.1 Authentication methods

| Database | Common methods | Notes |
|---|---|---|
| MySQL / MariaDB | Native password auth, socket auth, TLS client certs | Separate admin and app accounts |
| PostgreSQL | SCRAM-SHA-256, peer, cert auth, GSSAPI in some environments | Prefer SCRAM for password auth |
| MongoDB | SCRAM, X.509, LDAP in enterprise setups | Enable authorization explicitly |
| Redis | ACL users, passwords, TLS | Do not rely on network secrecy alone |
| Elasticsearch | Built-in users, API keys, SSO, TLS certs | Security features should be enabled early |
| SQLite | OS file permissions and app-layer controls | No server-side user model |

### 6.2 Encryption at rest

- Use full-disk or filesystem encryption for self-managed Linux hosts.
- Prefer cloud block-storage encryption where applicable.
- Protect backup repositories with encryption as well.
- Remember that encryption at rest does not replace access control or audit logging.

### 6.3 Encryption in transit

- Use TLS for all client-to-database traffic that leaves localhost.
- Use TLS for database-to-database replication links when supported.
- Rotate certificates before they expire and alert on upcoming expiry.
- Pin trusted CAs in automation and application clients when possible.

### 6.4 SQL injection prevention

Database security is not only a DBA concern; application query construction is part of the attack surface.

**Unsafe pattern:**
```python
# Bad: user input concatenated into SQL
query = f"SELECT * FROM users WHERE email = '{email}'"
cursor.execute(query)
```

**Safer pattern:**
```python
# Good: parameterized query
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

**Rules to follow:**
- Use parameterized queries or prepared statements.
- Avoid dynamic SQL unless identifiers are strictly allow-listed.
- Grant least privilege so application users cannot drop schemas or create superusers.
- Log rejected or suspicious input at the application boundary.

### 6.5 Audit logging

- Record login successes and failures.
- Track DDL changes such as CREATE, ALTER, and DROP.
- Audit high-risk operations such as privilege escalation and backup export.
- Forward database logs to a central log platform with retention and access controls.

### 6.6 Principle of least privilege

1. Create one database user per application or service.
2. Grant only required schemas, tables, collections, or key prefixes.
3. Split read-only, read-write, and admin roles.
4. Use short-lived break-glass credentials for emergency admin access.
5. Review privileges quarterly or after major architecture changes.

### 6.7 Security hardening checklist

- Default accounts rotated or disabled.
- Public internet exposure avoided or heavily restricted.
- Backups encrypted and access reviewed.
- TLS enforced where supported.
- Secrets stored outside code repositories.
- Audit and error logs centralized.
- Restore procedures tested without using production credentials casually.

---

## 8. Quick Reference Tables

### 8.1 Default ports

| Engine | Port(s) | Protocol / note |
|---|---|---|
| MySQL | 3306 | SQL over TCP |
| MariaDB | 3306 | MySQL-compatible wire protocol |
| PostgreSQL | 5432 | SQL over TCP |
| MongoDB | 27017 | Mongo wire protocol |
| Redis | 6379 | RESP over TCP |
| Elasticsearch HTTP | 9200 | REST/HTTP |
| Elasticsearch transport | 9300 | Cluster transport |
| CockroachDB SQL | 26257 | PostgreSQL-style SQL port |
| CockroachDB Admin UI | 8080 | Web UI |
| TiDB | 4000 | MySQL protocol |
| TiDB status | 10080 | HTTP status/API |
| Cassandra CQL | 9042 | CQL native transport |
| Neo4j Bolt | 7687 | Primary client protocol |
| Neo4j HTTP | 7474 | HTTP UI / API |
| InfluxDB | 8086 | HTTP API |
| Solr | 8983 | HTTP API |
| SQLite | N/A | Embedded file access only |

### 8.2 Common commands cheat sheet

| Task | MySQL/MariaDB | PostgreSQL | MongoDB | Redis | Elasticsearch | SQLite |
|---|---|---|---|---|---|---|
| Open local shell | `mysql -u root -p` | `sudo -u postgres psql` | `mongosh` | `redis-cli` | `curl http://localhost:9200` | `sqlite3 app.db` |
| List databases | `SHOW DATABASES;` | `\l` | `show dbs` | `INFO keyspace` | `GET /_cat/indices?v` | `.databases` or inspect files |
| Check connectivity | `mysqladmin ping` | `pg_isready` | `db.runCommand({ ping: 1 })` | `PING` | `GET /` or `_cluster/health` | `SELECT 1;` |
| Backup | `mysqldump` | `pg_dump` | `mongodump` | `BGSAVE` | `_snapshot` API | `.backup` |
| Restore | `mysql < backup.sql` | `pg_restore` | `mongorestore` | Replace RDB/AOF then restart | `_restore` API | Copy backup file into place |

### 8.3 Connection string templates

| Engine | Template |
|---|---|
| MySQL | `mysql://USER:PASSWORD@HOST:3306/DBNAME` |
| PostgreSQL | `postgresql://USER:PASSWORD@HOST:5432/DBNAME` |
| MongoDB | `mongodb://USER:PASSWORD@HOST:27017/DBNAME?authSource=admin` |
| MongoDB Atlas / SRV | `mongodb+srv://USER:PASSWORD@CLUSTER/DBNAME` |
| Redis | `redis://:PASSWORD@HOST:6379/0` |
| Redis TLS | `rediss://:PASSWORD@HOST:6380/0` |
| Elasticsearch | `https://USER:PASSWORD@HOST:9200` |
| SQLite | `sqlite:///absolute/path/to/app.db` |
| CockroachDB | `postgresql://USER:PASSWORD@HOST:26257/DBNAME?sslmode=require` |
| TiDB | `mysql://USER:PASSWORD@HOST:4000/DBNAME` |
| Cassandra | `cassandra://HOST:9042/KEYSPACE` (driver-specific) |
| Neo4j | `neo4j+s://USER:PASSWORD@HOST:7687` |
| InfluxDB | `http://HOST:8086` with token and org |
| Solr | `http://HOST:8983/solr/COLLECTION` |

### 8.4 Emergency triage checklist

1. Can clients still connect?
2. Is the host healthy: CPU, memory, disk, network?
3. Is replication healthy or lagging?
4. Did a recent schema, config, or application deployment change behavior?
5. Do you have a recent verified backup and a tested restore path?
6. Is the problem isolated to one shard, one replica, or one availability zone?
7. Do logs show authentication failures, lock waits, OOM kills, or disk-full conditions?

---

## 9. Real-World Scenarios

### 9.1 Online banking ledger

**Recommended choice:** PostgreSQL or a strongly consistent distributed SQL system

**Why:** Transactions, integrity, and auditability dominate all other concerns.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.2 E-commerce product catalog

**Recommended choice:** MongoDB plus PostgreSQL for orders

**Why:** Product attributes vary, but payments and orders need ACID semantics.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.3 API response caching

**Recommended choice:** Redis

**Why:** Reads are frequent, values are short-lived, and latency matters more than ad hoc queries.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.4 Centralized logging platform

**Recommended choice:** Elasticsearch

**Why:** Full-text search, faceting, and time-filtered analysis are critical.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.5 IoT metrics pipeline

**Recommended choice:** InfluxDB or TimescaleDB

**Why:** Data arrives by timestamp and is queried by time windows and retention periods.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.6 Graph fraud detection

**Recommended choice:** Neo4j

**Why:** Relationships and traversals are the first-class query primitive.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.7 Global SaaS control plane

**Recommended choice:** CockroachDB or TiDB

**Why:** The workload wants SQL plus geographic distribution.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.8 Developer laptop app

**Recommended choice:** SQLite

**Why:** A local file is simpler than a server process and good enough for modest concurrency.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.9 High-write event ingestion

**Recommended choice:** Cassandra

**Why:** Partitioned write-heavy workloads scale well with wide-column design.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.10 Search box for product discovery

**Recommended choice:** Elasticsearch or Solr

**Why:** Relevance scoring and stemming matter more than transactional updates.

- Start small but design for backup and monitoring on day one.
- Use the database for its strengths instead of forcing every workload into a single engine.
- Document the failure mode you are willing to accept: stale reads, delayed writes, or temporary unavailability.

### 9.11 Example architecture selection workflow

```mermaid
flowchart TD
    Req[Business requirement] --> Data{Data shape?}
    Data -->|Relational| Tx{Need strict ACID?}
    Data -->|Document| Doc[MongoDB]
    Data -->|Search| Search[Elasticsearch / Solr]
    Data -->|Cache| Cache[Redis]
    Tx -->|Yes| Scale{Need cross-node scale?}
    Tx -->|No| Simple[Standard RDBMS still often works]
    Scale -->|No| Pg[PostgreSQL / MySQL]
    Scale -->|Yes| DistSQL[CockroachDB / TiDB]
```

---

## 10. Checklists and Common Mistakes

### 10.1 Day-0 deployment checklist

- Choose the database based on access pattern, not hype.
- Create a private network path before enabling remote clients.
- Define backup frequency, retention, and restore ownership.
- Create non-admin application users.
- Enable monitoring and log shipping before go-live.
- Document config file locations and service management commands.
- Test restart, restore, and failover behavior in a safe environment.

### 10.2 Per-engine hardening checklist

#### 10.2.1 MySQL / MariaDB

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

#### 10.2.2 PostgreSQL

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

#### 10.2.3 MongoDB

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

#### 10.2.4 Redis

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

#### 10.2.5 Elasticsearch

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

#### 10.2.6 SQLite

- Confirm the service binds only to approved interfaces.
- Create named service users instead of sharing administrator credentials.
- Restrict inbound firewall rules to explicit source CIDRs.
- Enable TLS or a secure private transport path.
- Run a backup and a restore test before production cutover.
- Add monitoring for connection failures, capacity, and latency.
- Write down the first three incident-response commands operators should run.

### 10.3 Common mistakes

1. Leaving the database bound to all interfaces without a firewall.
2. Letting application code use the root, postgres, or elastic superuser account.
3. Treating replicas as backups.
4. Skipping restore tests because the backup job “succeeded.”
5. Ignoring disk latency and focusing only on CPU.
6. Assuming container persistence exists without checking volume mounts.
7. Using one database for every workload even when search, cache, and time-series needs are distinct.
8. Opening Redis publicly because it seems lightweight.
9. Forgetting to tune connection limits when the application scales out.
10. Allowing schema drift or ad hoc production DDL without review.
11. Running Elasticsearch with production data but development-grade single-node settings.
12. Assuming SQLite can safely handle every multi-writer server workload.
13. Choosing Cassandra before understanding query-first schema design.
14. Failing to rotate passwords, API keys, or certificates.
15. Ignoring audit trails for privileged operations.

### 10.4 Final advice

- A simple well-operated database is usually better than a sophisticated poorly operated one.
- Pick the smallest architecture that honestly meets your availability, scale, and correctness needs.
- Invest in backups, monitoring, security, and rehearsed recovery as early as you invest in schema design.
- Use this guide as a launch point, then go deeper into engine-specific documentation for production rollouts.
