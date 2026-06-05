# 9. Text Processing

## 9.1 Why Text Processing Is a Core Linux Skill

Linux treats text as a universal interface.

Logs, config files, command output, and data exports are often plain text.

Being able to filter and transform text is a superpower.

## 9.2 `grep`

### Purpose

Search text using patterns.

### Common Flags

| Flag | Meaning |
|---|---|
| `-i` | Ignore case |
| `-n` | Show line numbers |
| `-r` | Recursive search |
| `-v` | Invert match |
| `-E` | Extended regex |
| `-w` | Match whole words |
| `-c` | Count matches |

### Examples

```bash
grep root /etc/passwd
grep -i error app.log
grep -rn "Listen" /etc/httpd
grep -v '^#' config.conf
grep -E 'warn|error|fatal' app.log
```

### Real-World Example

Find failed login attempts:

```bash
grep -i 'failed' /var/log/auth.log
```

## 9.3 `sed`

### Purpose

Stream editor for substitutions and line transformations.

### Common Uses

- replace text
- print selected lines
- delete lines
- edit in place

### Examples

```bash
sed 's/foo/bar/' file.txt
sed 's/foo/bar/g' file.txt
sed -n '1,10p' file.txt
sed '/^#/d' config.conf
sed -i 's/localhost/db.internal/' app.conf
```

### Real-World Example

Update a configuration value:

```bash
sed -i 's/^PORT=.*/PORT=8080/' .env
```

> Warning:
> Test `sed` commands without `-i` first so you can review the result before modifying files.

## 9.4 `awk`

### Purpose

Pattern scanning and text processing language.

### Strengths

- column-based extraction
- calculations
- filtering by conditions
- report generation

### Examples

```bash
awk '{print $1}' file.txt
awk -F: '{print $1,$7}' /etc/passwd
awk '$3 > 1000 {print $1,$3}' /etc/passwd
awk '{sum += $1} END {print sum}' numbers.txt
```

### Real-World Example

List usernames and shells:

```bash
awk -F: '{print $1 " -> " $7}' /etc/passwd
```

## 9.5 `cut`

### Purpose

Extract selected parts of each line.

### Examples

```bash
cut -d: -f1 /etc/passwd
cut -d, -f2,4 data.csv
cut -c1-10 file.txt
```

### Real-World Example

Get just usernames from `/etc/passwd`:

```bash
cut -d: -f1 /etc/passwd
```

## 9.6 `sort`

### Purpose

Sort lines of text.

### Common Flags

| Flag | Meaning |
|---|---|
| `-n` | Numeric sort |
| `-r` | Reverse order |
| `-u` | Unique output |
| `-k` | Sort by key field |
| `-t` | Field separator |
| `-h` | Human-readable size sort |

### Examples

```bash
sort names.txt
sort -n numbers.txt
sort -r names.txt
sort -t: -k3 -n /etc/passwd
sort -u users.txt
```

### Real-World Example

Sort disk usage values:

```bash
du -sh * | sort -h
```

## 9.7 `uniq`

### Purpose

Filter or count adjacent duplicate lines.

### Important Note

`uniq` only handles adjacent duplicates.

Use `sort` first when needed.

### Examples

```bash
sort users.txt | uniq
sort users.txt | uniq -c
sort users.txt | uniq -d
```

### Real-World Example

Count repeated IPs in a log:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

## 9.8 `tr`

### Purpose

Translate or delete characters.

### Examples

```bash
echo 'abc' | tr 'a-z' 'A-Z'
tr -d '\r' < windows.txt > unix.txt
echo 'a b c' | tr ' ' '\n'
```

### Real-World Example

Normalize case before comparison:

```bash
echo 'Error' | tr 'A-Z' 'a-z'
```

## 9.9 `paste`

### Purpose

Merge lines from files side by side.

### Examples

```bash
paste file1.txt file2.txt
paste -d',' names.txt roles.txt
```

### Real-World Example

Combine usernames and shells from separate files:

```bash
paste -d':' users.txt shells.txt
```

## 9.10 `diff`

### Purpose

Compare files line by line.

### Examples

```bash
diff old.conf new.conf
diff -u old.conf new.conf
diff -r dir1 dir2
```

### Real-World Example

Compare config before and after a change:

```bash
diff -u app.conf.bak app.conf
```

## 9.11 `comm`

### Purpose

Compare two sorted files line by line.

### Output Columns

- lines only in file1
- lines only in file2
- lines common to both

### Examples

```bash
comm list1.txt list2.txt
comm -12 sorted1.txt sorted2.txt
```

### Real-World Example

Find common usernames between two inventories:

```bash
sort team1.txt > team1-sorted.txt
sort team2.txt > team2-sorted.txt
comm -12 team1-sorted.txt team2-sorted.txt
```

## 9.12 `join`

### Purpose

Join lines of two files on a common field.

### Examples

```bash
join users.txt shells.txt
join -t: -1 1 -2 1 file1 file2
```

### Real-World Example

Join an ID list with a name list:

```bash
join -t, ids.csv names.csv
```

## 9.13 End-to-End Text Pipelines

### Example 1: Top IP addresses in a web log

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

### Example 2: Non-comment configuration lines

```bash
grep -v '^#' app.conf | sed '/^$/d'
```

### Example 3: Extract usernames and shells

```bash
cut -d: -f1,7 /etc/passwd
```

### Example 4: Search and summarize errors

```bash
grep -i error app.log | awk '{print $5}' | sort | uniq -c | sort -nr
```

## 9.14 Choosing the Right Tool

| Need | Tool |
|---|---|
| Search lines by pattern | `grep` |
| Replace or rewrite streams | `sed` |
| Column logic and calculations | `awk` |
| Extract fields | `cut` |
| Order lines | `sort` |
| Collapse duplicates | `uniq` |
| Translate characters | `tr` |
| Merge files side by side | `paste` |
| Compare files | `diff` |
| Compare sorted lists | `comm` |
| Join records by key | `join` |

## 9.15 Regex Basics for Text Processing

A few useful regex concepts:

| Pattern | Meaning |
|---|---|
| `^` | Start of line |
| `$` | End of line |
| `.` | Any single character |
| `*` | Zero or more of previous item |
| `+` | One or more of previous item |
| `[abc]` | Any listed character |
| `[^abc]` | Any character not listed |

Examples:

```bash
grep '^root:' /etc/passwd
grep 'bash$' /etc/passwd
grep -E 'error|warn' app.log
```

## 9.16 Best Practices

- test commands on small samples first
- use `sort` before `uniq` when required
- avoid `sed -i` until you confirm the substitution
- quote patterns to avoid shell expansion
- learn one good log-analysis pipeline and reuse it

> Tip:
> `grep`, `awk`, `sort`, and `uniq` together solve a surprising percentage of real Linux troubleshooting problems.

---

