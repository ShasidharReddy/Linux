# Database Administration

Cross-engine administration, monitoring, backups, performance, Linux operations, and operational review material.
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

---

# 8. Appendix: Linux DBA Command Reference
## 8.1 Process and service commands
```bash
systemctl status mysql --no-pager
systemctl status postgresql --no-pager
systemctl status mongod --no-pager
systemctl status redis --no-pager
systemctl status elasticsearch --no-pager
journalctl -u mysql -n 100 --no-pager
journalctl -u postgresql -n 100 --no-pager
```

## 8.2 Networking commands
```bash
ss -tulpn
ip addr
ip route
ethtool eth0
sar -n TCP,DEV 1
```

## 8.3 Disk and memory commands
```bash
free -h
vmstat 1
iostat -xz 1
lsblk
df -h
```

## 8.4 File permission hygiene
Typical examples:

```bash
chown -R mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql
chown postgres:postgres /var/lib/postgresql/16/main/server.key
chmod 600 /var/lib/postgresql/16/main/server.key
```

## 8.5 Example backup validation checklist
- Verify backup job completed successfully.
- Verify checksum or artifact size expectations.
- Verify backup uploaded to remote storage.
- Perform periodic full restore test.
- Confirm application can read restored data.

## 8.6 Database comparison quick reference
| Engine | Strength | Primary caution |
|---|---|---|
| MySQL/MariaDB | Mature OLTP and web app ecosystem | Careful replication and schema-change planning needed |
| PostgreSQL | Rich SQL, extensions, correctness | Connection pooling often required at scale |
| MongoDB | Flexible documents and scaling patterns | Schema discipline and shard-key design are crucial |
| Redis | Very fast in-memory operations | Memory management and persistence choices matter |
| Elasticsearch | Search and analytics power | Shard/JVM sizing and ops discipline are essential |
| SQLite | Simple embedded database | Limited concurrent writes |

## 8.7 Example production readiness checklist
- Capacity forecast documented.
- Metrics and alerting configured.
- Backup restore tested.
- Security controls reviewed.
- Upgrade path documented.
- Failover tested.
- On-call runbook updated.

---

---

# 8. Database Backups Deep Dive
## 8.1 Backup objectives
Define clearly:

- RPO: how much data loss is acceptable.
- RTO: how fast service must recover.
- Retention: how long backups are kept.
- Immutability: whether backups are protected from deletion.

## 8.2 Full vs incremental vs differential
| Type | Description | Pros | Cons |
|---|---|---|---|
| Full | Entire dataset | Simplest restore | Large backup window |
| Incremental | Changes since last backup | Efficient storage | More complex restore chain |
| Differential | Changes since last full | Simpler than incremental chain | Grows over time |

## 8.3 Logical vs physical backups
| Type | Best for | Cautions |
|---|---|---|
| Logical | Portability and small datasets | Slower restore at scale |
| Physical | Large production systems | Tied more closely to engine/version/filesystem details |

## 8.4 Backup validation checklist
- Backup completed without warnings.
- Checksum validated.
- Artifact is restorable.
- Permissions are correct.
- Encryption is confirmed.
- Offsite copy succeeded.
- Monitoring alert exists for failures.

## 8.5 Retention policy example
- Daily backups: 7 days.
- Weekly backups: 5 weeks.
- Monthly backups: 12 months.
- Yearly backups: as required by policy.

## 8.6 Restore drill template
1. Choose sample backup set.
2. Provision isolated target host.
3. Restore according to runbook.
4. Verify engine starts cleanly.
5. Run integrity checks.
6. Run application smoke tests.
7. Record total recovery time.
8. Update documentation.

## 8.7 Backup storage locations
Options:

- Local disk for rapid operational restore.
- Network-attached storage.
- Object storage.
- Tape or archival cold storage.
- Immutable storage tiers.

## 8.8 Backup encryption workflow ideas
- Encrypt on source before upload.
- Use KMS-integrated storage encryption.
- Rotate encryption keys.
- Restrict decryption rights separately from backup read rights.

## 8.9 Air-gap and ransomware considerations
- Maintain offline or immutable copies.
- Separate backup admin credentials from production credentials.
- Test disaster restore under restricted network assumptions.

