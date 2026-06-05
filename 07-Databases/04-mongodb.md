# MongoDB

Installation, CRUD, replica set, sharding, backup, and operational guidance for MongoDB.
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

---

# 4. Extended MongoDB Operations Guide
## 4.1 Replica set health checks
```javascript
rs.status()
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()
```

## 4.2 Database statistics
```javascript
db.stats()
db.serverStatus()
```

## 4.3 Collection stats
```javascript
db.orders.stats()
```

## 4.4 Index review
```javascript
db.orders.getIndexes()
```

## 4.5 Profiling strategy
Use profiling only as needed and with awareness of workload overhead.

Levels:

- 0: off
- 1: slow operations
- 2: all operations

## 4.6 Common admin patterns
- Keep admin database credentials separate from app credentials.
- Prefer replica sets even for small production deployments.
- Avoid arbiters unless you really need them.
- Validate shard rebalancing after capacity changes.

## 4.7 Backup checklist for MongoDB
- Use auth-enabled backup user.
- Take backups from a secondary when possible.
- Ensure oplog coverage if consistent point restore is needed.
- Test restore into isolated environment.

## 4.8 Storage considerations
- Monitor WiredTiger cache utilization.
- Watch filesystem free space.
- Separate journal and data only when justified by platform design.
- Use XFS where vendor guidance recommends it.

## 4.9 Common anti-patterns
- Giant unbounded arrays in one document.
- Missing indexes on shard key prefixes.
- Treating MongoDB like a generic dump for arbitrary blobs.
- Ignoring schema validation when applications evolve.

## 4.10 Schema validation example
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

---
