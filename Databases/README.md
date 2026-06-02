# Databases on Linux

A production-focused guide to administering databases on Linux systems. It covers relational, document, key-value, search, and embedded databases with practical commands, architecture patterns, performance guidance, backup strategies, security controls, container deployments, and troubleshooting playbooks.

> Scope: Linux-first operations for self-managed, VM-based, and containerized environments.
> Audience: Sysadmins, SREs, DBAs, backend engineers, and platform teams.
> Distros: Ubuntu/Debian and RHEL/CentOS/Alma/Rocky examples are included where practical.

---

## Table of Contents

1. [Database Fundamentals](#1-database-fundamentals)
2. [MySQL / MariaDB](#2-mysql--mariadb)
3. [PostgreSQL](#3-postgresql)
4. [MongoDB](#4-mongodb)
5. [Redis](#5-redis)
6. [Elasticsearch](#6-elasticsearch)
7. [SQLite](#7-sqlite)
8. [Database Administration Tasks](#8-database-administration-tasks)
9. [Database Security](#9-database-security)
10. [Database in Containers](#10-database-in-containers)
11. [Troubleshooting](#11-troubleshooting)
12. [Appendix: Linux DBA Command Reference](#12-appendix-linux-dba-command-reference)

---

# 1. Database Fundamentals

## 1.1 What a database administrator does on Linux

A Linux database administrator typically owns or contributes to:

- Package installation and lifecycle management.
- Service startup, shutdown, restart, and boot persistence.
- Disk and filesystem sizing.
- User and role administration.
- Schema and data change governance.
- Backup and restore validation.
- Replication and high availability operations.
- Performance tuning and capacity planning.
- Security hardening and auditability.
- Incident response and recovery.

## 1.2 Core database categories

### RDBMS

Relational database management systems store data in tables with rows and columns and enforce schemas.

Common strengths:

- Strong consistency.
- Rich SQL support.
- Mature transactions.
- Powerful joins and aggregations.
- Referential integrity.

Common workloads:

- OLTP systems.
- ERP and CRM platforms.
- Financial systems.
- Inventory and order management.
- Multi-table reporting.

Examples:

- MySQL
- MariaDB
- PostgreSQL
- SQLite

### NoSQL

NoSQL is a broad class that includes document, key-value, wide-column, and graph databases.

Common strengths:

- Flexible schema models.
- Horizontal scale-out patterns.
- High write throughput in many designs.
- Natural fit for semi-structured data.

Common workloads:

- Content management.
- User profiles and event storage.
- Caching.
- Session management.
- Search and observability pipelines.

Examples:

- MongoDB
- Redis
- Elasticsearch
- Cassandra

### NewSQL

NewSQL systems aim to preserve SQL and ACID properties while scaling horizontally like distributed systems.

Common strengths:

- SQL compatibility.
- Strong transactional guarantees.
- Distributed architecture.
- Online scale-out.

Common workloads:

- Cloud-native transactional platforms.
- Global SaaS applications.
- High scale services needing SQL semantics.

Examples:

- CockroachDB
- YugabyteDB
- TiDB
- SingleStore

## 1.3 RDBMS vs NoSQL vs NewSQL

| Characteristic | RDBMS | NoSQL | NewSQL |
|---|---|---|---|
| Data model | Tables | Flexible or specialized | Relational |
| Schema | Usually strict | Often flexible | Strict relational |
| Transactions | Strong ACID | Varies by engine | Strong ACID |
| Scaling | Vertical first, some horizontal | Horizontal first | Horizontal with SQL |
| Query language | SQL | Engine-specific APIs/query DSLs | SQL |
| Joins | Excellent | Often limited or avoided | Supported |
| Operational complexity | Moderate | Moderate to high | Often high |
| Best fit | Structured transactional data | Semi-structured or extreme scale patterns | Distributed SQL workloads |

## 1.4 ACID properties

ACID defines transaction guarantees.

### Atomicity

A transaction succeeds entirely or fails entirely.

Example:

- Debit one account.
- Credit another account.
- If the credit fails, the debit must roll back.

### Consistency

A transaction moves the database from one valid state to another valid state.

Examples:

- Foreign keys remain valid.
- Check constraints remain satisfied.
- Unique keys remain unique.

### Isolation

Concurrent transactions should not interfere in invalid ways.

Common isolation levels:

| Isolation level | Prevents | Typical caveat |
|---|---|---|
| Read Uncommitted | Almost nothing | Dirty reads possible |
| Read Committed | Dirty reads | Non-repeatable reads possible |
| Repeatable Read | Dirty and non-repeatable reads | Phantom reads may still occur depending on engine |
| Serializable | Most anomalies | Reduced concurrency |

### Durability

Once committed, data survives crashes according to the engine's durability design.

Typical durability mechanisms:

- Write-ahead logging.
- Redo logs.
- fsync or equivalent flush behavior.
- Replication to secondary nodes.
- Snapshot and archival backups.

## 1.5 CAP theorem

CAP discusses trade-offs in distributed systems.

- Consistency: every read gets the latest write.
- Availability: every request receives a response.
- Partition Tolerance: the system continues despite network partitions.

Because partitions are unavoidable in distributed systems, designers usually trade between:

- CP systems: favor consistency over availability.
- AP systems: favor availability over immediate consistency.

### Practical CAP interpretation

- PostgreSQL single primary replication tends to prioritize consistency.
- Cassandra-like systems often emphasize availability with tunable consistency.
- MongoDB replica sets can vary behavior depending on read/write concerns.
- Redis Sentinel deployments often favor quick failover with application-level tolerance considerations.

## 1.6 OLTP vs OLAP

| Dimension | OLTP | OLAP |
|---|---|---|
| Goal | Fast transactions | Analytical queries |
| Query pattern | Small reads/writes | Large scans/aggregations |
| Data freshness | Real-time | Near real-time or batch |
| Schema style | Normalized | Often denormalized/star schema |
| Example engines | MySQL, PostgreSQL | ClickHouse, Redshift, BigQuery |

## 1.7 Row store vs column store

| Model | Best for | Example |
|---|---|---|
| Row store | Frequent small transactions | MySQL, PostgreSQL |
| Column store | Large analytics scans | ClickHouse, Parquet-based engines |

## 1.8 Database selection decision tree

Use the following decision process when choosing a database.

1. Do you need multi-row ACID transactions and joins?
   - Yes: choose an RDBMS or NewSQL.
   - No: continue.
2. Do you need flexible JSON-like documents with variable fields?
   - Yes: consider MongoDB.
   - No: continue.
3. Do you need sub-millisecond caching, counters, or ephemeral state?
   - Yes: consider Redis.
   - No: continue.
4. Do you need full-text search, log analytics, or inverted indexing?
   - Yes: consider Elasticsearch.
   - No: continue.
5. Do you need embedded local storage with zero server management?
   - Yes: consider SQLite.
   - No: continue.
6. Do you need distributed SQL with horizontal scale and ACID?
   - Yes: consider a NewSQL engine.
   - No: a traditional RDBMS is usually simpler.

## 1.9 Decision table

| Requirement | Recommended starting point |
|---|---|
| Web application with transactions | PostgreSQL or MySQL |
| GIS and geospatial | PostgreSQL + PostGIS |
| Embedded app or edge device | SQLite |
| Flexible product catalog documents | MongoDB |
| Cache and sessions | Redis |
| Log search and observability | Elasticsearch |
| Distributed SQL at scale | NewSQL platform |

## 1.10 Mermaid diagram: Database types and use cases

```mermaid
graph TD
    A["Database Families"] --> B["RDBMS<br/>ACID transactions<br/>joins<br/>structured schema"]
    A --> C["NoSQL<br/>flexible schema<br/>horizontal scale"]
    A --> D["NewSQL<br/>distributed SQL<br/>ACID at scale"]
    B --> B1["MySQL<br/>web apps<br/>OLTP"]
    B --> B2["PostgreSQL<br/>advanced SQL<br/>analytics<br/>GIS"]
    B --> B3["SQLite<br/>embedded apps<br/>single-file DB"]
    C --> C1["MongoDB<br/>documents<br/>content and profiles"]
    C --> C2["Redis<br/>cache<br/>sessions<br/>queues"]
    C --> C3["Elasticsearch<br/>search<br/>logs<br/>metrics"]
    D --> D1["Distributed SQL<br/>global SaaS<br/>scale-out OLTP"]
```

## 1.11 Storage engine concepts

Important terms across database engines:

- Page: fixed-size unit of disk I/O.
- Buffer cache: in-memory pages used to reduce disk reads.
- WAL or redo log: sequential log for durability and crash recovery.
- Checkpoint: point where dirty pages are flushed and recovery work is reduced.
- Vacuum or purge: cleanup of obsolete versions or undo data.
- Index: auxiliary structure to accelerate lookups.

## 1.12 Linux fundamentals that affect databases

### CPU

Database workloads are sensitive to:

- Single-thread performance for some query types.
- Core count for concurrency.
- NUMA behavior on large servers.

Useful commands:

```bash
lscpu
numactl --hardware
mpstat -P ALL 1
```

### Memory

Memory determines cache effectiveness and sort/hash working space.

Useful commands:

```bash
free -h
vmstat 1
cat /proc/meminfo | head
```

### Disk and filesystem

Database performance depends heavily on latency, throughput, queue depth, and fsync characteristics.

Useful commands:

```bash
lsblk
fio --name=randread --filename=/data/testfile --size=1G --bs=4k --rw=randread --iodepth=32
iostat -xz 1
```

Recommended practices:

- Use SSD/NVMe for primary data paths.
- Separate data, logs, and backups where justified.
- Monitor inode usage and free space.
- Choose filesystem options carefully.

### Network

Replication, clustering, and client access depend on stable latency.

Useful commands:

```bash
ss -tulpn
ip addr
ethtool eth0
sar -n DEV 1
```

## 1.13 Filesystem layout examples

| Component | Common mount |
|---|---|
| Database data | /var/lib/mysql or /var/lib/pgsql or /var/lib/mongodb |
| Logs | /var/log/mysql or journald or /var/log/postgresql |
| Backups | /backup or mounted object-storage gateway |
| WAL/binlogs | Dedicated fast volume in larger deployments |

## 1.14 Service management with systemd

Common commands:

```bash
sudo systemctl status mysql
sudo systemctl restart postgresql
sudo systemctl enable mongod
sudo journalctl -u redis -n 200 --no-pager
```

## 1.15 Packaging models

You may install databases using:

- Native distro repositories.
- Vendor repositories.
- Tarball or binary packages.
- Container images.
- Operators in Kubernetes.

General advice:

- Prefer vendor repositories for production features and patch cadence.
- Pin versions intentionally.
- Coordinate upgrades with backup verification and rollback plans.

---

# 2. MySQL / MariaDB

## 2.1 Overview

MySQL and MariaDB are popular relational databases for web applications and general OLTP workloads.

Common Linux service names:

- mysql
- mysqld
- mariadb

Default data paths commonly include:

- /var/lib/mysql
- /etc/mysql/
- /etc/my.cnf
- /etc/mysql/my.cnf

## 2.2 Installation on Ubuntu

### MySQL

```bash
sudo apt update
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
sudo mysql_secure_installation
```

### MariaDB

```bash
sudo apt update
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

## 2.3 Installation on CentOS/RHEL/Alma/Rocky

### MySQL using vendor repo

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
sudo dnf module disable -y mysql
sudo dnf install -y mysql-community-server
sudo systemctl enable --now mysqld
sudo grep 'temporary password' /var/log/mysqld.log
```

### MariaDB using distro or vendor repo

```bash
sudo dnf install -y mariadb-server
sudo systemctl enable --now mariadb
```

## 2.4 Post-install verification

```bash
systemctl status mysql --no-pager
ss -tulpn | grep 3306
mysql -uroot -p -e 'SELECT VERSION();'
```

## 2.5 Important directories and files

| Purpose | Common path |
|---|---|
| Main config | /etc/mysql/my.cnf or /etc/my.cnf |
| Additional config | /etc/mysql/conf.d/ or /etc/my.cnf.d/ |
| Data directory | /var/lib/mysql |
| Socket | /var/run/mysqld/mysqld.sock |
| Error log | journald or /var/log/mysql/error.log |

## 2.6 Configuration basics

Example `/etc/mysql/my.cnf` fragment:

```ini
[mysqld]
user = mysql
bind-address = 0.0.0.0
port = 3306
socket = /var/run/mysqld/mysqld.sock
pid-file = /run/mysqld/mysqld.pid
datadir = /var/lib/mysql
max_connections = 300
max_connect_errors = 1000
skip_name_resolve = 1
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
innodb_buffer_pool_size = 8G
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table = 1
tmp_table_size = 256M
max_heap_table_size = 256M
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 2M
read_rnd_buffer_size = 4M
thread_cache_size = 100
table_open_cache = 4000
open_files_limit = 65535
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
log_error_verbosity = 2
log_bin = /var/lib/mysql/mysql-bin
binlog_format = ROW
server_id = 1
expire_logs_days = 7
```

## 2.7 Key MySQL parameters explained

| Parameter | Purpose | General guidance |
|---|---|---|
| innodb_buffer_pool_size | Main InnoDB cache | 50 to 75 percent of RAM on dedicated servers |
| max_connections | Concurrent sessions limit | Size conservatively; too high causes memory pressure |
| query_cache_size | Legacy query cache | Disable on modern MySQL; MariaDB usage depends on workload |
| tmp_table_size | In-memory temp tables | Increase for sorts/grouping if justified |
| table_open_cache | Open table descriptors | Increase for many active tables |
| innodb_flush_log_at_trx_commit | Durability/performance trade-off | 1 for strongest durability |
| sync_binlog | Binlog durability | 1 for strict durability |
| long_query_time | Slow log threshold | Often 0.5 to 2 seconds |
| max_allowed_packet | Max packet size | Increase for large rows or bulk ops |
| innodb_log_file_size | Redo capacity | Larger values improve write-heavy workloads |

## 2.8 Query cache note

- MySQL 8 removed the query cache.
- MariaDB may still support it.
- In modern deployments, query cache is usually avoided because it can add contention and poor scalability.

## 2.9 Secure initial configuration

Checklist:

- Set strong root/admin authentication.
- Remove anonymous users.
- Remove test database.
- Restrict network exposure.
- Use TLS for remote clients.
- Use least-privilege accounts.
- Disable DNS lookups with `skip_name_resolve` when appropriate.

## 2.10 Basic service administration

```bash
sudo systemctl start mysql
sudo systemctl stop mysql
sudo systemctl restart mysql
sudo systemctl reload mysql
sudo systemctl enable mysql
sudo journalctl -u mysql -n 200 --no-pager
```

## 2.11 User management

### Create user

```sql
CREATE USER 'appuser'@'10.%' IDENTIFIED BY 'StrongPasswordHere';
```

### Grant privileges

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'appuser'@'10.%';
FLUSH PRIVILEGES;
```

### Revoke privileges

```sql
REVOKE DELETE ON appdb.* FROM 'appuser'@'10.%';
```

### Drop user

```sql
DROP USER 'appuser'@'10.%';
```

### Inspect grants

```sql
SHOW GRANTS FOR 'appuser'@'10.%';
```

## 2.12 Role-based access

In engines that support roles:

```sql
CREATE ROLE 'readonly_role';
GRANT SELECT ON reporting.* TO 'readonly_role';
GRANT 'readonly_role' TO 'reporter'@'%';
SET DEFAULT ROLE 'readonly_role' TO 'reporter'@'%';
```

## 2.13 Password and auth policies

Useful controls:

- Password validation plugin/component.
- Password expiration.
- Failed login lockout where supported.
- Auth plugins such as caching_sha2_password.

## 2.14 Database operations

### Create database

```sql
CREATE DATABASE appdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Create table

```sql
USE appdb;
CREATE TABLE customers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

### Show databases and tables

```sql
SHOW DATABASES;
SHOW TABLES FROM appdb;
```

## 2.15 Import and export basics

### Dump one database

```bash
mysqldump -u root -p --single-transaction --routines --triggers appdb > appdb.sql
```

### Dump all databases

```bash
mysqldump -u root -p --single-transaction --routines --triggers --events --all-databases > full.sql
```

### Restore

```bash
mysql -u root -p appdb < appdb.sql
```

### Compressed backup

```bash
mysqldump -u backup -p --single-transaction appdb | gzip > appdb-$(date +%F).sql.gz
```

### Restore compressed backup

```bash
gunzip -c appdb-2025-01-01.sql.gz | mysql -u root -p appdb
```

## 2.16 Binary log basics

Binary logs enable:

- Replication.
- Point-in-time recovery.
- Change data capture integrations.

Check status:

```sql
SHOW BINARY LOG STATUS;
SHOW BINARY LOGS;
```

## 2.17 Replication concepts

Replication modes include:

- Asynchronous replication.
- Semi-synchronous replication.
- Group replication.
- Galera synchronous certification model for MariaDB/MySQL-compatible clusters.

Key terms:

- Source or primary.
- Replica or secondary.
- Relay log.
- GTID.
- Binlog position.

## 2.18 Traditional source-replica setup

### Primary settings

```ini
[mysqld]
server_id = 1
log_bin = /var/lib/mysql/mysql-bin
binlog_format = ROW
gtid_mode = ON
enforce_gtid_consistency = ON
```

Create replication user:

```sql
CREATE USER 'repl'@'10.%' IDENTIFIED BY 'StrongReplicationPassword';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'10.%';
FLUSH PRIVILEGES;
```

Take a consistent backup and note GTID or binlog coordinates.

### Replica settings

```ini
[mysqld]
server_id = 2
relay_log = /var/lib/mysql/relay-bin
read_only = ON
super_read_only = ON
gtid_mode = ON
enforce_gtid_consistency = ON
```

Configure replica:

```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='10.0.0.10',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='StrongReplicationPassword',
  SOURCE_AUTO_POSITION=1;
START REPLICA;
SHOW REPLICA STATUS\G
```

## 2.19 Master-master considerations

Master-master setups can work but require discipline.

Common concerns:

- Auto-increment collisions.
- Conflict handling.
- Split-brain risks.
- Accidental circular replication complexity.

Typical mitigations:

```ini
server_id = 1
auto_increment_increment = 2
auto_increment_offset = 1
log_slave_updates = ON
```

On the other node:

```ini
server_id = 2
auto_increment_increment = 2
auto_increment_offset = 2
log_slave_updates = ON
```

Production guidance:

- Prefer single-writer patterns unless multi-writer is required.
- Use managed clustering technologies or write routing to avoid conflicts.

## 2.20 GTID-based replication

GTID simplifies failover and topology changes.

Benefits:

- Easier replica promotion.
- Less dependence on exact file/position tracking.
- Better automation compatibility.

Important checks:

```sql
SHOW VARIABLES LIKE 'gtid_mode';
SHOW VARIABLES LIKE 'enforce_gtid_consistency';
```

## 2.21 Replication monitoring

Key fields in `SHOW REPLICA STATUS\G`:

- Replica_IO_Running
- Replica_SQL_Running
- Seconds_Behind_Source
- Last_IO_Error
- Last_SQL_Error
- Retrieved_Gtid_Set
- Executed_Gtid_Set

## 2.22 Common replication problems

| Problem | Symptom | Typical fix |
|---|---|---|
| Network block | IO thread down | Fix firewall, routes, DNS, TLS |
| Missing binlog | Replica cannot continue | Re-clone or restore from fresh backup |
| Duplicate key on replica | SQL thread stops | Fix data divergence, then resume |
| Long lag | Seconds behind source grows | Tune queries, parallel apply, hardware, workload |

## 2.23 Backup strategies

### Logical backups with mysqldump

Best for:

- Small to medium datasets.
- Object-level portability.
- Human-readable backups.

Pros:

- Flexible restore.
- Easy single-schema export.

Cons:

- Slow for very large datasets.
- Restore time can be long.

### mysqlpump

Can parallelize some export tasks.

```bash
mysqlpump -u root -p --default-parallelism=4 --databases appdb > appdb-pump.sql
```

### Physical backups with Percona XtraBackup

Best for:

- Large InnoDB datasets.
- Hot backups with low impact.
- Faster restore objectives.

Conceptual steps:

```bash
xtrabackup --backup --target-dir=/backup/xtrabackup/base
xtrabackup --prepare --target-dir=/backup/xtrabackup/base
xtrabackup --copy-back --target-dir=/backup/xtrabackup/base
chown -R mysql:mysql /var/lib/mysql
```

### Binlog-based PITR

Strategy:

1. Take a full backup.
2. Retain binary logs.
3. Restore the backup.
4. Replay binlogs to a target time or position.

## 2.24 Cron-based backup example

```cron
0 2 * * * /usr/bin/mysqldump -u backup -p'StrongPassword' --single-transaction --all-databases | gzip > /backup/mysql/full-$(date +\%F).sql.gz
```

Better production approach:

- Use a root-owned script with credentials from a protected option file.
- Rotate backups.
- Verify restore regularly.
- Encrypt backup artifacts.

Example script pattern:

```bash
#!/usr/bin/env bash
set -euo pipefail
umask 077
STAMP=$(date +%F-%H%M)
BACKUP_DIR=/backup/mysql
mkdir -p "$BACKUP_DIR"
mysqldump --defaults-extra-file=/root/.my-backup.cnf --single-transaction --all-databases | gzip > "$BACKUP_DIR/full-$STAMP.sql.gz"
find "$BACKUP_DIR" -type f -mtime +7 -delete
```

## 2.25 Restore testing checklist

- Provision isolated restore host.
- Restore latest backup.
- Apply binlogs if required.
- Run integrity checks.
- Run application smoke tests.
- Measure recovery time.
- Document gaps.

## 2.26 Performance tuning principles

Primary goals:

- Reduce disk I/O.
- Improve index effectiveness.
- Keep transactions short.
- Size caches properly.
- Eliminate inefficient queries.

## 2.27 EXPLAIN usage

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 20;
```

Key columns to read:

| Column | Meaning |
|---|---|
| type | Access type, such as const, ref, range, ALL |
| possible_keys | Candidate indexes |
| key | Actual chosen index |
| rows | Estimated rows examined |
| Extra | Extra info such as Using filesort |

### EXPLAIN ANALYZE

Use when available for actual execution timing.

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 42;
```

## 2.28 Slow query log

Enable and analyze:

```ini
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
log_queries_not_using_indexes = 0
```

Then summarize:

```bash
mysqldumpslow -s t -t 20 /var/log/mysql/slow.log
```

Alternative tooling:

- pt-query-digest
- PMM
- Grafana dashboards

## 2.29 Indexing strategies

Good index design principles:

- Index predicates used frequently in WHERE clauses.
- Support sort order when possible.
- Avoid indexing every column.
- Favor selective columns.
- Use composite index leftmost-prefix logic intentionally.

Example:

```sql
CREATE INDEX idx_orders_customer_created ON orders (customer_id, created_at DESC);
```

This can help:

- `WHERE customer_id = ?`
- `WHERE customer_id = ? ORDER BY created_at DESC`

## 2.30 Query anti-patterns

Avoid:

- `SELECT *` in hot paths.
- Functions on indexed columns in predicates.
- Leading wildcard searches like `%term`.
- Huge transactions when not required.
- N+1 query patterns from applications.

## 2.31 InnoDB tuning notes

| Area | Consideration |
|---|---|
| Buffer pool | Largest performance lever for working set caching |
| Log file size | Helps write-heavy workloads and checkpoint spacing |
| Flushing | Balance durability and latency |
| IO threads | Tune for storage concurrency |
| File per table | Easier space reclaim and management |

## 2.32 Table maintenance

Useful operations:

```sql
ANALYZE TABLE orders;
OPTIMIZE TABLE orders;
CHECK TABLE orders;
```

Notes:

- `OPTIMIZE TABLE` can be disruptive.
- InnoDB behavior depends on version and table format.
- Prefer online schema tools for production changes.

## 2.33 Online schema change tools

Popular tools:

- pt-online-schema-change
- gh-ost

Use when adding indexes or altering large tables with minimal downtime.

## 2.34 High availability options

### MySQL InnoDB Cluster

Components:

- Group Replication.
- MySQL Shell for cluster admin.
- MySQL Router for routing.

Use cases:

- Integrated vendor-supported HA patterns.
- Single-primary or multi-primary cluster modes.

### Galera Cluster

Common in MariaDB and Percona ecosystems.

Characteristics:

- Virtually synchronous replication.
- Multi-writer capability.
- Certification-based conflict checks.

Requirements:

- Low-latency networks.
- Careful flow-control monitoring.

### ProxySQL

ProxySQL adds:

- Query routing.
- Read/write split.
- Connection multiplexing.
- Hostgroup-based failover patterns.

## 2.35 Mermaid diagram: MySQL replication architecture

```mermaid
graph LR
    A["Primary<br/>binlog enabled<br/>writes"] --> B["Replica 1<br/>IO thread<br/>SQL thread"]
    A --> C["Replica 2<br/>analytics reads"]
    A --> D["Replica 3<br/>backup offload"]
    E["Applications<br/>write traffic"] --> A
    F["Read-only clients"] --> B
    F --> C
    G["Backup jobs"] --> D
```

## 2.36 Mermaid diagram: MySQL HA architecture

```mermaid
graph TD
    A["Applications"] --> B["ProxySQL<br/>read-write split<br/>health checks"]
    B --> C["Primary MySQL<br/>writer"]
    B --> D["Replica A<br/>reader"]
    B --> E["Replica B<br/>reader"]
    F["Monitoring"] --> B
    F --> C
    F --> D
    F --> E
```

## 2.37 Linux operational checklist for MySQL

- Verify open files limit.
- Verify data volume mount options.
- Monitor inode usage.
- Capture slow log trends.
- Watch replication lag.
- Rotate and ship logs.
- Test backup restore monthly.
- Rehearse failover.

## 2.38 Common MySQL admin commands

```bash
mysqladmin -uroot -p status
mysqladmin -uroot -p variables
mysql -uroot -p -e "SHOW PROCESSLIST;"
mysql -uroot -p -e "SHOW ENGINE INNODB STATUS\G"
```

## 2.39 Example health queries

```sql
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Threads_running';
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read%';
SHOW GLOBAL STATUS LIKE 'Slow_queries';
SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

## 2.40 Upgrade guidance

Before upgrade:

- Read release notes.
- Verify plugin and auth compatibility.
- Test schema and application behavior.
- Confirm backup and rollback strategy.

After upgrade:

- Run post-upgrade checks.
- Watch slow log and error log.
- Validate replication and backup tooling.

---

# 3. PostgreSQL

## 3.1 Overview

PostgreSQL is an advanced open-source relational database with strong SQL support, extensibility, and a broad ecosystem.

Common Linux paths:

- /var/lib/postgresql/
- /var/lib/pgsql/
- /etc/postgresql/
- /var/lib/pgsql/data/

## 3.2 Installation on Ubuntu

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable --now postgresql
```

To install a specific major version from official repositories, add the PGDG repo as appropriate for your distro policy.

## 3.3 Installation on RHEL-family systems

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql
sudo dnf install -y postgresql16-server postgresql16
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb
sudo systemctl enable --now postgresql-16
```

## 3.4 Initial verification

```bash
sudo -u postgres psql -c 'SELECT version();'
systemctl status postgresql --no-pager
ss -tulpn | grep 5432
```

## 3.5 Important files

| File | Purpose |
|---|---|
| postgresql.conf | Main server configuration |
| pg_hba.conf | Client authentication rules |
| pg_ident.conf | User mapping for auth methods |
| pg_wal/ | WAL files |
| base/ | Main data files |

## 3.6 `postgresql.conf` essentials

Example settings:

```conf
listen_addresses = '*'
port = 5432
max_connections = 300
shared_buffers = 8GB
effective_cache_size = 24GB
work_mem = 16MB
maintenance_work_mem = 1GB
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 4GB
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
random_page_cost = 1.1
effective_io_concurrency = 200
shared_preload_libraries = 'pg_stat_statements'
log_min_duration_statement = 1000
log_checkpoints = on
log_connections = on
log_disconnections = on
```

## 3.7 Key PostgreSQL parameters explained

| Parameter | Purpose | General guidance |
|---|---|---|
| shared_buffers | Main shared cache | Often 25 percent of RAM on dedicated servers |
| effective_cache_size | Planner estimate of OS cache | Often 50 to 75 percent of RAM |
| work_mem | Per-operation work area | Set carefully; multiplied by concurrent operations |
| maintenance_work_mem | Vacuum/index build memory | Larger for maintenance tasks |
| wal_level | WAL detail for replication/logical decoding | replica or logical as needed |
| max_connections | Backend processes | Keep moderate; use pooling |
| checkpoint_completion_target | Checkpoint smoothing | Usually 0.7 to 0.9 |
| random_page_cost | Planner storage model | Lower on SSD |

## 3.8 `pg_hba.conf` examples

Local peer auth:

```conf
local   all             postgres                                peer
```

Password auth for app subnet:

```conf
host    appdb           appuser         10.0.0.0/24            scram-sha-256
```

Replication user:

```conf
host    replication     repl            10.0.0.0/24            scram-sha-256
```

After changes:

```bash
sudo systemctl reload postgresql
```

## 3.9 Initial role and database setup

```bash
sudo -u postgres createuser --interactive
sudo -u postgres createdb appdb
```

Using SQL:

```sql
CREATE ROLE appuser LOGIN PASSWORD 'StrongPassword';
CREATE DATABASE appdb OWNER appuser;
```

## 3.10 User and role management

### Create role

```sql
CREATE ROLE analyst LOGIN PASSWORD 'StrongPassword';
```

### Grant database connection and schema privileges

```sql
GRANT CONNECT ON DATABASE appdb TO analyst;
\c appdb
GRANT USAGE ON SCHEMA public TO analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO analyst;
```

### Role inheritance

```sql
CREATE ROLE reporting_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reporting_readonly;
GRANT reporting_readonly TO analyst;
```

## 3.11 Database operations

### Create schema and table

```sql
CREATE SCHEMA sales AUTHORIZATION appuser;
CREATE TABLE sales.orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    total_amount NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Basic maintenance SQL

```sql
VACUUM (VERBOSE, ANALYZE) sales.orders;
REINDEX TABLE sales.orders;
```

## 3.12 Service administration

```bash
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
sudo systemctl reload postgresql
sudo journalctl -u postgresql -n 200 --no-pager
```

## 3.13 Replication overview

Major PostgreSQL replication styles:

- Physical streaming replication.
- Logical replication.
- WAL shipping.
- Cascading replication.

## 3.14 Streaming replication setup

### Primary configuration

In `postgresql.conf`:

```conf
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 4GB
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
```

In `pg_hba.conf`:

```conf
host replication repl 10.0.0.0/24 scram-sha-256
```

Create replication role:

```sql
CREATE ROLE repl WITH REPLICATION LOGIN PASSWORD 'StrongReplicationPassword';
```

### Standby creation with `pg_basebackup`

```bash
sudo -u postgres pg_basebackup -h 10.0.0.10 -D /var/lib/postgresql/16/main -U repl -P -R -X stream -C -S standby1
```

Notes:

- `-R` writes standby connection settings.
- `-C -S standby1` creates a replication slot.

## 3.15 Replication monitoring

On primary:

```sql
SELECT pid, usename, application_name, client_addr, state, sync_state, write_lag, flush_lag, replay_lag
FROM pg_stat_replication;
```

On standby:

```sql
SELECT pg_is_in_recovery();
SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), now() - pg_last_xact_replay_timestamp() AS replay_delay;
```

## 3.16 Logical replication

Use when:

- Replicating a subset of tables.
- Upgrading between major versions with minimal downtime.
- Feeding downstream analytics or microservices.

### Publisher

```sql
CREATE PUBLICATION app_pub FOR TABLE sales.orders;
```

### Subscriber

```sql
CREATE SUBSCRIPTION app_sub
CONNECTION 'host=10.0.0.10 dbname=appdb user=repl password=StrongReplicationPassword'
PUBLICATION app_pub;
```

## 3.17 WAL archiving

WAL archiving supports PITR and disaster recovery.

Example:

```conf
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
archive_timeout = 60
```

Best practices:

- Archive to durable storage.
- Monitor failures aggressively.
- Retain enough WAL for recovery objectives.

## 3.18 Backup methods

### Logical backup with `pg_dump`

```bash
pg_dump -U postgres -Fc appdb > appdb.dump
```

Restore:

```bash
createdb -U postgres appdb_restored
pg_restore -U postgres -d appdb_restored appdb.dump
```

### Full cluster logical dump

```bash
pg_dumpall -U postgres > full-cluster.sql
```

### Physical base backup

```bash
pg_basebackup -h 10.0.0.10 -U repl -D /backup/pg/base -P -X stream
```

### pgBackRest

Highly recommended for production PostgreSQL backup management.

Benefits:

- Full, differential, incremental backup support.
- WAL archiving integration.
- Restore automation.
- Compression and retention control.

Conceptual example:

```bash
pgbackrest --stanza=main stanza-create
pgbackrest --stanza=main backup
pgbackrest --stanza=main restore
```

## 3.19 Point-in-time recovery workflow

1. Restore a base backup.
2. Provide WAL archives.
3. Set recovery target time or LSN.
4. Start server in recovery.
5. Promote after reaching target.

Recovery settings approach varies by PostgreSQL major version, but the concept remains the same.

## 3.20 Performance tuning principles

Key areas:

- Shared memory sizing.
- Query plan quality.
- Index design.
- Vacuum health.
- WAL and checkpoint tuning.
- Connection management.

## 3.21 EXPLAIN and EXPLAIN ANALYZE

```sql
EXPLAIN SELECT * FROM sales.orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 20;
EXPLAIN ANALYZE SELECT * FROM sales.orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 20;
```

What to inspect:

- Sequential scan vs index scan.
- Rows estimated vs actual rows.
- Buffers read and hit.
- Sort methods and temp disk usage.
- Hash join size and spills.

## 3.22 `pg_stat_statements`

Enable in `shared_preload_libraries` and create extension:

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

## 3.23 Vacuum and autovacuum

PostgreSQL uses MVCC, which means dead tuples must be cleaned.

Watch for:

- Table bloat.
- Transaction ID wraparound risk.
- Under-provisioned autovacuum.

Useful queries:

```sql
SELECT relname, n_live_tup, n_dead_tup, last_autovacuum, last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

## 3.24 Indexing strategies

Types:

- B-tree for equality and range lookups.
- GIN for arrays, JSONB, full-text search.
- GiST for geometric and nearest-neighbor cases.
- BRIN for very large naturally ordered tables.
- Hash in niche scenarios.

Examples:

```sql
CREATE INDEX idx_orders_customer_created ON sales.orders (customer_id, created_at DESC);
CREATE INDEX idx_events_payload_gin ON events USING GIN (payload jsonb_path_ops);
```

## 3.25 Partitioning

Use declarative partitioning for large time-based or key-based datasets.

Example:

```sql
CREATE TABLE metrics (
    ts timestamptz NOT NULL,
    value numeric NOT NULL
) PARTITION BY RANGE (ts);
```

Why partition:

- Faster retention deletion.
- Improved maintenance windows.
- Better planner pruning for large tables.

## 3.26 Connection pooling with PgBouncer

Why it matters:

- PostgreSQL backend processes are relatively heavy.
- Thousands of app connections can harm memory and throughput.

Pool modes:

- session
- transaction
- statement

Typical recommendation:

- Use transaction pooling for stateless apps when compatible.

## 3.27 HA setup options

### Patroni + etcd

A common HA stack uses:

- PostgreSQL nodes.
- Patroni for orchestration.
- etcd or Consul as distributed configuration store.
- HAProxy or similar for routing.

Benefits:

- Automated failover.
- Cluster state coordination.
- Better operational predictability.

### pgpool-II

Features:

- Connection pooling.
- Query routing.
- Some failover support.
- Load balancing for reads.

### Citus

Citus extends PostgreSQL for distributed scale-out workloads.

Best fit:

- Multi-tenant SaaS.
- Large distributed analytical or mixed workloads.

## 3.28 Extensions

### PostGIS

Adds advanced geospatial types and functions.

```sql
CREATE EXTENSION postgis;
```

### pg_partman

Automates partition maintenance.

```sql
CREATE EXTENSION pg_partman;
```

### TimescaleDB

Optimized for time-series workloads.

```sql
CREATE EXTENSION timescaledb;
```

### Other useful extensions

- pg_trgm
- hstore
- uuid-ossp
- citext

## 3.29 Logs and observability

Important log options:

- `log_min_duration_statement`
- `log_checkpoints`
- `log_temp_files`
- `log_lock_waits`
- `deadlock_timeout`

Example:

```conf
log_min_duration_statement = 500
log_temp_files = 0
log_lock_waits = on
deadlock_timeout = 1s
```

## 3.30 Mermaid diagram: PostgreSQL architecture

```mermaid
graph TD
    A["Clients"] --> B["PostgreSQL postmaster"]
    B --> C["Backend process 1"]
    B --> D["Backend process 2"]
    B --> E["Background writer"]
    B --> F["Checkpointer"]
    B --> G["WAL writer"]
    B --> H["Autovacuum launcher"]
    H --> I["Autovacuum workers"]
    C --> J["Shared buffers"]
    D --> J
    J --> K["Data files"]
    G --> L["WAL files"]
```

## 3.31 Mermaid diagram: WAL-based replication

```mermaid
graph LR
    A["Primary PostgreSQL<br/>WAL generation"] --> B["Streaming replication<br/>TCP 5432"]
    B --> C["Standby 1<br/>replay WAL"]
    B --> D["Standby 2<br/>replay WAL"]
    A --> E["Archive storage<br/>WAL archive"]
    E --> C
    E --> D
```

## 3.32 Common PostgreSQL admin commands

```bash
sudo -u postgres psql
sudo -u postgres psql -c '\l'
sudo -u postgres psql -c '\du'
sudo -u postgres psql -d appdb -c '\dt+'
sudo -u postgres vacuumdb --all --analyze-in-stages
```

## 3.33 Example monitoring queries

### Long-running queries

```sql
SELECT pid, usename, state, now() - query_start AS age, query
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY age DESC;
```

### Blocking locks

```sql
SELECT blocked.pid AS blocked_pid,
       blocked.query AS blocked_query,
       blocker.pid AS blocker_pid,
       blocker.query AS blocker_query
FROM pg_stat_activity blocked
JOIN pg_locks blocked_locks ON blocked.pid = blocked_locks.pid AND NOT blocked_locks.granted
JOIN pg_locks blocker_locks ON blocker_locks.locktype = blocked_locks.locktype
    AND blocker_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocker_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocker_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocker_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocker_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocker_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocker_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocker_locks.pid <> blocked_locks.pid
JOIN pg_stat_activity blocker ON blocker.pid = blocker_locks.pid
WHERE blocker_locks.granted;
```

## 3.34 Upgrade approaches

Options:

- In-place package upgrade when supported by distro packaging rules.
- `pg_upgrade` for fast major upgrade.
- Logical replication for minimal-downtime migration.
- Dump and restore for smaller environments.

## 3.35 Production checklist

- WAL archiving tested.
- Backup restore tested.
- Autovacuum monitored.
- Connection pool deployed.
- TLS enforced.
- Failover rehearsed.
- Extension compatibility verified.

---

# 4. MongoDB

## 4.1 Overview

MongoDB is a document database that stores JSON-like BSON documents.

Best suited for:

- Flexible schemas.
- Content and catalog systems.
- User profile stores.
- Event-style ingestion patterns.

## 4.2 Installation on Ubuntu

MongoDB packages are often installed from the vendor repository.

Conceptual example:

```bash
sudo apt update
sudo apt install -y gnupg curl
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl enable --now mongod
```

## 4.3 Installation on RHEL-family systems

Conceptual example:

```bash
cat <<'EOF' | sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-7.0.asc
EOF
sudo dnf install -y mongodb-org
sudo systemctl enable --now mongod
```

## 4.4 Verification

```bash
systemctl status mongod --no-pager
ss -tulpn | grep 27017
mongosh --eval 'db.runCommand({ ping: 1 })'
```

## 4.5 Main configuration file

Common path:

- `/etc/mongod.conf`

Example:

```yaml
storage:
  dbPath: /var/lib/mongo
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
net:
  port: 27017
  bindIp: 0.0.0.0
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
security:
  authorization: enabled
replication:
  replSetName: rs0
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100
```

## 4.6 Authentication setup

Create admin user:

```javascript
use admin
db.createUser({
  user: 'admin',
  pwd: 'StrongPassword',
  roles: [ { role: 'root', db: 'admin' } ]
})
```

Then connect with auth:

```bash
mongosh -u admin -p --authenticationDatabase admin
```

## 4.7 CRUD operations from shell

### Create / insert

```javascript
use appdb
db.customers.insertOne({ name: 'Alice', email: 'alice@example.com', createdAt: new Date() })
```

### Read

```javascript
db.customers.find({ email: 'alice@example.com' })
```

### Update

```javascript
db.customers.updateOne(
  { email: 'alice@example.com' },
  { $set: { name: 'Alice Smith' } }
)
```

### Delete

```javascript
db.customers.deleteOne({ email: 'alice@example.com' })
```

## 4.8 Common admin commands

```javascript
db.serverStatus()
db.stats()
show dbs
show collections
db.currentOp()
```

## 4.9 Replica sets overview

Replica sets provide:

- Redundancy.
- Automatic failover.
- Configurable read preferences.

A typical replica set has:

- One primary.
- One or more secondaries.
- Optional arbiter in small environments.

## 4.10 Replica set setup

On each node set the same replica set name in `mongod.conf`.

Example:

```yaml
replication:
  replSetName: rs0
```

Initialize on one node:

```javascript
rs.initiate({
  _id: 'rs0',
  members: [
    { _id: 0, host: 'mongo1:27017' },
    { _id: 1, host: 'mongo2:27017' },
    { _id: 2, host: 'mongo3:27017' }
  ]
})
```

Check state:

```javascript
rs.status()
rs.printSecondaryReplicationInfo()
```

## 4.11 Read and write concerns

Important for durability and consistency:

- `writeConcern: { w: 'majority' }`
- `readConcern: 'majority'`
- Read preference: primary, primaryPreferred, secondary, nearest

## 4.12 Sharding overview

MongoDB sharding uses:

- Shards to store data.
- Config servers to store metadata.
- `mongos` routers for query routing.

Good use cases:

- Very large datasets.
- Horizontal write scaling.
- Geo-distributed or large tenant sets.

## 4.13 Sharding setup outline

1. Deploy config server replica set.
2. Deploy shard replica sets.
3. Start `mongos` routers.
4. Add shards.
5. Enable sharding for database and collections.
6. Choose shard keys carefully.

Example commands:

```javascript
sh.addShard('rsShard1/mongo1:27017,mongo2:27017,mongo3:27017')
sh.enableSharding('appdb')
sh.shardCollection('appdb.orders', { customer_id: 1, createdAt: 1 })
```

## 4.14 Choosing a shard key

Important qualities:

- High cardinality.
- Good distribution.
- Supports major query patterns.
- Avoids monotonically increasing hotspots.

Bad examples:

- Pure timestamp for a write-heavy workload.
- Low-cardinality status field.

## 4.15 Indexing strategies

Common index types:

- Single field.
- Compound.
- Multikey for arrays.
- Text.
- Hashed.
- TTL.
- Partial.

Examples:

```javascript
db.orders.createIndex({ customer_id: 1, createdAt: -1 })
db.sessions.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 })
db.articles.createIndex({ title: 'text', body: 'text' })
```

## 4.16 Explain plans

```javascript
db.orders.find({ customer_id: 42 }).sort({ createdAt: -1 }).explain('executionStats')
```

Inspect:

- Winning plan.
- Documents examined.
- Keys examined.
- Stage types.
- Execution time.

## 4.17 Backup and restore

### `mongodump`

```bash
mongodump --uri="mongodb://backupuser:password@mongo1:27017,mongo2:27017/?replicaSet=rs0" --out /backup/mongo/dump
```

### `mongorestore`

```bash
mongorestore --uri="mongodb://admin:password@mongo1:27017/?authSource=admin" /backup/mongo/dump
```

Considerations:

- Use consistent backups for replica sets.
- For large deployments, filesystem snapshots may be used with proper coordination.
- Test restores regularly.

## 4.18 Monitoring basics

Watch:

- Opcounters.
- Replication lag.
- WiredTiger cache metrics.
- Page faults.
- Slow query profiling.
- Disk free space.

Useful shell commands:

```javascript
db.serverStatus().wiredTiger.cache
db.getProfilingStatus()
```

## 4.19 Slow query profiling

Enable profiling carefully:

```javascript
db.setProfilingLevel(1, { slowms: 100 })
```

Review:

```javascript
db.system.profile.find().sort({ ts: -1 }).limit(10)
```

## 4.20 Security notes

- Enable authentication.
- Restrict bind addresses.
- Use TLS.
- Use role-based access.
- Protect internal keyfiles or x.509 certs for cluster auth.

## 4.21 Mermaid diagram: MongoDB sharding architecture

```mermaid
graph TD
    A["Applications"] --> B["mongos Router 1"]
    A --> C["mongos Router 2"]
    B --> D["Config Server Replica Set"]
    C --> D
    B --> E["Shard 1 Replica Set"]
    B --> F["Shard 2 Replica Set"]
    C --> E
    C --> F
```

## 4.22 Operational checklist

- Avoid unbounded document growth.
- Keep document size under 16 MB limit.
- Design indexes around major filters and sorts.
- Use appropriate read/write concerns.
- Monitor chunk balance in sharded clusters.
- Rehearse node replacement and primary failover.

---

# 5. Redis

## 5.1 Overview

Redis is an in-memory data structure store used for caching, session storage, streaming, pub/sub, leaderboards, queues, and lightweight coordination.

## 5.2 Installation on Ubuntu

```bash
sudo apt update
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
```

## 5.3 Installation on RHEL-family systems

```bash
sudo dnf install -y redis
sudo systemctl enable --now redis
```

## 5.4 Verification

```bash
redis-cli ping
systemctl status redis --no-pager
ss -tulpn | grep 6379
```

## 5.5 Configuration basics

Common config file:

- `/etc/redis/redis.conf`

Important settings:

```conf
bind 0.0.0.0
port 6379
timeout 0
tcp-keepalive 300
supervised systemd
loglevel notice
databases 16
maxmemory 8gb
maxmemory-policy allkeys-lru
appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000
requirepass StrongPassword
```

## 5.6 Data types and commands

### Strings

```bash
redis-cli SET site:name linuxdbguide
redis-cli GET site:name
```

### Hashes

```bash
redis-cli HSET user:100 name Alice email alice@example.com
redis-cli HGETALL user:100
```

### Lists

```bash
redis-cli LPUSH jobs pending1 pending2
redis-cli BRPOP jobs 5
```

### Sets

```bash
redis-cli SADD tags redis cache session
redis-cli SMEMBERS tags
```

### Sorted sets

```bash
redis-cli ZADD leaderboard 100 user1 200 user2
redis-cli ZREVRANGE leaderboard 0 9 WITHSCORES
```

### Streams

```bash
redis-cli XADD orders '*' customer_id 42 amount 19.99
redis-cli XRANGE orders - + COUNT 10
```

## 5.7 Persistence modes

### RDB snapshots

Advantages:

- Compact snapshot files.
- Faster restarts for some cases.
- Lower write amplification than AOF.

Disadvantages:

- Risk of more data loss between snapshots.

### AOF

Advantages:

- Better durability.
- Replay-based restoration.

Disadvantages:

- Larger files.
- More write overhead.

### Hybrid approach

Many production environments use both or rely on modern AOF rewrite behavior depending on Redis version and objectives.

## 5.8 Replication

Simple replica config example:

```conf
replicaof 10.0.0.10 6379
masterauth StrongPassword
```

Check replication:

```bash
redis-cli INFO replication
```

## 5.9 Redis Sentinel

Sentinel provides:

- Monitoring.
- Primary discovery.
- Automatic failover.
- Notification hooks.

Example sentinel config fragment:

```conf
sentinel monitor mymaster 10.0.0.10 6379 2
sentinel auth-pass mymaster StrongPassword
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

## 5.10 Redis Cluster

Redis Cluster provides:

- Data sharding.
- Replication for each master shard.
- Slot-based partitioning.

Example cluster creation concept:

```bash
redis-cli --cluster create \
  10.0.0.11:6379 10.0.0.12:6379 10.0.0.13:6379 \
  10.0.0.14:6379 10.0.0.15:6379 10.0.0.16:6379 \
  --cluster-replicas 1
```

## 5.11 Use cases

| Use case | Why Redis fits |
|---|---|
| Caching | Extremely fast reads and writes |
| Sessions | TTL support and simple key model |
| Queues | Lists and streams |
| Rate limiting | Atomic counters and expiry |
| Pub/Sub | Native publish-subscribe primitives |
| Leaderboards | Sorted sets |

## 5.12 Memory management

Key concepts:

- `maxmemory`
- eviction policy
- fragmentation
- key TTL hygiene

Common policies:

- noeviction
- allkeys-lru
- volatile-lru
- allkeys-lfu

## 5.13 Performance tuning

- Keep values reasonably sized.
- Avoid huge keys and gigantic hash/list/set members.
- Pipeline commands from applications.
- Use Lua or transactions for atomic multi-step logic where needed.
- Monitor memory fragmentation ratio.

## 5.14 Backup and restore

Copying `dump.rdb` or AOF files is the basic mechanism, but production-safe backups should coordinate with persistence and replication status.

Operational ideas:

- Snapshot from a replica when possible.
- Verify RDB/AOF integrity.
- Store encrypted off-host backups.

## 5.15 Security notes

- Do not expose Redis directly to the internet.
- Use `bind` and firewall rules.
- Use ACLs or passwords depending on version.
- Consider TLS support.
- Rename or disable dangerous commands when appropriate.

## 5.16 Mermaid diagram: Redis Cluster architecture

```mermaid
graph LR
    A["Applications"] --> B["Redis Cluster"]
    B --> C["Master 1<br/>slots 0-5460"]
    B --> D["Master 2<br/>slots 5461-10922"]
    B --> E["Master 3<br/>slots 10923-16383"]
    C --> F["Replica 1"]
    D --> G["Replica 2"]
    E --> H["Replica 3"]
```

## 5.17 Operational checklist

- Set explicit `maxmemory`.
- Choose eviction policy intentionally.
- Monitor persistence latency.
- Validate Sentinel or Cluster failover.
- Track keyspace hit ratio.
- Review oversized keys.

---

# 6. Elasticsearch

## 6.1 Overview

Elasticsearch is a distributed search and analytics engine built around inverted indexes and JSON documents.

Typical use cases:

- Full-text search.
- Log analytics.
- Observability pipelines.
- Product search.
- Security event analysis.

## 6.2 Installation concepts

Elasticsearch is commonly installed from vendor packages or containers. JVM sizing and kernel settings are critical.

Ubuntu-style example:

```bash
sudo apt update
sudo apt install -y apt-transport-https openjdk-17-jdk
# Add Elastic repository according to current vendor guidance.
sudo apt install -y elasticsearch
sudo systemctl enable --now elasticsearch
```

RHEL-style example:

```bash
sudo dnf install -y java-17-openjdk
# Add Elastic repository according to current vendor guidance.
sudo dnf install -y elasticsearch
sudo systemctl enable --now elasticsearch
```

## 6.3 Important Linux settings

Set memory map and file limits as required.

Example:

```bash
sudo sysctl -w vm.max_map_count=262144
ulimit -n 65535
```

Persist sysctl:

```conf
vm.max_map_count = 262144
```

## 6.4 Single-node configuration

Common file:

- `/etc/elasticsearch/elasticsearch.yml`

Example:

```yaml
cluster.name: prod-search
node.name: es1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
xpack.security.enabled: true
```

## 6.5 Cluster configuration basics

Example:

```yaml
cluster.name: prod-search
node.name: es1
node.roles: [ master, data, ingest ]
network.host: 10.0.0.11
http.port: 9200
discovery.seed_hosts: ["10.0.0.11", "10.0.0.12", "10.0.0.13"]
cluster.initial_master_nodes: ["es1", "es2", "es3"]
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

## 6.6 Verify cluster health

```bash
curl -u elastic:password -k https://localhost:9200/_cluster/health?pretty
curl -u elastic:password -k https://localhost:9200/_cat/nodes?v
curl -u elastic:password -k https://localhost:9200/_cat/indices?v
```

## 6.7 Index management

### Create index

```bash
curl -u elastic:password -k -X PUT https://localhost:9200/products -H 'Content-Type: application/json' -d '
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}'
```

### Delete index

```bash
curl -u elastic:password -k -X DELETE https://localhost:9200/products
```

### Index a document

```bash
curl -u elastic:password -k -X POST https://localhost:9200/products/_doc/1 -H 'Content-Type: application/json' -d '
{
  "name": "Laptop",
  "price": 999,
  "category": "electronics"
}'
```

## 6.8 Mapping and queries

Explicit mapping example:

```bash
curl -u elastic:password -k -X PUT https://localhost:9200/products -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "category": { "type": "keyword" },
      "price": { "type": "double" },
      "created_at": { "type": "date" }
    }
  }
}'
```

Search example:

```bash
curl -u elastic:password -k -X GET https://localhost:9200/products/_search -H 'Content-Type: application/json' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" } }
      ],
      "filter": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "lte": 1200 } } }
      ]
    }
  }
}'
```

## 6.9 Index lifecycle management

ILM helps automate:

- Hot/warm/cold movement.
- Rollover.
- Retention deletion.
- Force merge.

Especially important for log and metrics indices.

## 6.10 Cluster management

Key concepts:

- Primary shards.
- Replica shards.
- Node roles.
- Cluster state.
- Rebalancing.

Watch for:

- Unassigned shards.
- Heap pressure.
- Large cluster state.
- Slow merges.
- Long garbage collection pauses.

## 6.11 Backup with snapshots

Elasticsearch backups should use snapshots, not filesystem copies of live data directories.

Register a repository example:

```bash
curl -u elastic:password -k -X PUT https://localhost:9200/_snapshot/fsrepo -H 'Content-Type: application/json' -d '
{
  "type": "fs",
  "settings": {
    "location": "/backup/elasticsearch"
  }
}'
```

Create snapshot:

```bash
curl -u elastic:password -k -X PUT https://localhost:9200/_snapshot/fsrepo/snap-2025-01-01?wait_for_completion=true
```

Restore snapshot:

```bash
curl -u elastic:password -k -X POST https://localhost:9200/_snapshot/fsrepo/snap-2025-01-01/_restore -H 'Content-Type: application/json' -d '{}'
```

## 6.12 Performance tuning basics

Key areas:

- JVM heap sizing.
- Shard count discipline.
- Mapping correctness.
- Bulk indexing strategies.
- Refresh interval tuning.
- Query cache and filesystem cache effectiveness.

Rules of thumb:

- Avoid too many tiny shards.
- Keep heap under compressed OOPs threshold where applicable.
- Use bulk indexing for ingestion.
- Prefer filters for exact matches.

## 6.13 ELK and EFK integration

Common stacks:

- ELK: Elasticsearch + Logstash + Kibana.
- EFK: Elasticsearch + Fluentd/Fluent Bit + Kibana.

Use cases:

- Application logging.
- Infrastructure logs.
- Security analytics.
- Trace and metric correlation.

## 6.14 Security essentials

- Enable x-pack security.
- Use TLS for HTTP and transport layers.
- Restrict roles and index privileges.
- Protect snapshot repositories.
- Avoid public exposure of node endpoints.

## 6.15 Mermaid diagram: Elasticsearch cluster architecture

```mermaid
graph TD
    A["Clients and Kibana"] --> B["Coordinating Node"]
    B --> C["Master Node 1"]
    B --> D["Master Node 2"]
    B --> E["Master Node 3"]
    B --> F["Data Node 1"]
    B --> G["Data Node 2"]
    B --> H["Ingest Node"]
    F --> I["Primary and replica shards"]
    G --> I
```

## 6.16 Operational checklist

- Validate `vm.max_map_count`.
- Monitor JVM GC and heap usage.
- Keep shard count sane.
- Enforce snapshot success monitoring.
- Review hot threads during incidents.
- Track indexing and search latency separately.

---

# 7. SQLite

## 7.1 When to use SQLite

SQLite is excellent when you need:

- Embedded storage.
- Simple deployment.
- Single-file portability.
- Moderate local workloads.
- Offline-first applications.

Less suitable when you need:

- Many concurrent writers.
- Networked multi-node clustering.
- Centralized enterprise access controls.

## 7.2 Installation

```bash
sudo apt install -y sqlite3
# or
sudo dnf install -y sqlite
```

## 7.3 Create and open a database

```bash
sqlite3 app.db
```

Inside the CLI:

```sql
CREATE TABLE customers (
  id INTEGER PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## 7.4 Useful CLI commands

```sql
.tables
.schema customers
.headers on
.mode column
SELECT * FROM customers;
.quit
```

## 7.5 Backup and recovery

### Online backup using `.backup`

```sql
.backup backup.db
```

### File copy backup

Only safe when coordinated correctly, especially in WAL mode and active workloads.

### Dump and restore

```bash
sqlite3 app.db .dump > app.sql
sqlite3 restored.db < app.sql
```

## 7.6 WAL mode

WAL mode improves concurrency characteristics in many scenarios.

```sql
PRAGMA journal_mode=WAL;
```

## 7.7 Performance tips

- Use transactions for bulk inserts.
- Use prepared statements in applications.
- Create indexes for common predicates.
- Use WAL mode when suitable.
- Vacuum periodically if space churn is high.

Bulk insert example:

```sql
BEGIN;
INSERT INTO customers(email, name) VALUES ('a@example.com', 'A');
INSERT INTO customers(email, name) VALUES ('b@example.com', 'B');
COMMIT;
```

## 7.8 Integrity checks

```sql
PRAGMA integrity_check;
PRAGMA quick_check;
```

## 7.9 When SQLite is a smart production choice

Good examples:

- Local application caches.
- Edge devices.
- Desktop applications.
- Small internal tools.
- Read-mostly content bundles.

---

# 8. Database Administration Tasks

## 8.1 Monitoring database performance

All production databases should be monitored at multiple layers:

- Host metrics.
- Database internal metrics.
- Query latency.
- Replication health.
- Backup success.
- Capacity growth.

## 8.2 Host metrics to watch

| Metric | Why it matters |
|---|---|
| CPU utilization | Saturation and thread contention |
| Load average | Queue buildup |
| Free memory | Risk of swapping and cache starvation |
| IOPS and latency | Storage bottlenecks |
| Disk queue depth | Device contention |
| Network throughput and retransmits | Replication and client performance |
| Filesystem free space | Outage prevention |

Useful commands:

```bash
top
iostat -xz 1
vmstat 1
sar -n DEV 1
df -h
```

## 8.3 Slow query analysis workflow

1. Enable or collect slow query source.
2. Aggregate by fingerprint.
3. Rank by total time and frequency.
4. Review schema and indexes.
5. Test rewritten queries.
6. Validate with plan analysis.
7. Deploy safely and re-measure.

## 8.4 Connection pooling

### PgBouncer

Best for PostgreSQL connection pooling.

Benefits:

- Drastically reduces backend process count.
- Improves stability under bursty app traffic.
- Simplifies failover routing in some architectures.

Example config fragment:

```ini
[databases]
appdb = host=10.0.0.10 port=5432 dbname=appdb

[pgbouncer]
listen_port = 6432
listen_addr = 0.0.0.0
auth_type = scram-sha-256
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 100
```

### ProxySQL

Best-known option for MySQL-family connection pooling and routing.

Benefits:

- Multiplexing.
- Read/write split.
- Query rules.
- Health checks.

## 8.5 Migration tools

### Flyway

Good for SQL-first versioned migrations.

Typical workflow:

- Write numbered SQL migration files.
- Run `flyway migrate` in CI/CD.
- Track schema history table.

### Liquibase

Good for complex change tracking with XML/YAML/SQL formats.

### Alembic

Widely used with SQLAlchemy-based Python projects.

### Django migrations

Application-integrated schema migration framework for Django apps.

## 8.6 Schema design best practices

- Model entities explicitly.
- Use correct data types.
- Keep naming conventions consistent.
- Normalize where it improves correctness.
- Denormalize intentionally for performance, not by accident.
- Add primary keys everywhere.
- Add foreign keys where integrity matters.
- Track creation/update timestamps.

## 8.7 Normalization levels in practice

### First normal form

- Atomic values.
- No repeating groups.

### Second normal form

- No partial dependency on composite keys.

### Third normal form

- Eliminate transitive dependencies.

Pragmatic note:

- Many production schemas blend normalization with targeted denormalization.

## 8.8 Index optimization

General principles:

- Every index has a write cost.
- Composite index order matters.
- Redundant indexes waste memory and CPU.
- Covering indexes can eliminate table lookups.
- Low-selectivity indexes may be useless.

### Redundant index example

If you have:

- `(customer_id, created_at)`
- `(customer_id)`

The second may be redundant depending on engine and workload.

## 8.9 Query optimization techniques

- Filter early.
- Return only needed columns.
- Avoid functions on indexed predicate columns.
- Limit result sets.
- Break apart pathological queries when necessary.
- Use prepared statements.
- Avoid row-by-row application loops for set-based operations.

## 8.10 Schema change planning

Questions to ask before a production migration:

- Is the change blocking?
- Does it rewrite the table?
- How long will locks last?
- Can it be done online?
- Is a backfill required?
- Can the application tolerate mixed-schema deployments?

## 8.11 Capacity planning

Track trends for:

- Total data size.
- Index size.
- WAL/binlog generation.
- QPS/TPS.
- Cache hit ratios.
- Top query classes.
- Backup duration and restore time.

## 8.12 Query execution pipeline mermaid diagram

```mermaid
graph LR
    A["Client query"] --> B["Parser"]
    B --> C["Rewriter"]
    C --> D["Optimizer"]
    D --> E["Executor"]
    E --> F["Buffer cache"]
    F --> G["Storage engine"]
    G --> H["Result returned"]
```

## 8.13 Documentation and runbooks

Every production database service should have:

- Owner and escalation path.
- Topology diagram.
- Backup policy.
- Restore playbook.
- Failover runbook.
- Upgrade runbook.
- Access request process.
- Capacity plan.

## 8.14 Common DBA routine schedule

| Frequency | Task |
|---|---|
| Daily | Review backups, replication, errors, space |
| Weekly | Review slow queries, top growth, failed jobs |
| Monthly | Test restores, patch review, failover drill |
| Quarterly | Access audit, capacity review, upgrade plan |

---

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

# 11. Troubleshooting

## 11.1 General troubleshooting process

1. Confirm scope and blast radius.
2. Determine whether issue is host, network, storage, or database internals.
3. Check recent changes.
4. Inspect logs and health metrics.
5. Stabilize service first.
6. Preserve evidence.
7. Remediate and document.

## 11.2 Connection issues

Checklist:

- Is the service running?
- Is the port listening?
- Is the firewall open?
- Is DNS resolving correctly?
- Is TLS configured correctly?
- Are credentials valid?
- Are connection limits exhausted?

Useful commands:

```bash
systemctl status mysql --no-pager
ss -tulpn | egrep '3306|5432|27017|6379|9200'
ping db-host
nc -vz db-host 5432
```

## 11.3 Slow queries

Workflow:

- Identify query class.
- Check plan.
- Review indexes.
- Measure rows examined vs returned.
- Check temp file or sort spill behavior.
- Evaluate cache hit ratios.
- Confirm storage latency.

## 11.4 Lock contention

Symptoms:

- Requests hang.
- Transaction age grows.
- Deadlock errors appear.
- CPU is low but latency is high.

Remediation ideas:

- Find blocker queries.
- Kill or cancel offenders carefully.
- Reduce transaction scope.
- Add indexes to reduce lock duration.
- Reorder statements consistently in app logic.

## 11.5 Replication lag

Causes:

- Slow disk on replica.
- Long-running queries on replica.
- Heavy write bursts.
- Network latency.
- Single-thread apply bottlenecks in some engines.

Actions:

- Check apply worker status.
- Remove expensive read traffic from lagging replicas.
- Rebuild irreparably diverged replicas.
- Scale hardware or tune replication workers.

## 11.6 Disk space issues

Emergency actions:

- Stop unnecessary log growth.
- Purge safe old backups or logs according to policy.
- Expand volumes.
- Move archives.
- Avoid deleting active WAL/binlog or live data blindly.

Useful commands:

```bash
df -h
du -sh /var/lib/mysql/* | sort -h | tail
find /var/log -type f -size +100M -ls
```

## 11.7 Recovery from corruption

Important rules:

- Do not panic-write to corrupted volumes.
- Preserve copies for forensic analysis if required.
- Prefer restore from known-good backup.
- Follow engine-specific recovery guidance.

Examples:

- MySQL: inspect InnoDB recovery options cautiously.
- PostgreSQL: restore from base backup and WAL.
- MongoDB: validate WiredTiger and restore if necessary.
- SQLite: `.recover` or dump salvage in some cases.

## 11.8 Engine-specific quick triage

### MySQL

```sql
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS\G
SHOW REPLICA STATUS\G
```

### PostgreSQL

```sql
SELECT * FROM pg_stat_activity;
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_locks;
```

### MongoDB

```javascript
db.serverStatus()
rs.status()
db.currentOp()
```

### Redis

```bash
redis-cli INFO all
redis-cli LATENCY LATEST
redis-cli SLOWLOG GET 20
```

### Elasticsearch

```bash
curl -u elastic:password -k https://localhost:9200/_cluster/health?pretty
curl -u elastic:password -k https://localhost:9200/_cat/shards?v
curl -u elastic:password -k https://localhost:9200/_nodes/stats?pretty
```

---

# 12. Appendix: Linux DBA Command Reference

## 12.1 Process and service commands

```bash
systemctl status mysql --no-pager
systemctl status postgresql --no-pager
systemctl status mongod --no-pager
systemctl status redis --no-pager
systemctl status elasticsearch --no-pager
journalctl -u mysql -n 100 --no-pager
journalctl -u postgresql -n 100 --no-pager
```

## 12.2 Networking commands

```bash
ss -tulpn
ip addr
ip route
ethtool eth0
sar -n TCP,DEV 1
```

## 12.3 Disk and memory commands

```bash
free -h
vmstat 1
iostat -xz 1
lsblk
df -h
```

## 12.4 File permission hygiene

Typical examples:

```bash
chown -R mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql
chown postgres:postgres /var/lib/postgresql/16/main/server.key
chmod 600 /var/lib/postgresql/16/main/server.key
```

## 12.5 Example backup validation checklist

- Verify backup job completed successfully.
- Verify checksum or artifact size expectations.
- Verify backup uploaded to remote storage.
- Perform periodic full restore test.
- Confirm application can read restored data.

## 12.6 Database comparison quick reference

| Engine | Strength | Primary caution |
|---|---|---|
| MySQL/MariaDB | Mature OLTP and web app ecosystem | Careful replication and schema-change planning needed |
| PostgreSQL | Rich SQL, extensions, correctness | Connection pooling often required at scale |
| MongoDB | Flexible documents and scaling patterns | Schema discipline and shard-key design are crucial |
| Redis | Very fast in-memory operations | Memory management and persistence choices matter |
| Elasticsearch | Search and analytics power | Shard/JVM sizing and ops discipline are essential |
| SQLite | Simple embedded database | Limited concurrent writes |

## 12.7 Example production readiness checklist

- Capacity forecast documented.
- Metrics and alerting configured.
- Backup restore tested.
- Security controls reviewed.
- Upgrade path documented.
- Failover tested.
- On-call runbook updated.

---

# 13. Extended MySQL Command Cookbook

## 13.1 Server introspection

```sql
SHOW VARIABLES;
SHOW STATUS;
SHOW FULL PROCESSLIST;
SHOW ENGINE INNODB STATUS\G
```

## 13.2 Session troubleshooting

```sql
SELECT * FROM performance_schema.threads LIMIT 10;
SELECT * FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;
```

## 13.3 Table size report

```sql
SELECT table_schema,
       table_name,
       ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_mb
FROM information_schema.tables
ORDER BY size_mb DESC
LIMIT 20;
```

## 13.4 Index inspection

```sql
SHOW INDEX FROM appdb.customers;
```

## 13.5 Find long-running queries

```sql
SELECT ID, USER, HOST, DB, COMMAND, TIME, STATE, INFO
FROM information_schema.PROCESSLIST
WHERE COMMAND <> 'Sleep'
ORDER BY TIME DESC;
```

## 13.6 Transaction and lock investigation tips

- Review InnoDB status for lock waits.
- Use Performance Schema tables when enabled.
- Correlate with application request IDs.
- Check whether missing indexes are widening lock scope.

## 13.7 Sample `my.cnf` profiles

### Small VM profile

```ini
[mysqld]
innodb_buffer_pool_size = 1G
max_connections = 100
slow_query_log = 1
long_query_time = 1
```

### Medium application server profile

```ini
[mysqld]
innodb_buffer_pool_size = 8G
max_connections = 300
table_open_cache = 4000
thread_cache_size = 100
```

### Large dedicated server profile

```ini
[mysqld]
innodb_buffer_pool_size = 48G
innodb_log_file_size = 4G
max_connections = 500
open_files_limit = 65535
```

## 13.8 Backup media strategy

- Local fast restore copies for immediate recovery.
- Remote encrypted copies for disaster recovery.
- Immutable backup tier where supported.
- Separate retention for daily, weekly, monthly backups.

## 13.9 MySQL replication pre-flight checklist

- Unique `server_id` on every node.
- Binary logging enabled on source.
- Time synchronized with NTP.
- Same major version compatibility validated.
- Firewall open on 3306.
- Replication user tested.
- Backup seeded correctly.

## 13.10 MySQL failover considerations

- Confirm replica fully caught up.
- Freeze application writes if needed.
- Promote chosen replica.
- Redirect clients using proxy or DNS.
- Rebuild old primary as replica.
- Document exact GTID or position state.

---

# 14. Extended PostgreSQL Command Cookbook

## 14.1 Useful psql meta-commands

```sql
\l
\du
\dn
\dt+
\d+ sales.orders
\x
\timing on
```

## 14.2 Database size reports

```sql
SELECT datname, pg_size_pretty(pg_database_size(datname))
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
```

## 14.3 Table and index size reports

```sql
SELECT schemaname,
       relname,
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 20;
```

## 14.4 Cache hit ratio approximation

```sql
SELECT sum(blks_hit) / nullif(sum(blks_hit) + sum(blks_read), 0)::numeric AS cache_hit_ratio
FROM pg_stat_database;
```

## 14.5 Vacuum diagnostics

```sql
SELECT relname,
       last_vacuum,
       last_autovacuum,
       n_dead_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

## 14.6 Connection pressure diagnostics

```sql
SELECT state, count(*)
FROM pg_stat_activity
GROUP BY state
ORDER BY count(*) DESC;
```

## 14.7 WAL generation estimate

```sql
SELECT pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0'));
```

## 14.8 Replication slot review

```sql
SELECT slot_name, slot_type, active, restart_lsn
FROM pg_replication_slots;
```

## 14.9 Postgres tuning heuristics

- Increase `shared_buffers` carefully.
- Keep `work_mem` moderate due to concurrency multiplication.
- Lower `random_page_cost` for SSD.
- Enable `pg_stat_statements` early.
- Use PgBouncer before raising `max_connections` too far.

## 14.10 Failover checklist

- Confirm standby health.
- Check replay lag.
- Promote standby.
- Repoint clients or load balancer.
- Verify writes and backups.
- Recreate replication for old primary.

---

# 15. Extended MongoDB Operations Guide

## 15.1 Replica set health checks

```javascript
rs.status()
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()
```

## 15.2 Database statistics

```javascript
db.stats()
db.serverStatus()
```

## 15.3 Collection stats

```javascript
db.orders.stats()
```

## 15.4 Index review

```javascript
db.orders.getIndexes()
```

## 15.5 Profiling strategy

Use profiling only as needed and with awareness of workload overhead.

Levels:

- 0: off
- 1: slow operations
- 2: all operations

## 15.6 Common admin patterns

- Keep admin database credentials separate from app credentials.
- Prefer replica sets even for small production deployments.
- Avoid arbiters unless you really need them.
- Validate shard rebalancing after capacity changes.

## 15.7 Backup checklist for MongoDB

- Use auth-enabled backup user.
- Take backups from a secondary when possible.
- Ensure oplog coverage if consistent point restore is needed.
- Test restore into isolated environment.

## 15.8 Storage considerations

- Monitor WiredTiger cache utilization.
- Watch filesystem free space.
- Separate journal and data only when justified by platform design.
- Use XFS where vendor guidance recommends it.

## 15.9 Common anti-patterns

- Giant unbounded arrays in one document.
- Missing indexes on shard key prefixes.
- Treating MongoDB like a generic dump for arbitrary blobs.
- Ignoring schema validation when applications evolve.

## 15.10 Schema validation example

```javascript
db.createCollection('customers', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['email', 'name'],
      properties: {
        email: { bsonType: 'string' },
        name: { bsonType: 'string' }
      }
    }
  }
})
```

---

# 16. Extended Redis Operations Guide

## 16.1 Key inspection

```bash
redis-cli DBSIZE
redis-cli INFO memory
redis-cli INFO stats
redis-cli INFO keyspace
```

## 16.2 Slow log review

```bash
redis-cli SLOWLOG LEN
redis-cli SLOWLOG GET 20
```

## 16.3 Latency diagnostics

```bash
redis-cli LATENCY DOCTOR
redis-cli LATENCY LATEST
```

## 16.4 ACL example

```bash
redis-cli ACL SETUSER appuser on >StrongPassword ~app:* +@read +@write -FLUSHALL -FLUSHDB
```

## 16.5 Persistence checklist

- Decide RDB, AOF, or hybrid.
- Measure fsync latency.
- Validate restart time from persistence files.
- Back up persistence files off-host.

## 16.6 Eviction policy decision hints

| Pattern | Policy |
|---|---|
| Pure cache | allkeys-lru or allkeys-lfu |
| Cache with TTL-managed keys | volatile-lru |
| No data loss allowed | noeviction with proper app handling |

## 16.7 Queue design caution

Redis is convenient for queues but not a universal message broker replacement.

Consider:

- Consumer acknowledgment model.
- Persistence needs.
- Ordering guarantees.
- Replay requirements.

## 16.8 Operational anti-patterns

- Running with unlimited memory.
- Exposing instance publicly.
- Storing very large binary blobs.
- Using `KEYS *` in production.
- Forgetting TTLs on cache keys.

## 16.9 Safe key scanning

```bash
redis-cli SCAN 0 MATCH app:* COUNT 100
```

## 16.10 Replication monitoring hints

- Monitor replication offset gaps.
- Track failover event history in Sentinel.
- Confirm client libraries support Sentinel or Cluster discovery.

---

# 17. Extended Elasticsearch Operations Guide

## 17.1 Cat APIs

```bash
curl -u elastic:password -k https://localhost:9200/_cat/health?v
curl -u elastic:password -k https://localhost:9200/_cat/nodes?v
curl -u elastic:password -k https://localhost:9200/_cat/shards?v
curl -u elastic:password -k https://localhost:9200/_cat/indices?v
```

## 17.2 Shard sizing guidance

General advice:

- Avoid many tiny shards.
- Avoid oversized shards that slow relocation and recovery.
- Align shard count with node count and growth patterns.

## 17.3 Bulk indexing example

```bash
curl -u elastic:password -k -X POST https://localhost:9200/_bulk -H 'Content-Type: application/x-ndjson' -d '
{ "index": { "_index": "products", "_id": "1" } }
{ "name": "Laptop", "category": "electronics" }
{ "index": { "_index": "products", "_id": "2" } }
{ "name": "Phone", "category": "electronics" }
'
```

## 17.4 Search performance tips

- Use keyword fields for exact filters and aggregations.
- Avoid expensive wildcard queries on large text fields.
- Use index templates for consistent mappings.
- Tune refresh interval during heavy ingestion.

## 17.5 Snapshot policy ideas

- Hourly snapshots for critical search platforms if RPO requires it.
- Daily retention tiers for older snapshots.
- Replicate snapshot repositories across regions when needed.

## 17.6 Recovery considerations

- Snapshot restore is preferred for disaster recovery.
- Reindex from source systems may be viable for derived search data.
- Maintain mapping templates in version control.

## 17.7 JVM and GC notes

- Monitor old generation pressure.
- Avoid swap.
- Right-size heap and keep room for filesystem cache.
- Review long GC pauses during incidents.

## 17.8 Index template example

```bash
curl -u elastic:password -k -X PUT https://localhost:9200/_index_template/logs-template -H 'Content-Type: application/json' -d '
{
  "index_patterns": ["logs-*"] ,
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "message": { "type": "text" }
      }
    }
  }
}'
```

## 17.9 Common pitfalls

- Treating Elasticsearch as primary source of truth for transactional data.
- Dynamic mappings exploding field counts.
- Oversharding small clusters.
- Ignoring snapshot restore tests.

## 17.10 Rolling restart basics

- Disable shard allocation if appropriate.
- Restart one node at a time.
- Wait for cluster to stabilize.
- Re-enable allocation.
- Verify shard recovery completion.

---

# 18. Database Backups Deep Dive

## 18.1 Backup objectives

Define clearly:

- RPO: how much data loss is acceptable.
- RTO: how fast service must recover.
- Retention: how long backups are kept.
- Immutability: whether backups are protected from deletion.

## 18.2 Full vs incremental vs differential

| Type | Description | Pros | Cons |
|---|---|---|---|
| Full | Entire dataset | Simplest restore | Large backup window |
| Incremental | Changes since last backup | Efficient storage | More complex restore chain |
| Differential | Changes since last full | Simpler than incremental chain | Grows over time |

## 18.3 Logical vs physical backups

| Type | Best for | Cautions |
|---|---|---|
| Logical | Portability and small datasets | Slower restore at scale |
| Physical | Large production systems | Tied more closely to engine/version/filesystem details |

## 18.4 Backup validation checklist

- Backup completed without warnings.
- Checksum validated.
- Artifact is restorable.
- Permissions are correct.
- Encryption is confirmed.
- Offsite copy succeeded.
- Monitoring alert exists for failures.

## 18.5 Retention policy example

- Daily backups: 7 days.
- Weekly backups: 5 weeks.
- Monthly backups: 12 months.
- Yearly backups: as required by policy.

## 18.6 Restore drill template

1. Choose sample backup set.
2. Provision isolated target host.
3. Restore according to runbook.
4. Verify engine starts cleanly.
5. Run integrity checks.
6. Run application smoke tests.
7. Record total recovery time.
8. Update documentation.

## 18.7 Backup storage locations

Options:

- Local disk for rapid operational restore.
- Network-attached storage.
- Object storage.
- Tape or archival cold storage.
- Immutable storage tiers.

## 18.8 Backup encryption workflow ideas

- Encrypt on source before upload.
- Use KMS-integrated storage encryption.
- Rotate encryption keys.
- Restrict decryption rights separately from backup read rights.

## 18.9 Air-gap and ransomware considerations

- Maintain offline or immutable copies.
- Separate backup admin credentials from production credentials.
- Test disaster restore under restricted network assumptions.

## 18.10 Backup monitoring examples

Alert when:

- Latest successful backup age exceeds threshold.
- Restore test has not occurred within policy window.
- Backup size deviates unexpectedly.
- WAL/binlog archival stops.

---

# 19. Database Performance Deep Dive

## 19.1 Performance hierarchy

Typical order of investigation:

1. Bad query pattern.
2. Missing or poor index.
3. Connection saturation.
4. Memory pressure.
5. Storage latency.
6. Network bottleneck.
7. Inefficient schema design.

## 19.2 Metrics that matter

Cross-engine metrics:

- Query latency percentiles.
- Throughput.
- Cache hit ratio.
- Disk read/write latency.
- Replication delay.
- Lock wait time.
- Active sessions.
- Temp file generation.

## 19.3 Benchmarking cautions

- Production-like data shape matters.
- Concurrency matters.
- Cache warm-up matters.
- Synthetic benchmarks can mislead.
- Always compare before and after under same conditions.

## 19.4 Index lifecycle management

Indexes should be:

- Proposed.
- Load tested.
- Measured for write impact.
- Reviewed for redundancy later.
- Dropped if unused and costly.

## 19.5 Query plan literacy

Always ask:

- Is the access path selective?
- Are row estimates accurate?
- Is sorting spilling to disk?
- Is join order sensible?
- Is parallelism helping or hurting?

## 19.6 Host tuning reminders

- Disable swap for Elasticsearch and often for latency-sensitive DB patterns where policy allows.
- Reserve RAM for filesystem cache.
- Use tuned profiles where appropriate.
- Verify scheduler and NUMA considerations on high-end servers.

## 19.7 Application collaboration

Database performance problems are often application problems.

Work jointly on:

- Query batching.
- Retry storm prevention.
- Connection pool sizing.
- Idempotent write design.
- Pagination strategy.

## 19.8 Pagination design

Avoid deep offset pagination for large datasets.

Prefer keyset pagination when possible.

Example idea:

```sql
SELECT * FROM orders
WHERE created_at < '2025-01-01T10:00:00Z'
ORDER BY created_at DESC
LIMIT 50;
```

## 19.9 Hotspot detection

Look for:

- One table dominating writes.
- One shard dominating load.
- One index causing contention.
- One tenant causing disproportionate activity.

## 19.10 Performance review cadence

- Daily: critical latency and saturation alerts.
- Weekly: top query and growth review.
- Monthly: capacity and tuning backlog review.

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

# 21. Database Architecture Patterns

## 21.1 Single primary with replicas

Good for:

- Most transactional systems.
- Clear write path.
- Simple failover models.

Trade-off:

- Writes scale vertically unless sharded or re-architected.

## 21.2 Multi-primary

Good only when justified by workload and conflict model.

Trade-offs:

- Higher operational complexity.
- Conflict handling.
- More careful application semantics.

## 21.3 Shared-nothing sharding

Good for:

- Large scale-out systems.
- Tenant partitioning.
- Very large write volumes.

Trade-offs:

- Application awareness.
- Complex rebalancing.
- Cross-shard queries and transactions may be harder.

## 21.4 CQRS and read replicas

Pattern:

- Primary handles writes.
- Read replicas serve reporting/search/API reads.

Caution:

- Replica lag means stale reads.

## 21.5 Search sidecar pattern

Common design:

- PostgreSQL or MySQL is source of truth.
- Changes are replicated to Elasticsearch for search.

Benefit:

- Each engine serves what it is best at.

## 21.6 Cache-aside pattern

With Redis:

1. App reads cache.
2. On miss, app reads DB.
3. App populates cache.
4. App invalidates or updates cache on write.

Main challenge:

- Cache invalidation correctness.

## 21.7 Event sourcing and append-only logs

Useful when:

- Full audit trail is essential.
- Reconstruction of state matters.

Caution:

- Querying current state usually needs projections or materialized views.

## 21.8 Data tiering

Separate hot and cold data by:

- Partitioning.
- Different storage classes.
- Archival tables or indices.

## 21.9 Blue-green database migration concepts

Common pattern:

- Build new target environment.
- Replicate data.
- Validate.
- Cut over traffic.
- Keep rollback window.

## 21.10 Architecture review checklist

- Are read/write paths explicit?
- Is failure mode understood?
- Is backup/restore aligned to topology?
- Are latency and consistency requirements documented?
- Is cost growth predictable?

---

# 22. Linux Operational Playbooks

## 22.1 Check listening ports

```bash
ss -tulpn | egrep '3306|5432|27017|6379|9200'
```

## 22.2 Check top IO consumers

```bash
iotop -o
```

## 22.3 Check CPU pressure

```bash
mpstat -P ALL 1
pidstat -dur 1
```

## 22.4 Check memory pressure

```bash
free -h
vmstat 1
sar -r 1
```

## 22.5 Check large files

```bash
find /var/lib -type f -size +1G -ls | head
```

## 22.6 Check open file limits

```bash
ulimit -n
cat /proc/$(pgrep -o mysqld)/limits | grep 'open files'
```

## 22.7 Check time sync

```bash
timedatectl
chronyc tracking
```

## 22.8 Firewall examples

```bash
sudo firewall-cmd --list-all
sudo ufw status verbose
```

## 22.9 Capture a support bundle mindset

When investigating incidents, gather:

- Service status.
- Logs.
- Config excerpts.
- Top queries.
- Disk and memory state.
- Replication state.
- Timeline of changes.

## 22.10 Change management reminder

For planned database changes:

- Open change record.
- Define backout plan.
- Notify stakeholders.
- Freeze conflicting work.
- Validate before and after.

---

# 23. Quick Reference Tables

## 23.1 Default ports

| Engine | Default port |
|---|---|
| MySQL/MariaDB | 3306 |
| PostgreSQL | 5432 |
| MongoDB | 27017 |
| Redis | 6379 |
| Elasticsearch | 9200 HTTP, 9300 transport |
| SQLite | N/A |

## 23.2 Key config files

| Engine | Main config |
|---|---|
| MySQL | /etc/mysql/my.cnf or /etc/my.cnf |
| PostgreSQL | postgresql.conf, pg_hba.conf |
| MongoDB | /etc/mongod.conf |
| Redis | /etc/redis/redis.conf |
| Elasticsearch | /etc/elasticsearch/elasticsearch.yml |
| SQLite | Application-defined file |

## 23.3 Backup tools

| Engine | Common tools |
|---|---|
| MySQL | mysqldump, mysqlpump, xtrabackup |
| PostgreSQL | pg_dump, pg_basebackup, pgBackRest |
| MongoDB | mongodump, filesystem snapshots |
| Redis | RDB/AOF copy with coordination |
| Elasticsearch | Snapshot API |
| SQLite | .backup, .dump |

## 23.4 HA tools

| Engine | Common HA options |
|---|---|
| MySQL | InnoDB Cluster, Galera, ProxySQL |
| PostgreSQL | Patroni, etcd, pgpool-II, HAProxy |
| MongoDB | Replica sets, sharding |
| Redis | Sentinel, Redis Cluster |
| Elasticsearch | Native cluster redundancy |
| SQLite | Usually application-level redundancy |

---

# 24. Sample Runbooks

## 24.1 MySQL restore runbook summary

1. Provision host.
2. Install same compatible version.
3. Stop service.
4. Restore logical or physical backup.
5. Apply binlogs if required.
6. Start service.
7. Validate data.
8. Repoint application if part of failover.

## 24.2 PostgreSQL restore runbook summary

1. Provision host and storage.
2. Restore base backup.
3. Configure WAL archive access.
4. Set recovery target.
5. Start instance.
6. Validate recovery completion.
7. Promote if needed.

## 24.3 MongoDB node replacement summary

1. Confirm replica set health.
2. Remove failed member if necessary.
3. Rebuild node.
4. Rejoin member.
5. Wait for initial sync.
6. Validate replication lag clears.

## 24.4 Redis failover summary

1. Confirm primary failure.
2. Validate Sentinel or Cluster election result.
3. Ensure clients reconnect correctly.
4. Check persistence state.
5. Rebuild failed node as replica.

## 24.5 Elasticsearch red cluster summary

1. Check cluster health.
2. Identify unassigned shards.
3. Confirm disk watermark status.
4. Check failed nodes.
5. Restore capacity or fix allocation blockers.
6. Consider snapshot restore only after diagnosis.

---

# 25. Practical Examples

## 25.1 Example: designing a web app stack

Recommended stack:

- PostgreSQL for transactional source of truth.
- Redis for caching and sessions.
- Elasticsearch for search.
- PgBouncer for pooling.
- Patroni for PostgreSQL HA.

Why:

- Strong relational core.
- Fast cache layer.
- Dedicated search engine.
- Operationally mature components.

## 25.2 Example: content platform with flexible schema

Recommended stack:

- MongoDB for primary content documents.
- Redis for cache.
- Elasticsearch for search and discovery.

Watchouts:

- Enforce schema validation.
- Design shard key early if growth is expected.
- Keep search indexing pipeline reliable.

## 25.3 Example: small internal utility

Recommended stack:

- SQLite for local persistence.

Why:

- Minimal operational overhead.
- Simple deployment.
- Adequate for single-user or low-write concurrency.

## 25.4 Example: high-read ecommerce platform

Recommended stack:

- MySQL or PostgreSQL primary.
- Read replicas.
- Redis cache.
- Elasticsearch product search.

Operational notes:

- Watch replica lag.
- Cache hot product pages.
- Keep search index rebuild strategy ready.

---

# 26. Final Recommendations

## 26.1 Start simple

The simplest architecture that meets requirements is usually the best first production architecture.

## 26.2 Prefer operational maturity over novelty

A team that deeply understands PostgreSQL or MySQL will often outperform a more exotic system deployed without deep expertise.

## 26.3 Backups are not real until restored

Always verify recoverability.

## 26.4 Security is part of administration

Database admin work includes access control, encryption, and auditability, not just uptime.

## 26.5 Performance is end-to-end

Query design, schema design, application behavior, Linux tuning, and storage quality all matter.

## 26.6 Measure continuously

Use dashboards, logs, slow query analysis, and periodic reviews.

## 26.7 Document everything important

Good runbooks reduce outage duration and upgrade risk.

---

# 27. Line-Filler Knowledge Cards for Operational Review

> The following cards intentionally provide compact, review-friendly operational reminders to make this guide comprehensive and reference-dense.

## 27.1 Card 001

- Prefer vendor-supported repositories for production upgrades.
- Keep version pinning explicit.
- Rehearse rollback before major version change.

## 27.2 Card 002

- Use dedicated backup users.
- Limit privileges to backup-related actions.
- Protect credential files with strict permissions.

## 27.3 Card 003

- Monitor disk latency, not only throughput.
- Databases can fail under latency spikes even when bandwidth seems fine.
- Storage tail latency matters.

## 27.4 Card 004

- Time sync matters for logs, certificates, and replication analysis.
- Run chrony or equivalent.
- Alert on drift.

## 27.5 Card 005

- Name databases, schemas, tables, and indexes consistently.
- Standard naming reduces troubleshooting mistakes.
- Conventions save time.

## 27.6 Card 006

- Retain enough WAL/binlog for recovery.
- Monitor archive success continuously.
- Restore drills should include replay.

## 27.7 Card 007

- Avoid manual changes on replicas without a runbook.
- Divergence is easy to create and painful to repair.
- Prefer declarative automation.

## 27.8 Card 008

- Track schema migration duration historically.
- Large-table changes can grow non-linearly.
- Treat them like risky production events.

## 27.9 Card 009

- Keep application connection pools bounded.
- Unlimited client pools can overload the server instantly.
- Pooling needs capacity planning too.

## 27.10 Card 010

- Review dead letter or failed ingestion pipelines feeding search systems.
- Search freshness issues are often pipeline issues, not engine bugs.
- Trace end-to-end.

## 27.11 Card 011

- Maintain separate dashboards for host metrics and query metrics.
- Both views are necessary.
- One without the other can mislead.

## 27.12 Card 012

- Validate filesystem ownership after physical restore.
- Many restore failures are simple permission problems.
- Fix ownership before restart.

## 27.13 Card 013

- Write-heavy workloads need careful checkpoint and log tuning.
- Measure before changing defaults.
- Durability trade-offs must be approved.

## 27.14 Card 014

- Compression reduces backup storage but can increase restore CPU time.
- Pick settings aligned to RTO goals.
- Balance cost and speed.

## 27.15 Card 015

- Document where certificates live on disk.
- Expired TLS certs can create avoidable outages.
- Track renewal ownership.

## 27.16 Card 016

- Avoid app-side timezone ambiguity.
- Store UTC where practical.
- Convert at presentation layer.

## 27.17 Card 017

- Use separate monitoring credentials.
- Read-only visibility is usually enough.
- Avoid over-privileged observability agents.

## 27.18 Card 018

- Large deletes can be operationally expensive.
- Prefer partition drops or chunked deletes.
- Control bloat and replication lag.

## 27.19 Card 019

- Always know your biggest tables and indexes.
- Capacity surprises often come from a few objects.
- Rank them regularly.

## 27.20 Card 020

- Validate application retry behavior during failover.
- Good infrastructure fails gracefully only with good clients.
- Chaos testing helps.

## 27.21 Card 021

- Monitor for backup size anomalies.
- Sudden drops can indicate incomplete dumps.
- Sudden growth can indicate data explosion.

## 27.22 Card 022

- Tune autovacuum before wraparound becomes urgent.
- Preventive work is cheaper than emergency response.
- Watch dead tuples early.

## 27.23 Card 023

- In Redis, define what data may be lost.
- Cache data and durable data deserve different persistence choices.
- Align policy to business needs.

## 27.24 Card 024

- In MongoDB, validate schema even if using a flexible document model.
- Flexibility without guardrails becomes debt.
- Enforce critical fields.

## 27.25 Card 025

- In Elasticsearch, map fields intentionally.
- Dynamic mapping can explode field counts.
- Prevent cluster state bloat.

## 27.26 Card 026

- Prefer idempotent migration scripts when possible.
- Safe re-runs reduce deployment risk.
- CI should catch drift.

## 27.27 Card 027

- Separate admin traffic from app traffic when feasible.
- Incident access should not compete with production load.
- Bastions help.

## 27.28 Card 028

- Avoid large transactions during peak load.
- They increase lock duration, WAL/binlog churn, and recovery work.
- Batch safely.

## 27.29 Card 029

- Keep a dependency inventory of backup, monitoring, and failover tools.
- Tool drift causes silent operational risk.
- Update alongside engine upgrades.

## 27.30 Card 030

- Track restore time as a first-class SLO input.
- Backup frequency alone is not enough.
- Recovery speed matters.

## 27.31 Card 031

- Capture query fingerprints, not just raw statements.
- Fingerprints group similar workload classes.
- This improves prioritization.

## 27.32 Card 032

- Review replication slots or equivalent retention mechanisms.
- Stalled consumers can fill disks.
- Alert on backlog growth.

## 27.33 Card 033

- Use canary queries in monitoring.
- They reveal latency from the application point of view.
- Synthetic probes are valuable.

## 27.34 Card 034

- Keep emergency free space on data volumes.
- Zero-free-space incidents are harder to recover from.
- Reserve headroom operationally.

## 27.35 Card 035

- Benchmark with realistic concurrency.
- Single-user results rarely predict production behavior.
- Use production-like query mixes.

## 27.36 Card 036

- Store infrastructure-as-code for database-adjacent automation.
- Manual topology drift is dangerous.
- Version control runbooks and configs.

## 27.37 Card 037

- Observe queue depth in storage metrics.
- Rising await plus queue depth signals saturation.
- Application latency usually follows.

## 27.38 Card 038

- Keep Linux kernel, filesystem, and database versions in compatibility review.
- Ops problems can sit at boundaries.
- Test the full stack.

## 27.39 Card 039

- For high connection counts, investigate pooling before scaling database backends.
- Backend process overhead is real.
- Pooling is often the simplest win.

## 27.40 Card 040

- Validate failover DNS TTL behavior.
- Routing changes are only as fast as clients respect them.
- Proxy layers reduce surprises.

## 27.41 Card 041

- Rehearse certificate rotation in non-production.
- Hot-reload behavior differs by engine.
- Avoid expiry-day improvisation.

## 27.42 Card 042

- Use separate users for schema migrations and applications.
- Apps do not need DDL in steady state.
- Reduce blast radius.

## 27.43 Card 043

- Review crash recovery duration after checkpoints and log tuning changes.
- Faster steady-state writes can slow recovery.
- Balance both objectives.

## 27.44 Card 044

- Treat search indexes as rebuildable if architecture permits.
- This can simplify backup priorities.
- But document rebuild time.

## 27.45 Card 045

- Protect snapshot repositories and backup buckets with strict IAM.
- Backup theft is a real data breach vector.
- Logs alone are insufficient.

## 27.46 Card 046

- Avoid world-readable config files.
- Secrets often leak through permissive defaults.
- Audit file modes regularly.

## 27.47 Card 047

- Track top 10 queries by total time and by mean latency.
- The rankings differ and both matter.
- Optimize accordingly.

## 27.48 Card 048

- Use read replicas for read scaling, not as a substitute for query tuning.
- Inefficient queries stay inefficient.
- They just move the pain.

## 27.49 Card 049

- When using containers, understand the storage driver and PVC semantics.
- Databases are sensitive to hidden storage behavior.
- Benchmark the actual platform.

## 27.50 Card 050

- Keep a restoration target matrix.
- Know which system restores where and how fast.
- Incidents reward preparation.

## 27.51 Card 051

- Track schema ownership by team.
- Orphaned tables and indexes accumulate quickly.
- Ownership drives cleanup.

## 27.52 Card 052

- Observe lock wait graphs during traffic peaks.
- Some contention is time-window dependent.
- Sample at the right times.

## 27.53 Card 053

- Prefer append-only event retention for auditing sensitive changes.
- Mutable audit trails are weak controls.
- Integrity matters.

## 27.54 Card 054

- Review high-cardinality metrics carefully.
- Observability tooling itself can become noisy or expensive.
- Keep labels disciplined.

## 27.55 Card 055

- Consider tenant isolation requirements early.
- Shared schema, separate schema, and separate database models differ significantly.
- Migrations and backup scope change too.

## 27.56 Card 056

- Runbook steps should include validation checks after every destructive action.
- Action without verification is risky.
- Write explicit success criteria.

## 27.57 Card 057

- Audit dormant privileged accounts.
- Old admin accounts are common risk.
- Remove or rotate regularly.

## 27.58 Card 058

- Protect monitoring endpoints and exporters.
- They can leak topology and performance data.
- Security includes observability.

## 27.59 Card 059

- Review biggest temp-file generators.
- Sort and hash spills are performance signals.
- They often point to missing indexes or memory mis-sizing.

## 27.60 Card 060

- Use staged rollouts for driver upgrades.
- Client behavior changes can impact pooling, TLS, and retries.
- Test real traffic patterns.

## 27.61 Card 061

- Document replication topology clearly.
- Human misunderstanding during incidents causes secondary outages.
- Diagrams matter.

## 27.62 Card 062

- For SQLite, ensure filesystem semantics on network storage are understood.
- Not all network filesystems behave safely.
- Local disks are simpler.

## 27.63 Card 063

- Review kernel dmesg during storage incidents.
- Filesystem and block device warnings can explain database stalls.
- Look below the engine.

## 27.64 Card 064

- Monitor CPU steal time in virtualized environments.
- Neighbor noise can look like database slowness.
- Host context matters.

## 27.65 Card 065

- When tuning caches, leave room for OS cache and auxiliary processes.
- 100 percent RAM allocation plans usually age badly.
- Keep headroom.

## 27.66 Card 066

- Version control configuration snippets.
- Drifted tuning changes are hard to explain months later.
- Commit them intentionally.

## 27.67 Card 067

- Record exact build and package versions during incidents.
- Reproducing bugs requires precise versions.
- Details matter.

## 27.68 Card 068

- For replication lag, distinguish network lag from apply lag.
- Different causes need different fixes.
- Inspect both sides.

## 27.69 Card 069

- Review table and index bloat trends, not only point-in-time sizes.
- Growth rate reveals maintenance issues.
- Trending beats snapshots.

## 27.70 Card 070

- Keep a tested emergency credential access process.
- Break-glass access should be controlled and auditable.
- Do not improvise during outages.

## 27.71 Card 071

- Avoid changing too many tuning variables at once.
- One change at a time preserves learning.
- Measure effects clearly.

## 27.72 Card 072

- Query timeouts protect systems from runaway workload.
- Apply at application and database layers where appropriate.
- Default forever is risky.

## 27.73 Card 073

- Monitor redo/WAL/binlog generation rate during launches.
- It predicts storage and backup pressure.
- Growth events often surprise teams.

## 27.74 Card 074

- Use prepared statements or server-side plan caches carefully.
- They improve efficiency but can have plan stability effects.
- Measure workload impact.

## 27.75 Card 075

- Keep failover authority clear.
- Ambiguous ownership slows incidents.
- Define who can promote, cut over, and rollback.

## 27.76 Card 076

- Review compression settings for hot paths.
- CPU savings and storage savings can trade off against latency.
- Align to workload priorities.

## 27.77 Card 077

- Protect against thundering herds after cache expiry.
- Use jitter, soft TTLs, or request coalescing.
- Databases feel cache mistakes immediately.

## 27.78 Card 078

- Confirm backup tools support engine version features you use.
- Tool compatibility can lag.
- Upgrade both intentionally.

## 27.79 Card 079

- Record primary keys and unique constraints as business invariants, not just technical features.
- They encode correctness.
- Preserve them in migrations.

## 27.80 Card 080

- Review optimizer statistics freshness.
- Bad stats lead to bad plans.
- Maintenance matters.

## 27.81 Card 081

- Use maintenance windows for disruptive operations when possible.
- Online does not always mean impact-free.
- Communicate risk.

## 27.82 Card 082

- When troubleshooting, separate symptom metrics from causal metrics.
- High CPU might be cause or effect.
- Build a timeline.

## 27.83 Card 083

- Keep application and database clocks aligned for time-based pagination and conflict detection.
- Time drift breaks logic.
- Sync matters beyond logging.

## 27.84 Card 084

- Review backup retention compliance after storage class changes.
- Cheap storage migrations can break governance.
- Validate policy implementation.

## 27.85 Card 085

- Prefer explicit application schemas over dumping everything into public or default spaces.
- Structure improves privilege management.
- Organization helps long-term maintenance.

## 27.86 Card 086

- Capture baseline performance before major releases.
- Without baseline, regressions are harder to prove.
- Benchmark routinely.

## 27.87 Card 087

- For document stores, version document schemas explicitly when applications evolve rapidly.
- Migration clarity improves safety.
- Flexibility still needs governance.

## 27.88 Card 088

- Review TLS protocol and cipher policies periodically.
- Security baselines change.
- Keep pace with standards.

## 27.89 Card 089

- Ensure your monitoring covers unsuccessful backups, not only successful ones.
- Silence is dangerous.
- Alert on absence too.

## 27.90 Card 090

- Keep a map of data classification by database and schema.
- Security controls depend on knowing where sensitive data lives.
- Inventory first.

## 27.91 Card 091

- Watch for retry storms after brief outages.
- Recovery traffic can extend incidents.
- Rate-limit or circuit-break when needed.

## 27.92 Card 092

- Log rotation must account for engine-specific reopen behavior.
- Coordinate with service reloads if required.
- Avoid losing logs.

## 27.93 Card 093

- Validate read-after-write expectations when using replicas.
- Some application paths need primary reads.
- Consistency choices are application features.

## 27.94 Card 094

- Prefer automation for recurring failover and backup tasks, but keep manual runbooks current.
- Automation also fails.
- Humans need a fallback path.

## 27.95 Card 095

- Check memory overcommit and swap policy for latency-sensitive services.
- Linux VM settings can influence failure modes.
- Understand host defaults.

## 27.96 Card 096

- Use `EXPLAIN` before and after schema changes that affect critical queries.
- Plans can regress unexpectedly.
- Validate hot paths.

## 27.97 Card 097

- For search clusters, separate ingest-heavy and query-heavy roles when scale justifies it.
- Workload isolation can improve stability.
- Architecture should match traffic.

## 27.98 Card 098

- During restores, verify ownership, permissions, SELinux context, and service unit expectations.
- Startup failures are often environmental.
- Linux details matter.

## 27.99 Card 099

- Review and prune obsolete indexes periodically.
- Old indexes quietly tax write performance.
- Keep the set intentional.

## 27.100 Card 100

- Every database team should know its last successful restore test date.
- If nobody knows, the risk is already too high.
- Put it on the dashboard.

---

# 28. Closing Notes

This guide emphasizes Linux-based operational excellence across major database families.

Use it as:

- A learning resource.
- A production checklist base.
- A runbook starting point.
- A platform architecture reference.

Recommended next steps for teams:

1. Standardize backup and restore drills.
2. Standardize observability dashboards.
3. Standardize least-privilege access patterns.
4. Standardize failover and upgrade runbooks.
5. Review whether each workload is on the right database.

End of guide.