## 8.10 Backup monitoring examples
Alert when:

- Latest successful backup age exceeds threshold.
- Restore test has not occurred within policy window.
- Backup size deviates unexpectedly.
- WAL/binlog archival stops.

---

---

# 8. Database Performance Deep Dive
## 8.1 Performance hierarchy
Typical order of investigation:

1. Bad query pattern.
2. Missing or poor index.
3. Connection saturation.
4. Memory pressure.
5. Storage latency.
6. Network bottleneck.
7. Inefficient schema design.

## 8.2 Metrics that matter
Cross-engine metrics:

- Query latency percentiles.
- Throughput.
- Cache hit ratio.
- Disk read/write latency.
- Replication delay.
- Lock wait time.
- Active sessions.
- Temp file generation.

## 8.3 Benchmarking cautions
- Production-like data shape matters.
- Concurrency matters.
- Cache warm-up matters.
- Synthetic benchmarks can mislead.
- Always compare before and after under same conditions.

## 8.4 Index lifecycle management
Indexes should be:

- Proposed.
- Load tested.
- Measured for write impact.
- Reviewed for redundancy later.
- Dropped if unused and costly.

## 8.5 Query plan literacy
Always ask:

- Is the access path selective?
- Are row estimates accurate?
- Is sorting spilling to disk?
- Is join order sensible?
- Is parallelism helping or hurting?

## 8.6 Host tuning reminders
- Disable swap for Elasticsearch and often for latency-sensitive DB patterns where policy allows.
- Reserve RAM for filesystem cache.
- Use tuned profiles where appropriate.
- Verify scheduler and NUMA considerations on high-end servers.

## 8.7 Application collaboration
Database performance problems are often application problems.

Work jointly on:

- Query batching.
- Retry storm prevention.
- Connection pool sizing.
- Idempotent write design.
- Pagination strategy.

## 8.8 Pagination design
Avoid deep offset pagination for large datasets.

Prefer keyset pagination when possible.

Example idea:

```sql
SELECT * FROM orders
WHERE created_at < '2025-01-01T10:00:00Z'
ORDER BY created_at DESC
LIMIT 50;
```

## 8.9 Hotspot detection
Look for:

- One table dominating writes.
- One shard dominating load.
- One index causing contention.
- One tenant causing disproportionate activity.

## 8.10 Performance review cadence
- Daily: critical latency and saturation alerts.
- Weekly: top query and growth review.
- Monthly: capacity and tuning backlog review.

---

---

# 8. Linux Operational Playbooks
## 8.1 Check listening ports
```bash
ss -tulpn | egrep '3306|5432|27017|6379|9200'
```

## 8.2 Check top IO consumers
```bash
iotop -o
```

## 8.3 Check CPU pressure
```bash
mpstat -P ALL 1
pidstat -dur 1
```

## 8.4 Check memory pressure
```bash
free -h
vmstat 1
sar -r 1
```

## 8.5 Check large files
```bash
find /var/lib -type f -size +1G -ls | head
```

## 8.6 Check open file limits
```bash
ulimit -n
cat /proc/$(pgrep -o mysqld)/limits | grep 'open files'
```

## 8.7 Check time sync
```bash
timedatectl
chronyc tracking
```

## 8.8 Firewall examples
```bash
sudo firewall-cmd --list-all
sudo ufw status verbose
```

## 8.9 Capture a support bundle mindset
When investigating incidents, gather:

- Service status.
- Logs.
- Config excerpts.
- Top queries.
- Disk and memory state.
- Replication state.
- Timeline of changes.

## 8.10 Change management reminder
For planned database changes:

- Open change record.
- Define backout plan.
- Notify stakeholders.
- Freeze conflicting work.
- Validate before and after.

---

---

# 8. Quick Reference Tables
## 8.1 Default ports
| Engine | Default port |
|---|---|
| MySQL/MariaDB | 3306 |
| PostgreSQL | 5432 |
| MongoDB | 27017 |
| Redis | 6379 |
| Elasticsearch | 9200 HTTP, 9300 transport |
| SQLite | N/A |

