# Backup and Recovery

---

Backups are only useful if they can be restored reliably.
Recovery planning matters as much as backup creation.

## 9.1 Core backup goals

A sound backup plan addresses:
- Recovery point objective.
- Recovery time objective.
- Data scope.
- Retention.
- Storage location.
- Security and encryption.
- Restore testing.

## 9.2 Backup strategy types

### 9.2.1 Full backup

A full backup copies everything in scope.

Pros:
- Simple restore.
- Clear recovery chain.

Cons:
- Largest time and storage cost.

### 9.2.2 Incremental backup

An incremental backup copies data changed since the last backup of any type.

Pros:
- Efficient storage.
- Fast daily backup windows.

Cons:
- Restore may require full plus many increments.

### 9.2.3 Differential backup

A differential backup copies data changed since the last full backup.

Pros:
- Easier restore than long incremental chains.

Cons:
- Grows larger over time until next full backup.

## 9.3 The 3-2-1 principle

A classic backup rule:
- Keep at least 3 copies of data.
- Store them on at least 2 different media types or systems.
- Keep at least 1 copy offsite or logically isolated.

## 9.4 rsync

`rsync` is one of the most useful Linux backup and synchronization tools.

Basic example:

```bash
rsync -avh /srv/data/ /backup/data/
```

Delete extraneous files at destination to mirror source:

```bash
rsync -avh --delete /srv/data/ /backup/data/
```

Remote transfer:

```bash
rsync -avh /srv/data/ backup@example.com:/backup/data/
```

Important flags:
- `-a` archive mode.
- `-v` verbose.
- `-h` human-readable.
- `--delete` remove files at destination that no longer exist at source.
- `-z` compress during transfer.
- `--numeric-ids` preserve numeric ownership across systems.

Dry run first:

```bash
rsync -avhn --delete /srv/data/ /backup/data/
```

## 9.5 tar

`tar` archives files and can compress them.

Create archive:

```bash
tar -cvf backup.tar /etc /var/www
```

Create gzip archive:

```bash
tar -czvf backup.tar.gz /etc /var/www
```

Extract archive:

```bash
tar -xzvf backup.tar.gz -C /restore
```

List contents:

```bash
tar -tvf backup.tar.gz
```

Use `tar` when you need a portable archive bundle.

## 9.6 dd

`dd` copies raw data block by block.
It is powerful and dangerous.

Clone a disk:

```bash
sudo dd if=/dev/sda of=/dev/sdb bs=64K status=progress conv=noerror,sync
```

Create an image file:

```bash
sudo dd if=/dev/sda of=disk-image.img bs=64K status=progress
```

Warnings:
- Reversing `if` and `of` can destroy data.
- Verify source and destination carefully.
- Prefer safer high-level tools when possible.

## 9.7 dump and restore

On some ext-family systems, `dump` and `restore` may still be used.

Example:

```bash
sudo dump -0u -f /backup/root.dump /
sudo restore -t -f /backup/root.dump
```

Availability and relevance vary by distribution and filesystem choice.
Check support before standardizing on it.

## 9.8 LVM snapshots for backup support

LVM snapshots can freeze a consistent point-in-time view.

Typical workflow:
1. Create snapshot.
2. Mount snapshot read-only.
3. Run `rsync` or `tar` from snapshot.
4. Remove snapshot.

Example:

```bash
sudo lvcreate -L 5G -s -n rootsnap /dev/vg0/root
sudo mount -o ro /dev/vg0/rootsnap /mnt/snap
rsync -avh /mnt/snap/ /backup/root/
sudo umount /mnt/snap
sudo lvremove /dev/vg0/rootsnap
```

## 9.9 Btrfs and ZFS snapshots

Btrfs and ZFS include native snapshot capabilities.

Btrfs example:

```bash
sudo btrfs subvolume snapshot -r /data /data-snapshots/data-2024-01-01
```

ZFS example:

```bash
sudo zfs snapshot pool1/data@daily-2024-01-01
```

Snapshot benefits:
- Fast creation.
- Efficient point-in-time capture.
- Support for rollback or replication workflows.

## 9.10 What to back up

Common backup scope includes:
- `/etc`
- Application data under `/srv`, `/var/lib`, or custom paths
- Databases using database-aware backup tools
- Home directories when relevant
- SSL certificates and secrets handled securely
- Infrastructure scripts and config management data

Often not worth backing up directly:
- Reinstallable package caches
- Temporary files
- Ephemeral container layers unless specially required
- Reconstructable build artifacts

## 9.11 Restore testing

