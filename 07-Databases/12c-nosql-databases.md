# NoSQL Databases

← Back to [12-database-essentials.md](./12-database-essentials.md)

MongoDB, Redis, Elasticsearch, and containerized NoSQL-oriented workflows.

---

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