## 8.2 Key config files
| Engine | Main config |
|---|---|
| MySQL | /etc/mysql/my.cnf or /etc/my.cnf |
| PostgreSQL | postgresql.conf, pg_hba.conf |
| MongoDB | /etc/mongod.conf |
| Redis | /etc/redis/redis.conf |
| Elasticsearch | /etc/elasticsearch/elasticsearch.yml |
| SQLite | Application-defined file |

## 8.3 Backup tools
| Engine | Common tools |
|---|---|
| MySQL | mysqldump, mysqlpump, xtrabackup |
| PostgreSQL | pg_dump, pg_basebackup, pgBackRest |
| MongoDB | mongodump, filesystem snapshots |
| Redis | RDB/AOF copy with coordination |
| Elasticsearch | Snapshot API |
| SQLite | .backup, .dump |

## 8.4 HA tools
| Engine | Common HA options |
|---|---|
| MySQL | InnoDB Cluster, Galera, ProxySQL |
| PostgreSQL | Patroni, etcd, pgpool-II, HAProxy |
| MongoDB | Replica sets, sharding |
| Redis | Sentinel, Redis Cluster |
| Elasticsearch | Native cluster redundancy |
| SQLite | Usually application-level redundancy |

---

---

# 8. Final Recommendations
## 8.1 Start simple
The simplest architecture that meets requirements is usually the best first production architecture.

## 8.2 Prefer operational maturity over novelty
A team that deeply understands PostgreSQL or MySQL will often outperform a more exotic system deployed without deep expertise.

## 8.3 Backups are not real until restored
Always verify recoverability.

## 8.4 Security is part of administration
Database admin work includes access control, encryption, and auditability, not just uptime.

## 8.5 Performance is end-to-end
Query design, schema design, application behavior, Linux tuning, and storage quality all matter.

## 8.6 Measure continuously
Use dashboards, logs, slow query analysis, and periodic reviews.

## 8.7 Document everything important
Good runbooks reduce outage duration and upgrade risk.

---

---

# 8. Line-Filler Knowledge Cards for Operational Review
> The following cards intentionally provide compact, review-friendly operational reminders to make this guide comprehensive and reference-dense.

## 8.1 Card 001
- Prefer vendor-supported repositories for production upgrades.
- Keep version pinning explicit.
- Rehearse rollback before major version change.

## 8.2 Card 002
- Use dedicated backup users.
- Limit privileges to backup-related actions.
- Protect credential files with strict permissions.

## 8.3 Card 003
- Monitor disk latency, not only throughput.
- Databases can fail under latency spikes even when bandwidth seems fine.
- Storage tail latency matters.

## 8.4 Card 004
- Time sync matters for logs, certificates, and replication analysis.
- Run chrony or equivalent.
- Alert on drift.

## 8.5 Card 005
- Name databases, schemas, tables, and indexes consistently.
- Standard naming reduces troubleshooting mistakes.
- Conventions save time.

## 8.6 Card 006
- Retain enough WAL/binlog for recovery.
- Monitor archive success continuously.
- Restore drills should include replay.

## 8.7 Card 007
- Avoid manual changes on replicas without a runbook.
- Divergence is easy to create and painful to repair.
- Prefer declarative automation.

## 8.8 Card 008
- Track schema migration duration historically.
- Large-table changes can grow non-linearly.
- Treat them like risky production events.

## 8.9 Card 009
- Keep application connection pools bounded.
- Unlimited client pools can overload the server instantly.
- Pooling needs capacity planning too.

## 8.10 Card 010
- Review dead letter or failed ingestion pipelines feeding search systems.
- Search freshness issues are often pipeline issues, not engine bugs.
- Trace end-to-end.

## 8.11 Card 011
- Maintain separate dashboards for host metrics and query metrics.
- Both views are necessary.
- One without the other can mislead.

## 8.12 Card 012
- Validate filesystem ownership after physical restore.
- Many restore failures are simple permission problems.
- Fix ownership before restart.

## 8.13 Card 013
- Write-heavy workloads need careful checkpoint and log tuning.
- Measure before changing defaults.
- Durability trade-offs must be approved.

## 8.14 Card 014
- Compression reduces backup storage but can increase restore CPU time.
- Pick settings aligned to RTO goals.
- Balance cost and speed.

## 8.15 Card 015
- Document where certificates live on disk.
- Expired TLS certs can create avoidable outages.
- Track renewal ownership.

