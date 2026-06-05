# MySQL and PostgreSQL

← Back to [12-database-essentials.md](./12-database-essentials.md)

Setup, connectivity, remote access, and core administration for the major relational engines.

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
