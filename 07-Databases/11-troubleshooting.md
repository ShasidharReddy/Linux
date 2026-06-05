# Database Troubleshooting

General triage workflows and sample recovery/failover runbook summaries.
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

---

# 11. Sample Runbooks
## 11.1 MySQL restore runbook summary
1. Provision host.
2. Install same compatible version.
3. Stop service.
4. Restore logical or physical backup.
5. Apply binlogs if required.
6. Start service.
7. Validate data.
8. Repoint application if part of failover.

## 11.2 PostgreSQL restore runbook summary
1. Provision host and storage.
2. Restore base backup.
3. Configure WAL archive access.
4. Set recovery target.
5. Start instance.
6. Validate recovery completion.
7. Promote if needed.

## 11.3 MongoDB node replacement summary
1. Confirm replica set health.
2. Remove failed member if necessary.
3. Rebuild node.
4. Rejoin member.
5. Wait for initial sync.
6. Validate replication lag clears.

## 11.4 Redis failover summary
1. Confirm primary failure.
2. Validate Sentinel or Cluster election result.
3. Ensure clients reconnect correctly.
4. Check persistence state.
5. Rebuild failed node as replica.

## 11.5 Elasticsearch red cluster summary
1. Check cluster health.
2. Identify unassigned shards.
3. Confirm disk watermark status.
4. Check failed nodes.
5. Restore capacity or fix allocation blockers.
6. Consider snapshot restore only after diagnosis.

---

---