## 8.16 Card 016
- Avoid app-side timezone ambiguity.
- Store UTC where practical.
- Convert at presentation layer.

## 8.17 Card 017
- Use separate monitoring credentials.
- Read-only visibility is usually enough.
- Avoid over-privileged observability agents.

## 8.18 Card 018
- Large deletes can be operationally expensive.
- Prefer partition drops or chunked deletes.
- Control bloat and replication lag.

## 8.19 Card 019
- Always know your biggest tables and indexes.
- Capacity surprises often come from a few objects.
- Rank them regularly.

## 8.20 Card 020
- Validate application retry behavior during failover.
- Good infrastructure fails gracefully only with good clients.
- Chaos testing helps.

## 8.21 Card 021
- Monitor for backup size anomalies.
- Sudden drops can indicate incomplete dumps.
- Sudden growth can indicate data explosion.

## 8.22 Card 022
- Tune autovacuum before wraparound becomes urgent.
- Preventive work is cheaper than emergency response.
- Watch dead tuples early.

## 8.23 Card 023
- In Redis, define what data may be lost.
- Cache data and durable data deserve different persistence choices.
- Align policy to business needs.

## 8.24 Card 024
- In MongoDB, validate schema even if using a flexible document model.
- Flexibility without guardrails becomes debt.
- Enforce critical fields.

## 8.25 Card 025
- In Elasticsearch, map fields intentionally.
- Dynamic mapping can explode field counts.
- Prevent cluster state bloat.

## 8.26 Card 026
- Prefer idempotent migration scripts when possible.
- Safe re-runs reduce deployment risk.
- CI should catch drift.

## 8.27 Card 027
- Separate admin traffic from app traffic when feasible.
- Incident access should not compete with production load.
- Bastions help.

## 8.28 Card 028
- Avoid large transactions during peak load.
- They increase lock duration, WAL/binlog churn, and recovery work.
- Batch safely.

## 8.29 Card 029
- Keep a dependency inventory of backup, monitoring, and failover tools.
- Tool drift causes silent operational risk.
- Update alongside engine upgrades.

## 8.30 Card 030
- Track restore time as a first-class SLO input.
- Backup frequency alone is not enough.
- Recovery speed matters.

## 8.31 Card 031
- Capture query fingerprints, not just raw statements.
- Fingerprints group similar workload classes.
- This improves prioritization.

## 8.32 Card 032
- Review replication slots or equivalent retention mechanisms.
- Stalled consumers can fill disks.
- Alert on backlog growth.

## 8.33 Card 033
- Use canary queries in monitoring.
- They reveal latency from the application point of view.
- Synthetic probes are valuable.

## 8.34 Card 034
- Keep emergency free space on data volumes.
- Zero-free-space incidents are harder to recover from.
- Reserve headroom operationally.

## 8.35 Card 035
- Benchmark with realistic concurrency.
- Single-user results rarely predict production behavior.
- Use production-like query mixes.

## 8.36 Card 036
- Store infrastructure-as-code for database-adjacent automation.
- Manual topology drift is dangerous.
- Version control runbooks and configs.

## 8.37 Card 037
- Observe queue depth in storage metrics.
- Rising await plus queue depth signals saturation.
- Application latency usually follows.

## 8.38 Card 038
- Keep Linux kernel, filesystem, and database versions in compatibility review.
- Ops problems can sit at boundaries.
- Test the full stack.

## 8.39 Card 039
- For high connection counts, investigate pooling before scaling database backends.
- Backend process overhead is real.
- Pooling is often the simplest win.

## 8.40 Card 040
- Validate failover DNS TTL behavior.
- Routing changes are only as fast as clients respect them.
- Proxy layers reduce surprises.

## 8.41 Card 041
- Rehearse certificate rotation in non-production.
- Hot-reload behavior differs by engine.
- Avoid expiry-day improvisation.

## 8.42 Card 042
- Use separate users for schema migrations and applications.
- Apps do not need DDL in steady state.
- Reduce blast radius.

## 8.43 Card 043
- Review crash recovery duration after checkpoints and log tuning changes.
- Faster steady-state writes can slow recovery.
- Balance both objectives.

