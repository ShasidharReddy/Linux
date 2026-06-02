# Backup and Restore

This guide covers backups, restore validation, automation, and data protection workflows.

## 7.1 Backup principles

- Follow 3-2-1 backup strategy.
- Encrypt backups in transit and at rest.
- Verify backups regularly.
- Test restores, not just backup jobs.
- Separate backup credentials.
- Use immutable storage where possible.
- Monitor backup age and success.

## 7.2 rsync backup script

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC="/etc /var/www /opt/app"
DEST="backup@example.com:/backups/$(hostname -s)/daily"
LOGFILE="/var/log/rsync-backup.log"

/usr/bin/rsync -aHAX --delete --numeric-ids --stats $SRC "$DEST" >> "$LOGFILE" 2>&1
```

## 7.3 rsync with exclusions

```bash
rsync -aHAX --delete \
  --exclude='/var/cache/' \
  --exclude='/var/tmp/' \
  --exclude='*.log' \
  / backup@example.com:/backups/server01/rootfs/
```

## 7.4 Database backup for MySQL or MariaDB

### Full dump

```bash
mysqldump --single-transaction --routines --triggers --events -u root -p --all-databases | gzip > all-databases-$(date +%F).sql.gz
```

### Single database

```bash
mysqldump --single-transaction -u backup -p appdb | gzip > appdb-$(date +%F-%H%M).sql.gz
```

### Backup all schemas separately

```bash
mysql -NBe 'show databases' | egrep -v '^(information_schema|performance_schema|mysql|sys)$' | while read -r db; do
  mysqldump --single-transaction "$db" | gzip > "${db}-$(date +%F).sql.gz"
done
```

## 7.5 Database backup for PostgreSQL

### Full cluster logical backup

```bash
pg_dumpall -U postgres | gzip > pgdumpall-$(date +%F).sql.gz
```

### Single database custom format

```bash
pg_dump -U postgres -Fc appdb > appdb-$(date +%F).dump
```

### Schema-only backup

```bash
pg_dump -U postgres -s appdb > appdb-schema-$(date +%F).sql
```

## 7.6 Automated backup with cron

```cron
0 2 * * * /usr/local/bin/rsync-backup.sh
30 2 * * * /usr/local/bin/mysql-backup.sh
0 3 * * 0 /usr/local/bin/backup-verify.sh
```

## 7.7 Backup verification

### Verify file exists and size is sane

```bash
ls -lh *.sql.gz *.dump 2>/dev/null
```

### Test gzip archive

```bash
gzip -t appdb-2025-01-01.sql.gz
```

### Validate MySQL dump header

```bash
zcat appdb-2025-01-01.sql.gz | head -20
```

### Validate PostgreSQL dump

```bash
pg_restore -l appdb-2025-01-01.dump | head -20
```

### Verify rsync backup freshness

```bash
find /backups/server01 -type f -mtime -1 | head
```

## 7.8 Restore procedures

### Restore files with rsync

```bash
rsync -aHAX backup@example.com:/backups/server01/daily/etc/ /etc/
```

### Restore MySQL dump

```bash
gunzip -c appdb-2025-01-01.sql.gz | mysql -u root -p appdb
```

### Restore PostgreSQL custom dump

```bash
createdb -U postgres appdb_restore
pg_restore -U postgres -d appdb_restore appdb-2025-01-01.dump
```

### Restore PostgreSQL plain SQL

```bash
gunzip -c pgdumpall-2025-01-01.sql.gz | psql -U postgres
```

## 7.9 Example backup verification script

```bash
#!/usr/bin/env bash
set -euo pipefail

LATEST=$(ls -1t /backups/db/appdb-*.sql.gz | head -1)
[ -n "$LATEST" ] || { echo "No backup found"; exit 1; }
gzip -t "$LATEST"
SIZE=$(stat -c%s "$LATEST")
[ "$SIZE" -gt 1048576 ] || { echo "Backup too small"; exit 1; }
echo "Backup verification passed: $LATEST"
```

## 7.10 Backup operations checklist

- Confirm source paths.
- Confirm exclusion rules.
- Confirm destination reachable.
- Confirm free space at destination.
- Confirm encryption if required.
- Confirm retention job exists.
- Confirm verification job exists.
- Confirm restore test completed recently.

## 7.11 Backup one-liners

```bash
find /backups -type f -printf '%TY-%Tm-%Td %TT %s %p
' | sort | tail -20
```

```bash
ssh backup@example.com 'df -h /backups && find /backups/server01 -type f | wc -l'
```

---