A backup without restore testing is only a hope.

Test these regularly:
- Single file restore.
- Full directory restore.
- Bare-metal or VM restore procedure.
- Database restore consistency.
- Application startup after restore.

Document:
- Exact restore commands.
- Credentials and key access process.
- Expected recovery time.
- Validation steps.

## 9.12 Backup verification

Verification methods:
- Compare checksums.
- Run `rsync --checksum` when appropriate.
- Validate archive listing.
- Mount restored snapshot or image.
- Start application with restored data in a test environment.

Examples:

```bash
sha256sum backup.tar.gz
rsync -avhn --checksum /source/ /backup-copy/
tar -tvf backup.tar.gz | head
```

## 9.13 Common backup mistakes

- No restore test.
- Backups stored on same host only.
- Credentials not backed up or recoverable.
- Database files copied without consistency guarantees.
- Silent backup failures not monitored.
- Retention too short.
- `rsync --delete` used without care.
- Snapshots created but never removed.

## 9.14 Practical backup patterns

Config backup:

```bash
rsync -avh /etc/ /backup/etc/
```

Web content backup:

```bash
rsync -avh --delete /var/www/ backup@example.com:/backup/web/
```

Compressed archive of app directory:

```bash
tar -czvf /backup/app-$(date +%F).tar.gz /opt/myapp
```

## 9.15 Recovery planning checklist

- Define critical systems.
- Define acceptable data loss.
- Define acceptable downtime.
- Protect backup credentials.
- Keep at least one isolated copy.
- Test restore quarterly or more often.
- Automate reports and alerting.
- Review capacity and retention.

---

## 13.9 Backup commands reference

- `rsync -avh /src/ /dst/`
- `rsync -avhn --delete /src/ /dst/`
- `tar -cvf backup.tar /path`
- `tar -czvf backup.tar.gz /path`
- `tar -xzvf backup.tar.gz -C /restore`
- `tar -tvf backup.tar.gz`
- `dd if=/dev/sda of=/dev/sdb bs=64K status=progress`
- `dump -0u -f /backup/root.dump /`
- `restore -t -f /backup/root.dump`
- `lvcreate -L 5G -s -n snap /dev/vg0/root`
- `btrfs subvolume snapshot -r /data /snapshots/data-1`
- `zfs snapshot pool/data@daily-1`
- `sha256sum backup.tar.gz`

---

## B.9 Backup quick reminders
- A backup not tested is not trustworthy.
- Protect backup credentials separately.
- Maintain at least one offline or isolated copy where possible.
- Use application-aware backups for databases.
- Use dry-run mode with destructive `rsync` patterns.
- Keep restore instructions near the backup policy.
- Verify retention and pruning behavior.
- Audit backup success daily for critical systems.
- Use snapshots to reduce inconsistency during live backups.
- Store checksums for critical archives.
- Encrypt sensitive backups.
- Test single-file restores frequently.
- Test full environment restore periodically.
- Document RPO and RTO clearly.
- Ensure ownership and permission restoration are validated.

---

### Backup inspection examples
```bash
rsync -avhn --delete /srv/app/ backup:/srv/backup/app/
tar -tvf /backup/app.tar.gz | head
sha256sum /backup/app.tar.gz
```

---

## B.20 Mini runbook: backup verification
1. Pick recent backup set.
2. Verify metadata and checksums.
3. Restore a sample file.
4. Restore a sample directory.
5. If applicable, restore a database to test.
6. Validate permissions and ownership.
7. Validate application readability of restored data.
8. Record restore time.
9. Record issues found.
10. Fix backup process if needed.

---

### Backup cluster
- `rsync -avhn --delete /src/ /dst/`
- `tar -tvf backup.tar.gz`
- `sha256sum backup.tar.gz`
- `zfs list`
- `btrfs subvolume list /mnt`

---

## B.32 More backup examples
```bash
rsync -aHAX --numeric-ids / /backup/root/
ssh backup@example.com 'df -h /backup'
restorecon -Rv /restored/path
```
- Extended attributes and ACLs may matter for correct restores.
- SELinux contexts may need to be restored after data recovery.
- Remote backup capacity should be checked before large jobs begin.
- Bandwidth constraints may require compression or scheduling windows.
- Do not forget to back up custom service units and application config.
- Verify backup exclusions so you do not miss critical state.
- Document encryption key recovery procedures.
- Rotate media or snapshots according to retention policy.
- Keep at least one restore workflow simple enough to run under pressure.
- Practice recovery with the same team that will operate during incidents.