## 8.44 Card 044
- Treat search indexes as rebuildable if architecture permits.
- This can simplify backup priorities.
- But document rebuild time.

## 8.45 Card 045
- Protect snapshot repositories and backup buckets with strict IAM.
- Backup theft is a real data breach vector.
- Logs alone are insufficient.

## 8.46 Card 046
- Avoid world-readable config files.
- Secrets often leak through permissive defaults.
- Audit file modes regularly.

## 8.47 Card 047
- Track top 10 queries by total time and by mean latency.
- The rankings differ and both matter.
- Optimize accordingly.

## 8.48 Card 048
- Use read replicas for read scaling, not as a substitute for query tuning.
- Inefficient queries stay inefficient.
- They just move the pain.

## 8.49 Card 049
- When using containers, understand the storage driver and PVC semantics.
- Databases are sensitive to hidden storage behavior.
- Benchmark the actual platform.

## 8.50 Card 050
- Keep a restoration target matrix.
- Know which system restores where and how fast.
- Incidents reward preparation.

## 8.51 Card 051
- Track schema ownership by team.
- Orphaned tables and indexes accumulate quickly.
- Ownership drives cleanup.

## 8.52 Card 052
- Observe lock wait graphs during traffic peaks.
- Some contention is time-window dependent.
- Sample at the right times.

## 8.53 Card 053
- Prefer append-only event retention for auditing sensitive changes.
- Mutable audit trails are weak controls.
- Integrity matters.

## 8.54 Card 054
- Review high-cardinality metrics carefully.
- Observability tooling itself can become noisy or expensive.
- Keep labels disciplined.

## 8.55 Card 055
- Consider tenant isolation requirements early.
- Shared schema, separate schema, and separate database models differ significantly.
- Migrations and backup scope change too.

## 8.56 Card 056
- Runbook steps should include validation checks after every destructive action.
- Action without verification is risky.
- Write explicit success criteria.

## 8.57 Card 057
- Audit dormant privileged accounts.
- Old admin accounts are common risk.
- Remove or rotate regularly.

## 8.58 Card 058
- Protect monitoring endpoints and exporters.
- They can leak topology and performance data.
- Security includes observability.

## 8.59 Card 059
- Review biggest temp-file generators.
- Sort and hash spills are performance signals.
- They often point to missing indexes or memory mis-sizing.

## 8.60 Card 060
- Use staged rollouts for driver upgrades.
- Client behavior changes can impact pooling, TLS, and retries.
- Test real traffic patterns.

## 8.61 Card 061
- Document replication topology clearly.
- Human misunderstanding during incidents causes secondary outages.
- Diagrams matter.

## 8.62 Card 062
- For SQLite, ensure filesystem semantics on network storage are understood.
- Not all network filesystems behave safely.
- Local disks are simpler.

## 8.63 Card 063
- Review kernel dmesg during storage incidents.
- Filesystem and block device warnings can explain database stalls.
- Look below the engine.

## 8.64 Card 064
- Monitor CPU steal time in virtualized environments.
- Neighbor noise can look like database slowness.
- Host context matters.

## 8.65 Card 065
- When tuning caches, leave room for OS cache and auxiliary processes.
- 100 percent RAM allocation plans usually age badly.
- Keep headroom.

## 8.66 Card 066
- Version control configuration snippets.
- Drifted tuning changes are hard to explain months later.
- Commit them intentionally.

## 8.67 Card 067
- Record exact build and package versions during incidents.
- Reproducing bugs requires precise versions.
- Details matter.

## 8.68 Card 068
- For replication lag, distinguish network lag from apply lag.
- Different causes need different fixes.
- Inspect both sides.

## 8.69 Card 069
- Review table and index bloat trends, not only point-in-time sizes.
- Growth rate reveals maintenance issues.
- Trending beats snapshots.

## 8.70 Card 070
- Keep a tested emergency credential access process.
- Break-glass access should be controlled and auditable.
- Do not improvise during outages.

## 8.71 Card 071
- Avoid changing too many tuning variables at once.
- One change at a time preserves learning.
- Measure effects clearly.

## 8.72 Card 072
- Query timeouts protect systems from runaway workload.
- Apply at application and database layers where appropriate.
- Default forever is risky.

