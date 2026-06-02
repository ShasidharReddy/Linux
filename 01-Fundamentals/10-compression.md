# 12. Compression and Archiving

## 12.1 Archive vs Compression

These are related but not identical.

- archiving combines files into one package
- compression reduces size

`tar` is primarily an archiver.

`gzip`, `bzip2`, and `xz` are compressors.

## 12.2 `tar`

### Purpose

Create or extract archives.

### Common Flags

| Flag | Meaning |
|---|---|
| `-c` | Create archive |
| `-x` | Extract archive |
| `-t` | List archive contents |
| `-f` | Archive file name |
| `-v` | Verbose |
| `-z` | Use gzip |
| `-j` | Use bzip2 |
| `-J` | Use xz |

### Examples

```bash
tar -cvf project.tar project/
tar -xvf project.tar
tar -tvf project.tar
tar -czvf project.tar.gz project/
tar -xzvf project.tar.gz
tar -cJvf logs.tar.xz /var/log
```

## 12.3 `gzip`

### Purpose

Compress files using gzip.

### Examples

```bash
gzip file.txt
gzip -k file.txt
gzip -d file.txt.gz
```

### Notes

By default, `gzip` replaces the original file unless `-k` is used.

## 12.4 `bzip2`

### Purpose

Compress files using bzip2.

### Examples

```bash
bzip2 report.txt
bzip2 -dk report.txt.bz2
```

### Notes

Often compresses better than gzip, but can be slower.

## 12.5 `xz`

### Purpose

High compression ratio for single files.

### Examples

```bash
xz bigfile.log
xz -dk bigfile.log.xz
```

### Notes

`xz` is common for distribution images and large text archives.

## 12.6 `zip` and `unzip`

### Purpose

Create and extract ZIP archives.

### Examples

```bash
zip -r project.zip project/
unzip project.zip
unzip -l project.zip
```

### Notes

ZIP is very common for cross-platform exchange.

## 12.7 `zcat` and `zgrep`

### Purpose

Read or search compressed gzip files without manually decompressing them first.

### Examples

```bash
zcat app.log.gz | head
zgrep -i error app.log.gz
```

## 12.8 Common Archive Extensions

| Extension | Meaning |
|---|---|
| `.tar` | Uncompressed tar archive |
| `.tar.gz` or `.tgz` | tar archive compressed with gzip |
| `.tar.bz2` | tar archive compressed with bzip2 |
| `.tar.xz` | tar archive compressed with xz |
| `.zip` | ZIP archive |
| `.gz` | gzip-compressed single file |
| `.bz2` | bzip2-compressed single file |
| `.xz` | xz-compressed single file |

## 12.9 Practical Examples

### Back up a configuration directory

```bash
sudo tar -czvf etc-backup.tar.gz /etc
```

### Archive logs older than today

```bash
tar -czvf logs.tar.gz /var/log
```

### Inspect a compressed log

```bash
zgrep -i fail /var/log/auth.log.1.gz
```

### Extract to a specific directory

```bash
mkdir restore
 tar -xvf project.tar -C restore
```

## 12.10 Best Practices

- use `tar -tvf` before extraction if archive contents are unknown
- extract into a dedicated directory when unsure
- preserve permissions with tar archives when backing up Unix data
- use gzip for speed, xz for smaller size

> Warning:
> Be careful when extracting archives from untrusted sources.
> They may contain unexpected paths or overwrite files if extracted carelessly.

---

