# SQLite

Usage guidance, CLI examples, backup options, and performance tips for SQLite.
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

---