## 8.73 Card 073
- Monitor redo/WAL/binlog generation rate during launches.
- It predicts storage and backup pressure.
- Growth events often surprise teams.

## 8.74 Card 074
- Use prepared statements or server-side plan caches carefully.
- They improve efficiency but can have plan stability effects.
- Measure workload impact.

## 8.75 Card 075
- Keep failover authority clear.
- Ambiguous ownership slows incidents.
- Define who can promote, cut over, and rollback.

## 8.76 Card 076
- Review compression settings for hot paths.
- CPU savings and storage savings can trade off against latency.
- Align to workload priorities.

## 8.77 Card 077
- Protect against thundering herds after cache expiry.
- Use jitter, soft TTLs, or request coalescing.
- Databases feel cache mistakes immediately.

## 8.78 Card 078
- Confirm backup tools support engine version features you use.
- Tool compatibility can lag.
- Upgrade both intentionally.

## 8.79 Card 079
- Record primary keys and unique constraints as business invariants, not just technical features.
- They encode correctness.
- Preserve them in migrations.

## 8.80 Card 080
- Review optimizer statistics freshness.
- Bad stats lead to bad plans.
- Maintenance matters.

## 8.81 Card 081
- Use maintenance windows for disruptive operations when possible.
- Online does not always mean impact-free.
- Communicate risk.

## 8.82 Card 082
- When troubleshooting, separate symptom metrics from causal metrics.
- High CPU might be cause or effect.
- Build a timeline.

## 8.83 Card 083
- Keep application and database clocks aligned for time-based pagination and conflict detection.
- Time drift breaks logic.
- Sync matters beyond logging.

## 8.84 Card 084
- Review backup retention compliance after storage class changes.
- Cheap storage migrations can break governance.
- Validate policy implementation.

## 8.85 Card 085
- Prefer explicit application schemas over dumping everything into public or default spaces.
- Structure improves privilege management.
- Organization helps long-term maintenance.

## 8.86 Card 086
- Capture baseline performance before major releases.
- Without baseline, regressions are harder to prove.
- Benchmark routinely.

## 8.87 Card 087
- For document stores, version document schemas explicitly when applications evolve rapidly.
- Migration clarity improves safety.
- Flexibility still needs governance.

## 8.88 Card 088
- Review TLS protocol and cipher policies periodically.
- Security baselines change.
- Keep pace with standards.

## 8.89 Card 089
- Ensure your monitoring covers unsuccessful backups, not only successful ones.
- Silence is dangerous.
- Alert on absence too.

## 8.90 Card 090
- Keep a map of data classification by database and schema.
- Security controls depend on knowing where sensitive data lives.
- Inventory first.

## 8.91 Card 091
- Watch for retry storms after brief outages.
- Recovery traffic can extend incidents.
- Rate-limit or circuit-break when needed.

## 8.92 Card 092
- Log rotation must account for engine-specific reopen behavior.
- Coordinate with service reloads if required.
- Avoid losing logs.

## 8.93 Card 093
- Validate read-after-write expectations when using replicas.
- Some application paths need primary reads.
- Consistency choices are application features.

## 8.94 Card 094
- Prefer automation for recurring failover and backup tasks, but keep manual runbooks current.
- Automation also fails.
- Humans need a fallback path.

## 8.95 Card 095
- Check memory overcommit and swap policy for latency-sensitive services.
- Linux VM settings can influence failure modes.
- Understand host defaults.

## 8.96 Card 096
- Use `EXPLAIN` before and after schema changes that affect critical queries.
- Plans can regress unexpectedly.
- Validate hot paths.

## 8.97 Card 097
- For search clusters, separate ingest-heavy and query-heavy roles when scale justifies it.
- Workload isolation can improve stability.
- Architecture should match traffic.

## 8.98 Card 098
- During restores, verify ownership, permissions, SELinux context, and service unit expectations.
- Startup failures are often environmental.
- Linux details matter.

## 8.99 Card 099
- Review and prune obsolete indexes periodically.
- Old indexes quietly tax write performance.
- Keep the set intentional.

## 8.100 Card 100
- Every database team should know its last successful restore test date.
- If nobody knows, the risk is already too high.
- Put it on the dashboard.

---

---

# 8. Closing Notes
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

---
