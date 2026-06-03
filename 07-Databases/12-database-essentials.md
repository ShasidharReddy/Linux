# 📘 Database Essentials: From Basics to Advanced Operations

> A Linux-first, operations-friendly database guide that moves from first principles to production-ready administration.
> Scope: architecture, database selection, connectivity, remote access, backups, monitoring, security, containers, and high availability.

---

## 🗺️ How to Use This Guide

- Read Sections 1 and 2 if you are new to databases.
- Jump to Section 3 when you need install, startup, and connection commands.
- Use Section 4 before exposing any database to a network.
- Use Section 5 for day-2 operations such as backup, monitoring, and replication.
- Use Section 6 when hardening a production deployment.
- Use Section 7 for containerized local labs and development stacks.
- Bookmark Section 8 for quick lookups during incidents or interviews.

## 📚 Table of Contents

1. [What Are Databases?](#1-what-are-databases)
2. [Types of Databases](#2-types-of-databases)
3. [How to Connect to Major Databases](#3-how-to-connect-to-major-databases)
4. [Remote Access Configuration](#4-remote-access-configuration)
5. [Database Administration Essentials](#5-database-administration-essentials)
6. [Database Security](#6-database-security)
7. [Databases in Containers](#7-databases-in-containers)
8. [Quick Reference Tables](#8-quick-reference-tables)
9. [Real-World Scenarios](#9-real-world-scenarios)
10. [Checklists and Common Mistakes](#10-checklists-and-common-mistakes)
11. [Operational Appendices](#11-operational-appendices)
12. [Glossary](#12-glossary)
13. [Hands-On Practice Labs](#13-hands-on-practice-labs)

---

## 1. What Are Databases?

### 1.1 Definition and purpose

A database is an organized system for storing, retrieving, updating, and protecting data.

A database management system (DBMS) provides the software layer that helps applications:
- Store information in a structured or semi-structured form.
- Query data efficiently without reading entire files by hand.
- Protect data integrity through constraints, transactions, and permissions.
- Handle many users and workloads concurrently.
- Recover from failures using logs, checkpoints, backups, or replicas.
- Scale from a laptop demo to a multi-node production deployment.

Without databases, teams often end up building fragile file-based systems with weak validation, poor concurrency control, and no reliable recovery path.

### 1.2 Why databases matter

1. They keep critical business data durable beyond process restarts and machine crashes.
2. They provide predictable query patterns for applications, analytics, and operations.
3. They allow multiple services and users to share the same source of truth safely.
4. They reduce application complexity by moving indexing, locking, filtering, sorting, and aggregation into a dedicated engine.
5. They provide guardrails such as unique keys, foreign keys, schema validation, ACLs, and audit trails.
6. They make disaster recovery possible because data and metadata can be backed up, replicated, and restored consistently.

### 1.3 History and evolution

Database technology evolved in waves as storage media, hardware economics, and application needs changed.

```mermaid
timeline
    title Database evolution at a glance
    1960s : Hierarchical and network databases
    1970s : Relational model formalized by E. F. Codd
    1980s : Commercial RDBMS adoption expands
    1990s : Client/server SQL systems dominate enterprises
    2000s : Web scale drives NoSQL experimentation
    2010s : Cloud-native distributed databases and NewSQL rise
    2020s : Multi-model, serverless, vector, and globally distributed platforms mature
```

### Key milestones
- **Hierarchical systems** stored data in tree-like structures and worked well for rigid parent-child records.
- **Network databases** expanded relationships but still required application logic to understand storage paths.
- **Relational databases** popularized tables, SQL, declarative querying, and normalized design.
- **Data warehousing** introduced columnar analytics engines and large-scale reporting patterns.
- **NoSQL systems** addressed flexible schemas, extreme write volume, distributed storage, and specialized access patterns.
- **NewSQL systems** attempted to combine SQL and strong transactions with horizontal scalability.
- **Cloud platforms** added managed backups, autoscaling, global replication, and API-driven operations.

### 1.4 Core database responsibilities

| Responsibility | What it means in practice | Example |
|---|---|---|
| Storage | Persisting data on disk or memory with an engine-specific format | InnoDB pages, PostgreSQL heap files, Redis in-memory structures |
| Indexing | Building data structures that make reads fast | B-tree on customer_id |
| Concurrency control | Allowing many sessions to work safely at once | MVCC in PostgreSQL |
| Transactions | Grouping related writes into a unit of work | Bank transfer of debit and credit |
| Security | Controlling who can connect and what they can do | Read-only analyst role |
| Recovery | Replaying logs or restoring from backups after failure | WAL replay after crash |
| Replication | Sending changes to replicas for scale or resilience | Primary to standby streaming |
| Optimization | Choosing efficient plans for queries | Index scan instead of full table scan |

### 1.5 ACID properties explained with examples

ACID describes transaction guarantees commonly associated with relational systems and some distributed databases.

```mermaid
sequenceDiagram
    autonumber
    participant App as Application
    participant DB as Database
    participant Log as Commit Log
    App->>DB: BEGIN
    App->>DB: Debit account A by 100
    App->>DB: Credit account B by 100
    DB->>Log: Flush commit record
    Log-->>DB: Commit durable
    DB-->>App: COMMIT OK
```

#### Atomicity

Atomicity means a transaction is all-or-nothing.

**Bank transfer example:**
- Step 1: subtract $100 from checking account A.
- Step 2: add $100 to savings account B.
- If step 2 fails, step 1 must also be rolled back.
- The database should never leave money “lost in transit.”

#### Consistency

Consistency means every committed transaction moves the database from one valid state to another valid state.

**Order system example:**
- An order row cannot reference a non-existent customer if a foreign key exists.
- A username marked UNIQUE cannot be inserted twice.
- A CHECK constraint can prevent a negative quantity for inventory_on_hand.

#### Isolation

Isolation means concurrent transactions should not see each other in unsafe ways.

| Isolation level | What it prevents | Typical risk that may remain | Example |
|---|---|---|---|
| Read Uncommitted | Almost nothing | Dirty reads | Session B sees uncommitted price change from Session A |
| Read Committed | Dirty reads | Non-repeatable reads | Row value changes between two SELECTs |
| Repeatable Read | Dirty and many non-repeatable reads | Phantoms depending on engine | New matching rows appear in a range query |
| Serializable | Most anomalies | Lower concurrency and more retries | Concurrent booking workflow turns into ordered transactions |

**Practical example:** two workers attempting to reserve the last available seat should not both succeed.

#### Durability

Durability means committed data survives crashes according to the system design.

**Practical example:**
- A checkout transaction is committed.
- The server loses power five seconds later.
- After restart, the committed order should still exist because the commit record was flushed to durable storage or replicated safely.

### 1.6 ACID in SQL example

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 10;
UPDATE accounts SET balance = balance + 100 WHERE id = 20;
INSERT INTO transfers(from_account, to_account, amount, created_at)
VALUES (10, 20, 100, NOW());
COMMIT;
```

If any statement fails, the application should issue `ROLLBACK;` so partial state is not committed.

### 1.7 CAP theorem explained

CAP theorem describes trade-offs in distributed systems when a network partition occurs.

- **Consistency (C):** every read receives the latest write or an error.
- **Availability (A):** every request receives a non-error response, even if it may not reflect the newest data.
- **Partition tolerance (P):** the system continues operating even when network links fail between nodes.

In real distributed systems, partition tolerance is usually mandatory, so the practical choice during a partition is often between stronger consistency and higher availability.

```mermaid
flowchart TD
    P[Partition happens between nodes]
    P --> C[Choose stronger consistency]
    P --> A[Choose higher availability]
    C --> C1[Some requests fail or block]
    C --> C2[Clients avoid stale reads]
    A --> A1[Requests still succeed]
    A --> A2[Clients may see stale data]
    C1 --> CP[CP-oriented systems]
    A1 --> AP[AP-oriented systems]
    P --> CA[No partition? Many systems deliver both C and A locally]
```

### 1.8 CAP examples by workload

| Workload | Usually favors | Why |
|---|---|---|
| Bank ledger | Consistency | Incorrect balances are worse than temporary unavailability |
| Shopping cart cache | Availability | A stale cart is tolerable for a short time |
| Global configuration store | Consistency | Split-brain config can break systems |
| Social feed counters | Availability | Slight lag is acceptable if the app keeps serving users |
| Distributed SQL checkout flow | Consistency | Orders and payments must line up exactly |

### 1.9 Databases vs files vs object storage

| System | Best at | Weakness |
|---|---|---|
| Flat files | Small local configs and ad hoc exports | Poor concurrency and indexing |
| Object storage | Large blobs, backups, media, archives | Not a transactional query engine |
| Database | Structured retrieval, updates, integrity, concurrent workloads | Operational overhead compared with plain files |

### 1.10 Summary of the fundamentals

- Databases exist to provide durable, queryable, multi-user data management.
- Relational systems emphasize integrity and SQL power.
- NoSQL systems emphasize flexible models and specialized workloads.
- Distributed databases force trade-offs around consistency, availability, and partitions.
- Operational quality matters as much as schema design: backups, monitoring, and security are part of the database story.

---

## 2. Types of Databases

### 2.1 Relational databases (RDBMS)

Relational databases store data in tables made of rows and columns, with schemas, keys, joins, and transactions.

- **MySQL** is common for web applications, SaaS platforms, and general OLTP.
- **PostgreSQL** is favored when SQL features, extensibility, and analytical richness matter.
- **MariaDB** is a MySQL-family engine with community-driven features and compatible tooling in many cases.
- **Oracle Database** dominates some large enterprise and legacy environments.
- **SQL Server** is strong in Microsoft-centric shops and hybrid enterprise estates.

### 2.2 NoSQL databases

NoSQL is not one product category. It includes multiple specialized models.

#### Document databases

- Store JSON-like documents.
- Useful when application objects do not fit cleanly into many normalized tables.
- Example: MongoDB.

#### Key-value stores

- Store values by key with extremely fast lookup.
- Great for caching, sessions, rate limiting, counters, and queues.
- Example: Redis.

#### Column-family / wide-column stores

- Designed for massive scale-out writes and large distributed datasets.
- Optimize around partition keys and denormalized access paths.
- Example: Cassandra.

#### Graph databases

- Model nodes, edges, and properties.
- Good for fraud graphs, social relationships, recommendations, and network maps.
- Example: Neo4j.

### 2.3 NewSQL databases

- **CockroachDB** offers distributed SQL with strong consistency and PostgreSQL-compatible tooling for many workflows.
- **TiDB** provides MySQL-compatible distributed SQL with separation of compute and storage.
- NewSQL systems target globally distributed or horizontally scalable transactional workloads without abandoning SQL.

### 2.4 Time-series databases

- **InfluxDB** is designed for timestamped metrics, events, and IoT workloads.
- **TimescaleDB** builds time-series capabilities on top of PostgreSQL.
- Time-series systems optimize ingestion, retention, downsampling, and time-window analytics.

### 2.5 Search engines

- **Elasticsearch** powers full-text search, observability, security analytics, and log indexing.
- **Solr** is another Lucene-based search platform used for enterprise search and content indexing.
- Search engines prioritize inverted indexes, text relevance, faceting, and flexible query DSLs.

### 2.6 Comparison table of major database types

| Type | Examples | Data model | Strengths | Trade-offs | Common use cases |
|---|---|---|---|---|---|
| Relational | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | Tables and relations | ACID, SQL, joins, constraints | Schema changes require planning | OLTP, ERP, billing, inventory |
| Document | MongoDB | JSON/BSON documents | Flexible schema, developer-friendly objects | Joins are weaker or avoided | Catalogs, profiles, content |
| Key-Value | Redis | Key to value | Very fast, simple access, cache-friendly | Limited ad hoc querying | Caching, sessions, queues |
| Column-Family | Cassandra | Partitioned wide rows | High write throughput, scale-out | Query model must be designed up front | Events, telemetry, large-scale writes |
| Graph | Neo4j | Nodes and edges | Relationship traversals are natural | Not ideal for every OLTP workload | Fraud, identity, network topology |
| NewSQL | CockroachDB, TiDB | Relational and distributed | SQL plus horizontal scale | Operational complexity and cost | Global SaaS, distributed transactions |
| Time-Series | InfluxDB, TimescaleDB | Timestamp-centric | Retention, compression, rollups | Specialized around time data | Metrics, IoT, monitoring |
| Search | Elasticsearch, Solr | Documents plus inverted index | Full-text search and analytics | Not a drop-in transactional source of truth | Search boxes, logs, SIEM |

### 2.7 Choosing the right database

Ask these questions before selecting a technology:
1. Is the source of truth highly structured with strong consistency requirements?
2. Do you need flexible schema and document-shaped records?
3. Is the workload mostly cache lookups or ephemeral state?
4. Do you need full-text relevance scoring or faceted search?
5. Will you write time-stamped events continuously and query by time windows?
6. Do you need distributed SQL across regions with ACID guarantees?
7. What is the operational skill level of the team running the platform?

### 2.8 Decision tree: when to use which type

```mermaid
flowchart TD
    A[Start with workload] --> B{Need multi-row transactions and joins?}
    B -->|Yes| C{Need horizontal scale across nodes?}
    C -->|No| D[RDBMS: MySQL, PostgreSQL, MariaDB]
    C -->|Yes| E[NewSQL: CockroachDB, TiDB]
    B -->|No| F{Primary access pattern?}
    F -->|Flexible JSON documents| G[Document DB: MongoDB]
    F -->|Simple key lookup, cache, session| H[Key-Value DB: Redis]
    F -->|Massive write throughput by partition key| I[Column-Family DB: Cassandra]
    F -->|Relationship traversal| J[Graph DB: Neo4j]
    F -->|Time-window analytics| K[Time-Series DB: InfluxDB or TimescaleDB]
    F -->|Full-text relevance and faceting| L[Search Engine: Elasticsearch or Solr]
```

### 2.9 Quick selection matrix

| If you need... | Prefer... | Because... |
|---|---|---|
| Financial correctness and referential integrity | PostgreSQL or MySQL | ACID and mature relational tooling |
| A flexible product catalog with changing attributes | MongoDB | Documents absorb schema variation well |
| Microsecond cache lookups and short-lived data | Redis | In-memory key-value design |
| Global SQL writes with scale-out | CockroachDB or TiDB | Distributed SQL |
| Metrics retention and downsampling | InfluxDB or TimescaleDB | Time-centric storage and queries |
| Search relevance, autocomplete, and faceting | Elasticsearch or Solr | Inverted index and DSL |
| Graph traversals such as “friends of friends” | Neo4j | Edges are first-class citizens |

### 2.10 Anti-patterns to avoid

- Using Redis as the only persistent source of truth for critical financial data without careful durability design.
- Using Elasticsearch as the only write path for transactional records that need strict consistency.
- Choosing Cassandra before the team understands partition-key-led query design.
- Forcing graph problems into a relational schema when deep traversals dominate latency.
- Choosing distributed SQL when a single-node PostgreSQL instance would easily meet scale and simplicity requirements.

---

## 3. How to Connect to Major Databases

This section focuses on six common engines used on Linux machines and development laptops: MySQL/MariaDB, PostgreSQL, MongoDB, Redis, Elasticsearch, and SQLite.

### 3.1 🐬 MySQL / MariaDB

**Category:** Relational (RDBMS)
**Default port:** 3306

#### Best fit
- Web applications
- General OLTP
- LAMP stacks
- Read replicas for scale-out reads

#### Installation steps on Ubuntu / Debian
```bash
sudo apt update
sudo apt install -y mysql-server mysql-client
# MariaDB alternative
sudo apt install -y mariadb-server mariadb-client
```

#### Installation steps on RHEL / Rocky / AlmaLinux
```bash
sudo dnf install -y mysql-server
# MariaDB alternative
sudo dnf install -y mariadb-server
```

#### Installation steps on macOS with Homebrew
```bash
brew install mysql
# or
brew install mariadb
```

#### Starting the service
```bash
sudo systemctl enable --now mysql
# MariaDB often uses the service name mariadb
sudo systemctl enable --now mariadb
sudo systemctl status mysql || sudo systemctl status mariadb
```

#### Local connection commands
```bash
mysql -u root -p
mysql -u appuser -p appdb
mysql --protocol=TCP -h 127.0.0.1 -P 3306 -u appuser -p
```

#### Remote connection setup
1. Edit `/etc/mysql/mysql.conf.d/mysqld.cnf`, `/etc/my.cnf`, or the MariaDB equivalent.
2. Set `bind-address = 0.0.0.0` or a specific private IP, not a public interface unless required.
3. Create a network-scoped user such as `appuser@10.10.%` instead of `appuser@%` when possible.
4. Require TLS for remote clients in production.
5. Open TCP 3306 only from approved application subnets.

#### Example configuration
```conf
[mysqld]
bind-address = 0.0.0.0
require_secure_transport = ON
skip_name_resolve = ON
```

#### Connection string formats
- `mysql -h db.example.com -P 3306 -u appuser -p appdb`
- `mysql://appuser:password@db.example.com:3306/appdb`
- `jdbc:mysql://db.example.com:3306/appdb?useSSL=true`
- `mysql+pymysql://appuser:password@db.example.com:3306/appdb`

#### GUI tools
- MySQL Workbench
- DBeaver
- DataGrip
- TablePlus

#### Common connection issues
- Authentication plugin mismatch between client and server.
- Trying to log in as `root` remotely when root is local-only.
- Firewall open on host but blocked by cloud security group.
- TLS required on server but client connection string missing SSL flags.

#### Real-world scenario
- A PHP or Node.js app needs a classic relational database.
- Developers connect locally with `mysql` and administrators use MySQL Workbench.
- Production traffic uses a private subnet and a ProxySQL or HAProxy layer in front.

### 3.2 🐘 PostgreSQL

**Category:** Relational (RDBMS)
**Default port:** 5432

#### Best fit
- Transactional systems
- Complex SQL
- Geospatial and extension-heavy workloads
- Analytics on operational data

#### Installation steps on Ubuntu / Debian
```bash
sudo apt update
sudo apt install -y postgresql postgresql-client
```

#### Installation steps on RHEL / Rocky / AlmaLinux
```bash
sudo dnf install -y postgresql-server postgresql
sudo postgresql-setup --initdb
```

#### Installation steps on macOS with Homebrew
```bash
brew install postgresql
brew services start postgresql
```

#### Starting the service
```bash
sudo systemctl enable --now postgresql
sudo systemctl status postgresql
```

#### Local connection commands
```bash
sudo -u postgres psql
psql -h 127.0.0.1 -U appuser -d appdb
psql "postgresql://appuser:password@127.0.0.1:5432/appdb"
```

#### Remote connection setup
1. Edit `postgresql.conf` and `pg_hba.conf`.
2. Set `listen_addresses = "*"` or a specific private IP.
3. Add a narrow host rule to `pg_hba.conf` for the application subnet.
4. Prefer `scram-sha-256` over older password methods.
5. Restart PostgreSQL and test with `psql` from the application network.

#### Example configuration
```conf
listen_addresses = '*'
ssl = on
password_encryption = scram-sha-256

# pg_hba.conf
hostssl appdb appuser 10.10.0.0/16 scram-sha-256
```

#### Connection string formats
- `psql -h db.example.com -U appuser -d appdb`
- `postgresql://appuser:password@db.example.com:5432/appdb`
- `jdbc:postgresql://db.example.com:5432/appdb`
- `postgresql+psycopg://appuser:password@db.example.com:5432/appdb`

#### GUI tools
- pgAdmin
- DBeaver
- DataGrip
- TablePlus

#### Common connection issues
- `pg_hba.conf` missing the client IP range.
- Server listens only on localhost.
- SSL mode mismatch between client and server.
- Role exists but lacks CONNECT or schema privileges.

#### Real-world scenario
- A fintech API needs strong SQL support and strict data integrity.
- Apps connect through PgBouncer to reduce backend connection pressure.
- Standby servers receive WAL streaming for HA and read-only reporting.

### 3.3 🍃 MongoDB

**Category:** NoSQL Document Database
**Default port:** 27017

#### Best fit
- JSON-like documents
- Content/catalog workloads
- Event-rich application objects
- Rapid schema evolution

#### Installation steps on Ubuntu / Debian
```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
```

#### Installation steps on RHEL / Rocky / AlmaLinux
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
```

#### Installation steps on macOS with Homebrew
```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
```

#### Starting the service
```bash
sudo systemctl enable --now mongod
sudo systemctl status mongod
```

#### Local connection commands
```bash
mongosh
mongosh "mongodb://localhost:27017"
mongosh "mongodb://appuser:password@localhost:27017/appdb?authSource=admin"
```

#### Remote connection setup
1. Edit `/etc/mongod.conf`.
2. Set `net.bindIp` to a private address or `0.0.0.0` only when protected by firewall rules.
3. Enable `security.authorization: enabled` before exposing MongoDB remotely.
4. For replica sets, also define `replication.replSetName`.
5. Use TLS for production networks and cloud environments.

#### Example configuration
```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,10.10.0.20
security:
  authorization: enabled
replication:
  replSetName: rs0
```

#### Connection string formats
- `mongosh "mongodb://db.example.com:27017"`
- `mongodb://appuser:password@db.example.com:27017/appdb?authSource=admin`
- `mongodb+srv://appuser:password@cluster0.example.mongodb.net/appdb`
- `mongodb://appuser:password@db1,db2,db3/appdb?replicaSet=rs0`

#### GUI tools
- MongoDB Compass
- Studio 3T
- DBeaver
- DataGrip

#### Common connection issues
- Authentication enabled but user created in the wrong database.
- Replica set URI missing `replicaSet=...`.
- Client allowed through firewall but not listed in bindIp.
- TLS CA or certificate hostname mismatch.

#### Real-world scenario
- An e-commerce catalog stores products with different attributes.
- Backend services use `mongosh` for ad hoc admin checks and Compass for visual exploration.
- Production uses a three-member replica set before any sharding is introduced.

### 3.4 🟥 Redis

**Category:** NoSQL Key-Value Store
**Default port:** 6379

#### Best fit
- Caching
- Sessions
- Rate limiting
- Short-lived queues and counters

#### Installation steps on Ubuntu / Debian
```bash
sudo apt update
sudo apt install -y redis-server
```

#### Installation steps on RHEL / Rocky / AlmaLinux
```bash
sudo dnf install -y redis
```

#### Installation steps on macOS with Homebrew
```bash
brew install redis
brew services start redis
```

#### Starting the service
```bash
sudo systemctl enable --now redis-server || sudo systemctl enable --now redis
sudo systemctl status redis-server || sudo systemctl status redis
```

#### Local connection commands
```bash
redis-cli
redis-cli -a StrongPass! ping
redis-cli -u redis://:StrongPass!@127.0.0.1:6379/0
```

#### Remote connection setup
1. Edit `/etc/redis/redis.conf` or `/etc/redis.conf`.
2. Set `bind` to a private IP list or `0.0.0.0` only with strong controls.
3. Keep `protected-mode yes` unless the network boundary and authentication are solid.
4. Use ACLs or at minimum `requirepass` for older setups.
5. Consider TLS and a private load balancer for production access.

#### Example configuration
```conf
bind 127.0.0.1 10.10.0.30
protected-mode yes
port 6379
appendonly yes
# ACLs recommended over a single shared password
```

#### Connection string formats
- `redis-cli -h redis.example.com -p 6379 -a StrongPass!`
- `redis://:password@redis.example.com:6379/0`
- `rediss://:password@redis.example.com:6380/0`
- `redis-sentinel://sentinel1:26379,sentinel2:26379,mymaster`

#### GUI tools
- RedisInsight
- Medis
- DBeaver
- Another Redis Desktop Manager

#### Common connection issues
- Protected mode rejects remote clients.
- ACL user exists but command category permissions are insufficient.
- Persistence disabled by accident so restarts lose data.
- The application needs TLS but only plain TCP is enabled.

#### Real-world scenario
- A web API caches profile lookups and stores rate-limit counters.
- Developers use `redis-cli` for quick inspection and RedisInsight for dashboards.
- Production often pairs Redis with Sentinel for failover or Cluster for horizontal partitioning.

### 3.5 🔎 Elasticsearch

**Category:** Search Engine / Distributed Document Store
**Default port:** 9200 (HTTP), 9300 (transport)

#### Best fit
- Full-text search
- Observability
- Security analytics
- Faceted exploration and log indexing

#### Installation steps on Ubuntu / Debian
```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
sudo apt install -y elasticsearch kibana
```

#### Installation steps on RHEL / Rocky / AlmaLinux
```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat <<'EOF' | sudo tee /etc/yum.repos.d/elasticsearch.repo
[elasticsearch]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
sudo dnf install -y elasticsearch kibana
```

#### Installation steps on macOS with Homebrew
```bash
brew tap elastic/tap
brew install elastic/tap/elasticsearch-full
brew install elastic/tap/kibana-full
```

#### Starting the service
```bash
sudo systemctl enable --now elasticsearch
sudo systemctl enable --now kibana
sudo systemctl status elasticsearch
```

#### Local connection commands
```bash
curl -k https://localhost:9200
curl -u elastic:password -k https://localhost:9200/_cluster/health?pretty
curl -u elastic:password -k https://localhost:9200/_cat/indices?v
```

#### Remote connection setup
1. Edit `/etc/elasticsearch/elasticsearch.yml`.
2. Set `network.host` and optionally `http.port`.
3. Enable security features and TLS before remote exposure.
4. For clusters, define discovery and node roles explicitly.
5. Expose Kibana separately and keep transport port 9300 private between cluster nodes only.

#### Example configuration
```yaml
cluster.name: prod-search
node.name: es-1
network.host: 10.10.0.40
http.port: 9200
discovery.seed_hosts: ["10.10.0.41", "10.10.0.42"]
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
```

#### Connection string formats
- `curl -u elastic:password https://search.example.com:9200`
- `https://elastic:password@search.example.com:9200`
- `https://search.example.com:9200/_search`
- `https://kibana.example.com:5601` for GUI access via Kibana

#### GUI tools
- Kibana
- Elasticvue
- Cerebro (older clusters)
- DBeaver with HTTP connectors for basic exploration

#### Common connection issues
- TLS enabled but curl missing `--cacert` or trusted CA.
- Single-node discovery settings accidentally copied into multi-node production.
- Opening port 9300 to clients instead of keeping it internal for cluster traffic.
- Trying to treat Elasticsearch like a transactional relational primary database.

#### Real-world scenario
- A logging platform indexes application logs and security events.
- Operators test queries with `curl` and visualize data in Kibana.
- Production uses multiple master-eligible and data nodes with replica shards.

### 3.6 🗄️ SQLite

**Category:** Embedded Relational Database
**Default port:** No server port

#### Best fit
- Desktop apps
- CLI tools
- Small services
- Local testing and edge devices

#### Installation steps on Ubuntu / Debian
```bash
sudo apt update
sudo apt install -y sqlite3
```

#### Installation steps on RHEL / Rocky / AlmaLinux
```bash
sudo dnf install -y sqlite
```

#### Installation steps on macOS with Homebrew
```bash
brew install sqlite
```

#### Starting the service
```bash
# SQLite does not run as a background service.
# It is a library plus a database file accessed directly by the application.
```

#### Local connection commands
```bash
sqlite3 app.db
sqlite3 app.db ".tables"
sqlite3 app.db "SELECT sqlite_version();"
```

#### Remote connection setup
1. SQLite is not designed for remote TCP access like client/server databases.
2. Do not place SQLite on unreliable network filesystems for multi-writer workloads.
3. If remote access is needed, put an application or API server in front of the SQLite file.
4. Use file permissions, backups, and WAL mode for safer local concurrency.

#### Example configuration
```conf
PRAGMA journal_mode = WAL;
PRAGMA foreign_keys = ON;
PRAGMA synchronous = NORMAL;
```

#### Connection string formats
- `sqlite3 /path/to/app.db`
- `sqlite:///absolute/path/to/app.db`
- `sqlite:////absolute/path/to/app.db`
- `jdbc:sqlite:/absolute/path/to/app.db`

#### GUI tools
- DB Browser for SQLite
- SQLiteStudio
- DBeaver
- TablePlus

#### Common connection issues
- File permission errors on the database file or parent directory.
- Concurrent writers blocked because the workload outgrew SQLite.
- Application assumes server features such as users and network listeners that SQLite does not provide.
- Database file copied while writes were active without using backup mode.

#### Real-world scenario
- A local CLI utility stores metadata without needing a separate server process.
- Developers use `sqlite3` for testing and DB Browser for SQLite to inspect tables.
- Production use works best when write concurrency is modest and the deployment is simple.

### 3.7 Other important engines at a glance

| Engine | Install / client | Typical connection command | Notes |
|---|---|---|---|
| CockroachDB | `cockroach` binary | `cockroach sql --url "postgresql://root@host:26257/defaultdb?sslmode=require"` | Distributed SQL with PostgreSQL-style connectivity |
| TiDB | MySQL-compatible clients | `mysql -h tidb.example.com -P 4000 -u root -p` | Distributed SQL with MySQL wire compatibility |
| Cassandra | `cqlsh` | `cqlsh cassandra.example.com 9042` | Wide-column model; query design centers on partition keys |
| Neo4j | `cypher-shell` | `cypher-shell -a neo4j+s://graph.example.com:7687 -u neo4j -p password` | Graph query language is Cypher |
| InfluxDB | `influx` CLI | `influx config create --host-url http://metrics.example.com:8086` | Time-series engine with tokens and buckets |
| TimescaleDB | PostgreSQL clients | `psql -h tsdb.example.com -U metrics -d telemetry` | Extension on PostgreSQL |
| Solr | curl or Solr UI | `curl http://solr.example.com:8983/solr/admin/collections?action=LIST` | Search engine frequently run with ZooKeeper in older versions |

---

## 4. Remote Access Configuration

### 4.1 Start with a private network mindset

- Expose databases to private networks first, not the public internet.
- Prefer application-to-database traffic over VPNs, private VPCs, or zero-trust overlays.
- Expose a bastion or SSH endpoint before exposing a database listener.
- Keep management planes and data planes separate when possible.

### 4.2 Binding to network interfaces

| Database | Config directive | Safe default | Common production choice |
|---|---|---|---|
| MySQL/MariaDB | `bind-address` | `127.0.0.1` | Private subnet IP or `0.0.0.0` behind firewall |
| PostgreSQL | `listen_addresses` | `localhost` | Private IPs or `*` with strict `pg_hba.conf` |
| MongoDB | `net.bindIp` | `127.0.0.1` | `127.0.0.1,<private-ip>` |
| Redis | `bind` | `127.0.0.1` | Private IPs with ACLs and protected mode |
| Elasticsearch | `network.host` | `localhost` | Private IP for cluster nodes and clients |
| SQLite | N/A | Local file | Application-mediated access only |

### 4.3 Authentication configuration

Remote reachability without authentication is an outage waiting to happen.

1. Create dedicated service accounts per application.
2. Do not reuse administrator credentials in application code.
3. Rotate passwords, keys, or certificates on a schedule.
4. Prefer modern auth methods such as SCRAM, ACL users, or X.509-based options.
5. Use secret stores or environment injection instead of committing credentials to repositories.

### 4.4 Firewall rules by database port

| Database | Port(s) | Protocol | Comment |
|---|---|---|---|
| MySQL / MariaDB | 3306 | TCP | App servers only |
| PostgreSQL | 5432 | TCP | App servers, poolers, bastions |
| MongoDB | 27017 | TCP | Replica set peers and apps only |
| Redis | 6379 | TCP | Private network only |
| Elasticsearch HTTP | 9200 | TCP | Clients and Kibana integrations |
| Elasticsearch transport | 9300 | TCP | Cluster-internal only |
| CockroachDB | 26257 | TCP | SQL clients |
| TiDB | 4000 | TCP | MySQL-compatible clients |
| Cassandra | 9042 | TCP | CQL clients |
| Neo4j | 7687 | TCP | Bolt protocol clients |
| Solr | 8983 | TCP | Search/API clients |
| InfluxDB | 8086 | TCP | Metrics writers and readers |

#### UFW examples
```bash
sudo ufw allow from 10.10.0.0/16 to any port 3306 proto tcp comment 'MySQL from app subnet'
sudo ufw allow from 10.10.0.0/16 to any port 5432 proto tcp comment 'PostgreSQL from app subnet'
sudo ufw allow from 10.10.0.0/16 to any port 27017 proto tcp comment 'MongoDB from app subnet'
sudo ufw allow from 10.10.0.0/16 to any port 6379 proto tcp comment 'Redis from app subnet'
sudo ufw allow from 10.10.0.0/16 to any port 9200 proto tcp comment 'Elasticsearch HTTP from app subnet'
sudo ufw allow from 10.10.1.0/24 to any port 9300 proto tcp comment 'Elasticsearch transport between nodes'
```

#### firewalld examples
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.0.0/16" port protocol="tcp" port="3306" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.0.0/16" port protocol="tcp" port="5432" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.0.0/16" port protocol="tcp" port="27017" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.0.0/16" port protocol="tcp" port="6379" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.10.0.0/16" port protocol="tcp" port="9200" accept'
sudo firewall-cmd --reload
```

#### iptables examples
```bash
sudo iptables -A INPUT -p tcp -s 10.10.0.0/16 --dport 3306 -j ACCEPT
sudo iptables -A INPUT -p tcp -s 10.10.0.0/16 --dport 5432 -j ACCEPT
sudo iptables -A INPUT -p tcp -s 10.10.0.0/16 --dport 27017 -j ACCEPT
sudo iptables -A INPUT -p tcp -s 10.10.0.0/16 --dport 6379 -j ACCEPT
sudo iptables -A INPUT -p tcp -s 10.10.0.0/16 --dport 9200 -j ACCEPT
```

### 4.5 SSL/TLS encrypted connections

TLS protects credentials and query traffic from interception.

**General steps:**
1. Issue server certificates with correct DNS names or IP Subject Alternative Names.
2. Distribute the CA certificate to clients.
3. Enable TLS on the server side and force encrypted transport where possible.
4. Update client connection strings to require or verify TLS.
5. Test certificate expiration and renewal before production rollout.

**Examples by engine:**
- MySQL/MariaDB: set `require_secure_transport = ON`, then provide `ssl_ca`, `ssl_cert`, and `ssl_key` as needed.
- PostgreSQL: set `ssl = on`, place `server.crt` and `server.key`, and use `hostssl` rules in `pg_hba.conf`.
- MongoDB: use `net.tls.mode: requireTLS` and provide CA and certificate files.
- Redis: use `tls-port`, certificates, and optionally disable the plain port.
- Elasticsearch: enable `xpack.security.http.ssl.enabled` and `xpack.security.transport.ssl.enabled`.
- SQLite: no network TLS because there is no server listener.

### 4.6 SSH tunneling for secure remote access

SSH tunneling is ideal for administrators who need occasional access without permanently opening database ports broadly.

```bash
# MySQL through bastion
ssh -L 3307:127.0.0.1:3306 admin@bastion.example.com
mysql -h 127.0.0.1 -P 3307 -u appuser -p appdb

# PostgreSQL through bastion
ssh -L 5433:127.0.0.1:5432 admin@bastion.example.com
psql -h 127.0.0.1 -p 5433 -U appuser -d appdb

# MongoDB through bastion
ssh -L 27018:127.0.0.1:27017 admin@bastion.example.com
mongosh "mongodb://127.0.0.1:27018/appdb"

# Redis through bastion
ssh -L 6380:127.0.0.1:6379 admin@bastion.example.com
redis-cli -h 127.0.0.1 -p 6380
```

### 4.7 Connection pooling

Pooling reduces connection overhead and shields the database from connection storms.

| Database family | Popular pooler / proxy | Why it helps |
|---|---|---|
| PostgreSQL | PgBouncer | Reduces backend process count and smooths spiky traffic |
| MySQL/MariaDB | ProxySQL or MaxScale | Multiplexing, routing, and read/write splitting |
| MongoDB | Driver-managed pools | The driver usually handles socket pools natively |
| Redis | Client pool in app or Twemproxy in some setups | Avoids connect/disconnect overhead |
| Elasticsearch | HTTP keep-alive and client-side node pools | Reduces TCP churn and improves routing |
| SQLite | App-level file access discipline | No central network pooler concept |

#### PgBouncer example
```ini
[databases]
appdb = host=10.10.0.20 port=5432 dbname=appdb

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256
pool_mode = transaction
default_pool_size = 100
max_client_conn = 1000
```

#### ProxySQL example
```sql
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (10, '10.10.0.20', 3306);
INSERT INTO mysql_users(username, password, default_hostgroup) VALUES ('appuser', 'StrongPass!', 10);
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
```

### 4.8 Remote access network diagram

```mermaid
flowchart LR
    U[Admin laptop] -->|SSH tunnel| B[Bastion host]
    A1[App server 1] -->|Private subnet| P1[PgBouncer / ProxySQL]
    A2[App server 2] -->|Private subnet| P1
    B --> P1
    P1 --> DB[(Primary database)]
    DB --> R1[(Replica 1)]
    DB --> R2[(Replica 2)]
    O[Observability stack] --> DB
    F[Firewall / SG rules] -. allows specific subnets .-> DB
```

### 4.9 Remote access checklist

- Server binds only to the needed interface.
- Authentication is enabled and tested.
- Strong per-application users exist.
- Firewall rules allow only approved source CIDRs.
- TLS is enabled or an SSH tunnel / VPN is used.
- Monitoring detects failed logins and unusual traffic patterns.
- Backups exist before exposure changes are rolled out.

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

## 7. Databases in Containers

Containers are excellent for local development, CI labs, and some production architectures when persistent storage is handled carefully.

### 7.1 Container principles

- Always mount persistent volumes for stateful engines.
- Keep configuration in environment variables, mounted config files, or secrets.
- Health checks and resource limits matter for stateful services.
- Do not confuse a running container with a durable backup strategy.

### 7.2 MySQL / MariaDB in Docker

#### `docker run` example
```bash
docker run -d --name mysql-demo \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=appdb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppass \
  -p 3306:3306 \
  -v mysql_data:/var/lib/mysql \
  mysql:8.4
```

#### Docker Compose example
```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: mysql-demo
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
volumes:
  mysql_data:
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.3 PostgreSQL in Docker

#### `docker run` example
```bash
docker run -d --name postgres-demo \
  -e POSTGRES_DB=appdb \
  -e POSTGRES_USER=appuser \
  -e POSTGRES_PASSWORD=apppass \
  -p 5432:5432 \
  -v pg_data:/var/lib/postgresql/data \
  postgres:16
```

#### Docker Compose example
```yaml
services:
  postgres:
    image: postgres:16
    container_name: postgres-demo
    restart: unless-stopped
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data
volumes:
  pg_data:
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.4 MongoDB in Docker

#### `docker run` example
```bash
docker run -d --name mongo-demo \
  -p 27017:27017 \
  -v mongo_data:/data/db \
  mongo:7
```

#### Docker Compose example
```yaml
services:
  mongo:
    image: mongo:7
    container_name: mongo-demo
    restart: unless-stopped
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
volumes:
  mongo_data:
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.5 Redis in Docker

#### `docker run` example
```bash
docker run -d --name redis-demo \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:7 redis-server --appendonly yes
```

#### Docker Compose example
```yaml
services:
  redis:
    image: redis:7
    container_name: redis-demo
    restart: unless-stopped
    command: ["redis-server", "--appendonly", "yes"]
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
volumes:
  redis_data:
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.6 Elasticsearch in Docker

#### `docker run` example
```bash
docker run -d --name elastic-demo \
  -e discovery.type=single-node \
  -e xpack.security.enabled=false \
  -p 9200:9200 \
  -v es_data:/usr/share/elasticsearch/data \
  docker.elastic.co/elasticsearch/elasticsearch:8.14.3
```

#### Docker Compose example
```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.3
    container_name: elastic-demo
    restart: unless-stopped
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      ES_JAVA_OPTS: -Xms1g -Xmx1g
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data
volumes:
  es_data:
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.7 SQLite in Docker

#### `docker run` example
```bash
docker run --rm -it \
  -v "$PWD":/workspace \
  alpine:3.20 sh -c "apk add --no-cache sqlite && sqlite3 /workspace/app.db"
```

#### Docker Compose example
```yaml
services:
  sqlite-toolbox:
    image: alpine:3.20
    container_name: sqlite-toolbox
    command: ["sh", "-c", "apk add --no-cache sqlite && sleep infinity"]
    volumes:
      - ./:/workspace
```

#### Volume note
- Use named volumes or bind mounts for persistent data.
- Back up the volume or use engine-native backup tools from inside the container or a sidecar.
- Validate permissions and SELinux or AppArmor policies when bind-mounting host paths.

### 7.8 Multi-service example with application and database

```yaml
services:
  app:
    image: ghcr.io/example/app:latest
    environment:
      DATABASE_URL: postgresql://appuser:apppass@postgres:5432/appdb
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - pg_data:/var/lib/postgresql/data
  redis:
    image: redis:7
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
volumes:
  pg_data:
  redis_data:
```

### 7.9 Persistent storage patterns

| Pattern | Good for | Watch out for |
|---|---|---|
| Named volume | Simple local development | Host placement can be opaque |
| Bind mount | Explicit path control and easier host-level inspection | Permissions and portability |
| CSI / cloud volume | Kubernetes and production orchestration | Performance tier and failover behavior |
| Snapshot-based backup | Fast point-in-time copy of volumes | Application consistency still matters |

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

## 11. Operational Appendices

These appendices turn the high-level guide into day-to-day operational playbooks.

### 11.1 Query lifecycle by engine

#### MySQL / MariaDB

1. Client sends SQL text to the server.
2. The parser validates syntax and resolves identifiers.
3. The optimizer compares plans and index choices.
4. InnoDB reads pages from buffer pool or disk.
5. Locks or MVCC rules control visibility.
6. The server returns rows and status metadata.

#### PostgreSQL

1. Client sends SQL through the wire protocol.
2. Parser and rewriter normalize the statement.
3. Planner estimates row counts and costs.
4. Executor uses MVCC snapshots to read visible tuples.
5. Buffers, WAL, and background workers support execution.
6. Results return to the client and statistics update.

#### MongoDB

1. Client sends a command or query document.
2. The query layer validates namespace and auth.
3. Indexes and collection metadata are consulted.
4. The storage engine reads documents and applies filters.
5. Replication logs the change if the operation is a write.
6. The BSON result returns to the client.

#### Redis

1. Client sends a RESP command.
2. Redis parses the command on the event loop.
3. The keyspace lookup finds the target value.
4. The in-memory structure is updated or read.
5. Persistence may append to AOF or trigger an RDB snapshot later.
6. A reply is written back over the socket.

#### Elasticsearch

1. Client sends an HTTP request.
2. The coordinating node parses the JSON DSL.
3. Relevant shards are selected and queried.
4. Shard-level results are scored or aggregated.
5. Replica and primary shard rules apply to writes.
6. The coordinating node merges results and returns JSON.

#### SQLite

1. The application opens the database file.
2. SQLite parses SQL inside the process.
3. The query planner selects indexes or table scans.
4. The pager reads pages from the database file.
5. A journal or WAL file protects transactional writes.
6. Results return directly to the calling process.

### 11.2 Schema design playbook

- Model business entities first: users, orders, invoices, sessions, products, devices, logs, and permissions.
- Write down the read paths and write paths separately.
- Identify natural keys, surrogate keys, and external IDs.
- Decide which relationships must be enforced by the database versus the application.
- Create retention rules before data volume becomes painful.
- Document who owns each table, collection, index, or pipeline.

| Question | Why it matters | Example |
|---|---|---|
| What is the primary key? | Determines uniqueness and many access paths | order_id as bigint or UUID |
| How large can rows/documents get? | Affects IO, cache efficiency, and replication | product documents with many attributes |
| Which fields are queried together? | Drives composite index design | customer_id + created_at |
| What is immutable? | Makes auditing and partitioning easier | invoice line items after finalization |
| What expires? | Controls TTL, archiving, and storage cost | sessions older than 30 days |
| What is security-sensitive? | Guides encryption and access design | PII, tokens, payroll data |

#### Relational design checklist

- Use explicit primary keys everywhere.
- Name foreign keys and indexes consistently.
- Keep data types precise: avoid generic text when a boolean, numeric, or timestamp fits better.
- Use CHECK constraints when the business rule is stable enough to enforce centrally.
- Avoid nullable columns for values that the application always requires.
- Track created_at and updated_at for operational visibility.

#### Document design checklist

- Embed small child objects when they are always read with the parent.
- Reference large or independently changing substructures.
- Avoid unbounded arrays in frequently updated documents.
- Design shard keys and indexes before write volume explodes.
- Be clear about which fields must exist even in a flexible schema.

#### Cache and key design checklist

- Use descriptive key prefixes such as `user:123:profile`.
- Set TTLs intentionally rather than leaving stale data forever.
- Decide whether cache misses should fall back to the database or fail fast.
- Avoid storing values that are too large or too hot for the chosen memory budget.
- Track hit ratio, eviction rate, and key cardinality.

### 11.3 Normalization and denormalization

Normalization improves correctness and reduces duplication; denormalization improves read performance or simplifies access patterns when applied deliberately.

| Pattern | Best use | Risk if overused |
|---|---|---|
| 1NF and 3NF tables | Transactional sources of truth | Extra joins may appear in read-heavy paths |
| Materialized summary table | Dashboards and reports | Staleness if refresh logic is weak |
| Embedded document | Read-mostly aggregates | Large writes and duplicated subdocuments |
| Precomputed cache value | Fast landing pages and hot lookups | Cache invalidation complexity |
| Search index projection | Text search and faceting | Dual-write drift if indexing pipeline breaks |

#### When to denormalize

1. The read path is far hotter than the write path.
2. The duplication can be rebuilt safely from the source of truth.
3. The business can tolerate slight staleness.
4. The team has monitoring for projection lag or refresh failures.
5. The storage overhead is acceptable compared with performance gain.

### 11.4 Index design patterns

| Pattern | Use it when | Example |
|---|---|---|
| Single-column B-tree | One filter dominates | Find user by email |
| Composite B-tree | Multiple predicates appear together | customer_id + created_at |
| Covering index | You can satisfy the query from the index alone | status + created_at + total |
| Partial index | Only a subset of rows matters | orders where status = open |
| GIN / full-text | Search inside text or JSON structures | PostgreSQL JSONB search |
| TTL index | Data should expire automatically | MongoDB session collection |
| Inverted index | Search relevance matters | Elasticsearch terms and analyzers |

#### Index review questions

1. Does the index match the WHERE clause order and selectivity?
2. Will the index help sorting and pagination?
3. Does the index duplicate another wider index?
4. What is the write penalty for every insert or update?
5. Does the optimizer actually choose the index in practice?
6. Have you measured the before and after impact?

### 11.5 Change-management patterns

#### Expand-contract migrations

1. Add the new column or structure first.
2. Deploy application code that writes both old and new representations if needed.
3. Backfill historical rows or documents in small batches.
4. Switch reads to the new representation.
5. Remove the old column only after verification and rollback planning.

#### Zero-downtime migration habits

- Avoid long blocking DDL during peak traffic.
- Batch large backfills to keep replication healthy.
- Measure lock duration in staging with production-like sizes.
- Use feature flags when application changes and schema changes must overlap.
- Keep a rollback story for both data and code.

#### Migration preflight checklist

- Recent backup exists.
- Replica lag is healthy.
- Disk free space covers temp files and index builds.
- On-call engineer knows how to pause or reverse the rollout.
- Observability dashboards are open before the change starts.

### 11.6 Backup and recovery workflow diagram

```mermaid
flowchart TD
    Start[Planned backup window] --> Freeze{Need consistency point?}
    Freeze -->|Yes| Prep[Flush or checkpoint workload]
    Freeze -->|No| Backup[Run engine-native backup]
    Prep --> Backup
    Backup --> Verify[Verify backup metadata]
    Verify --> Encrypt[Encrypt and store off-host]
    Encrypt --> RestoreTest[Perform restore test]
    RestoreTest --> Success[Mark backup as valid]
```

### 11.7 Disaster recovery tabletop scenarios

#### Primary host lost due to hardware failure

1. Confirm whether replicas are current enough for failover.
2. Promote the best replica or run orchestrated failover.
3. Redirect applications through DNS, VIP, or pooler.
4. Capture the failed node for forensics if possible.
5. Rebuild a new replica after service is stable.

#### Application accidentally dropped a table or collection

1. Stop destructive automation quickly.
2. Confirm last known good backup and binlog/WAL/oplog position.
3. Restore into an isolated environment first.
4. Extract only the required object or rows when possible.
5. Review permissions and deployment controls afterward.

#### Disk filled up on the database host

1. Identify which filesystem is full.
2. Remove safe temporary files or extend storage.
3. Check whether WAL, binlog, snapshots, or logs grew unexpectedly.
4. Validate database integrity after reclaiming space.
5. Add alerts and retention controls to prevent recurrence.

#### Credential leak suspected

1. Rotate the exposed credential immediately.
2. Review audit logs for unusual logins and queries.
3. Invalidate cached secrets in applications and CI/CD.
4. Issue new certificates or passwords from the secret store.
5. Document the incident and tighten least-privilege rules.

#### Replica lag spikes during a migration

1. Pause the migration or backfill job.
2. Measure write rate, IO pressure, and lock behavior.
3. Determine whether replicas are CPU-bound, disk-bound, or network-bound.
4. Catch replicas up before the next phase.
5. Adjust batch sizes and maintenance windows for the retry.

### 11.8 First 15 minutes runbooks

#### MySQL / MariaDB

1. Run `mysqladmin ping`.
2. Inspect `SHOW PROCESSLIST;` for blocked sessions.
3. Check disk space and InnoDB status.
4. Confirm replication status if replicas exist.
5. Review recent DDL or deployment events.

#### PostgreSQL

1. Run `pg_isready`.
2. Inspect `pg_stat_activity` and lock waits.
3. Check `pg_stat_replication` for lag.
4. Review `postgresql.log` for crashes or checkpoints.
5. Confirm pooler health if PgBouncer is in front.

#### MongoDB

1. Run `db.serverStatus()`.
2. Check `rs.status()` if using a replica set.
3. Verify free disk and oplog window.
4. Look for slow operations or long-running index builds.
5. Confirm the primary and secondaries are reachable.

#### Redis

1. Run `PING` and `INFO`.
2. Check memory pressure and eviction rate.
3. Confirm persistence files and last save status.
4. Review Sentinel or Cluster state if applicable.
5. Identify whether a hot key or command storm is occurring.

#### Elasticsearch

1. Query `_cluster/health`.
2. Check unassigned shards.
3. Inspect JVM heap pressure and node availability.
4. Confirm ingest pipelines or index templates were not changed badly.
5. Review disk watermarks and pending tasks.

#### SQLite

1. Check file existence and permissions.
2. Run `PRAGMA integrity_check;`.
3. Confirm the application has not placed the file on an unsafe network share.
4. Review WAL and journal files.
5. Consider whether concurrency demands exceeded SQLite's design.

### 11.9 Maintenance calendar example

| Cadence | Tasks |
|---|---|
| Daily | Backup verification, failed login review, disk free space check, replication health review |
| Weekly | Restore test, slow query review, capacity trend review, certificate expiry check |
| Monthly | Privilege audit, index review, failover rehearsal, patch planning |
| Quarterly | Architecture review, DR tabletop exercise, retention policy validation, secret rotation |
| Yearly | Major version upgrade planning, hardware refresh review, compliance evidence refresh |

### 11.10 Study and interview questions

1. What is the difference between a backup and a replica?
2. Why can a unique index be both a correctness tool and a performance tool?
3. When would you choose PostgreSQL over MySQL, and when would the reverse be true?
4. Why is Elasticsearch a poor sole system of record for financial transactions?
5. What makes Redis excellent for caches but risky as the only store for critical data?
6. How does WAL or binlog-based recovery help after a crash?
7. What does MVCC solve, and what does it not solve?
8. Why do network partitions matter more in distributed systems than on a single host?
9. What is the difference between logical and physical backups?
10. Why are partial or filtered indexes useful?
11. How does connection pooling improve reliability?
12. What is the safest way to expose a database to an administrator over the internet?
13. Why are restore tests more important than backup job success logs?
14. What signs show that SQLite has outgrown the workload?
15. How do TTLs, retention policies, and archival strategies differ?
16. Why should application accounts rarely have DDL privileges?
17. What is a shard key and why is it hard to change later?
18. What happens if your failover plan ignores replica lag?
19. Why is least privilege an operational reliability feature as well as a security feature?
20. What database type best fits search relevance, time-series rollups, and graph traversals respectively?

## 12. Glossary

- **ACID:** Atomicity, consistency, isolation, and durability transaction guarantees.
- **Arbiter:** A MongoDB voting member that does not store data.
- **Availability:** The ability of a system to return a response to requests.
- **Backup:** A copy of data that can be restored after loss or corruption.
- **B-tree:** A balanced tree index structure used by many relational engines.
- **CAP theorem:** A model describing consistency, availability, and partition-tolerance trade-offs.
- **Checkpoint:** A point where dirty data or logs are synchronized for recovery purposes.
- **Cluster:** A set of cooperating database nodes.
- **Collection:** A MongoDB grouping of documents, roughly analogous to a table.
- **Composite index:** An index on multiple columns or fields in order.
- **Connection pool:** A reusable set of open connections managed by an app or proxy.
- **Constraint:** A rule enforced by the database, such as UNIQUE or FOREIGN KEY.
- **Consistency:** In CAP, reads reflect the most recent write or an error is returned.
- **Durability:** Committed data survives failure according to the system design.
- **Failover:** Promoting or switching to a healthy node after failure.
- **Foreign key:** A relational constraint tying a child row to a parent row.
- **Full-text search:** Searching analyzed text for relevance-ranked matches.
- **GIN index:** A PostgreSQL index useful for arrays, JSONB, and full-text search.
- **High availability:** Architecture and operations designed to minimize downtime.
- **Hot key:** A frequently accessed cache key that can create imbalance.
- **Index selectivity:** How effectively an indexed value narrows down matching rows.
- **Isolation level:** A rule defining visibility behavior between concurrent transactions.
- **Journal:** A file or log used to protect or recover transactional changes.
- **Keyspace:** A namespace concept in some NoSQL systems; in Redis it often refers to the set of keys.
- **Least privilege:** Granting only the permissions required for a task.
- **MVCC:** Multi-version concurrency control; readers can see a stable snapshot while writers progress.
- **Node:** A server or process participating in a database cluster.
- **Normalization:** Reducing duplication by organizing relational data into well-structured tables.
- **Oplog:** MongoDB replication log.
- **Partition:** A horizontal slice of data or a network split between nodes, depending on context.
- **Partition tolerance:** The ability of a distributed system to continue despite network splits.
- **Primary:** The write leader in many replicated architectures.
- **Projection:** A derived copy of data shaped for a specific read path.
- **Quorum:** The minimum number of votes or nodes needed for a decision.
- **Read replica:** A node that receives replicated changes and serves read traffic.
- **Recovery point objective:** Maximum acceptable data loss window.
- **Recovery time objective:** Maximum acceptable service restoration time.
- **Replica set:** A MongoDB replication group.
- **Retention policy:** Rules controlling how long data is kept.
- **Schema:** The structure, types, and relationships of stored data.
- **Shard:** A partition of data distributed across nodes.
- **Snapshot:** A point-in-time copy of data or storage blocks.
- **Source of truth:** The authoritative data system from which other copies are derived.
- **Standby:** A replica prepared to take over as primary.
- **TLS:** Transport Layer Security for encrypted connections.
- **Transaction:** A unit of work that is committed or rolled back as one whole.
- **TTL:** Time to live; automatic expiration for cache entries or documents.
- **VACUUM:** Space reclamation or file-compaction operation, depending on engine.
- **WAL:** Write-ahead log used for crash recovery and replication in many systems.
- **Working set:** The portion of data actively used and ideally kept in memory.
- **Write amplification:** Extra physical writes caused by internal maintenance beyond the logical change.

## 13. Hands-On Practice Labs

Use these labs to turn the concepts in this guide into repeatable muscle memory.

### 13.1 Secure PostgreSQL remote access

**Objective:** Start a local or VM-based PostgreSQL instance and expose it only to a private subnet or SSH tunnel.

**Tasks:**
1. Install PostgreSQL and confirm `psql` works locally.
2. Set `listen_addresses`, add a `hostssl` rule in `pg_hba.conf`, and enable SCRAM.
3. Create a non-admin application role and a database.
4. Connect through an SSH tunnel and through the private IP.
5. Verify failed connections from unauthorized IPs are denied.

**Success criteria:**
- `pg_isready` returns success.
- The app role can connect but cannot create superusers.
- Connections without a matching `pg_hba.conf` rule fail as expected.

### 13.2 MySQL backup and restore drill

**Objective:** Practice a no-surprises backup and restore cycle.

**Tasks:**
1. Create a sample schema and load test rows.
2. Run `mysqldump --single-transaction`.
3. Restore into a second database.
4. Compare row counts and checksums.
5. Document how long the restore took.

**Success criteria:**
- Backup file created successfully.
- Restored data matches expected counts.
- You know the real recovery time for the dataset size.

### 13.3 MongoDB replica set basics

**Objective:** Build a three-node lab or container replica set.

**Tasks:**
1. Start three MongoDB nodes.
2. Initialize with `rs.initiate()`.
3. Create an application user.
4. Stop the primary and observe election behavior.
5. Reconnect using a replica set URI.

**Success criteria:**
- A new primary is elected.
- Clients reconnect with the correct URI.
- You can describe primary, secondary, and election behavior clearly.

### 13.4 Redis durability and failover

**Objective:** Understand what persistence and failover actually protect.

**Tasks:**
1. Run Redis with AOF enabled.
2. Write keys and trigger `BGSAVE`.
3. Restart Redis and verify persistence.
4. Add a replica and a Sentinel set if possible.
5. Observe what happens during primary failure.

**Success criteria:**
- You can explain RDB vs AOF trade-offs.
- Keys survive the restart path you configured.
- You know how Sentinel changes client connection strategy.

### 13.5 Elasticsearch index operations

**Objective:** Practice search-oriented operations instead of generic database habits.

**Tasks:**
1. Create an index with explicit shard and replica settings.
2. Index sample documents.
3. Run a term query and a full-text match query.
4. Check `_cluster/health` and `_cat/shards`.
5. Take and restore a snapshot in a lab repository.

**Success criteria:**
- You can distinguish document indexing from relational writes.
- You can explain why replicas are not backups.
- You can identify cluster health colors and their meaning.

### 13.6 SQLite WAL mode and integrity checks

**Objective:** Learn the limits and strengths of embedded databases.

**Tasks:**
1. Create a local database file.
2. Enable WAL mode.
3. Open concurrent reader and writer sessions.
4. Run `.backup` and `PRAGMA integrity_check;`.
5. Move the workload to PostgreSQL on paper and identify why you would migrate.

**Success criteria:**
- You understand how SQLite handles local concurrency.
- You know how to back it up safely.
- You can articulate when a server database is the better choice.

### 13.7 Suggested weekly practice rotation

| Week | Focus area | Deliverable |
|---|---|---|
| Week 1 | Install and local connectivity | CLI connection screenshots or command transcript |
| Week 2 | Remote access and TLS | Working tunnel or TLS-enabled connection |
| Week 3 | Backup and restore | Successful restore into a clean target |
| Week 4 | Monitoring and failover | Short runbook and observed metrics |
| Week 5 | Security review | Least-privilege matrix per application |
| Week 6 | Containers | Compose stack with persistent volumes |

### 13.8 Cross-engine mastery checklist

- I can install the engine on Linux and start the service.
- I know the default port and how to restrict it with a firewall.
- I can create a database or equivalent namespace and a non-admin user.
- I can connect locally and remotely using CLI tools.
- I can enable or describe TLS for the engine.
- I can take a backup and perform a restore test.
- I can check basic health, storage, and replication status.
- I can explain one good use case and one anti-pattern for the engine.
- I know whether the engine is a source of truth, cache, index, or embedded store in my architecture.
- I have documented the first-response commands for an outage.

