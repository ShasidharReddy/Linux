# Loops

> Iteration with for, while, until, select, break, and continue.

## 7. Loops

### 7.1 `for` Loop

Loop over a list:

```bash
for item in one two three; do
  echo "$item"
done
```

### 7.2 `for` Loop Over Files

```bash
for file in *.log; do
  [[ -e $file ]] || continue
  echo "Processing $file"
done
```

### 7.3 C-Style `for` Loop

```bash
for ((i=0; i<5; i++)); do
  echo "$i"
done
```

### 7.4 `while` Loop

```bash
count=1
while (( count <= 3 )); do
  echo "$count"
  ((count++))
done
```

### 7.5 `until` Loop

`until` runs until the condition becomes true.

```bash
count=1
until (( count > 3 )); do
  echo "$count"
  ((count++))
done
```

### 7.6 `select` Loop

Useful for menus.

```bash
select option in start stop quit; do
  case "$option" in
    start) echo "Starting" ;;
    stop) echo "Stopping" ;;
    quit) break ;;
    *) echo "Invalid" ;;
  esac
done
```

### 7.7 `break`

Exit a loop early.

```bash
for n in 1 2 3 4 5; do
  [[ $n -eq 3 ]] && break
  echo "$n"
done
```

### 7.8 `continue`

Skip current iteration.

```bash
for n in 1 2 3 4 5; do
  [[ $n -eq 3 ]] && continue
  echo "$n"
done
```

### 7.9 Loop Over Script Arguments

```bash
for arg in "$@"; do
  echo "Arg: $arg"
done
```

### 7.10 Reading File Line by Line

```bash
while IFS= read -r line; do
  printf 'Line: %s\n' "$line"
done < file.txt
```

Why this is good:

- `IFS=` preserves leading/trailing spaces
- `-r` prevents backslash escaping

### 7.11 Reading Command Output with Process Substitution

```bash
while IFS= read -r user; do
  echo "User: $user"
done < <(cut -d: -f1 /etc/passwd)
```

### 7.12 Loop Over Array

```bash
servers=(web1 web2 db1)
for server in "${servers[@]}"; do
  echo "$server"
done
```

### 7.13 Infinite Loop

```bash
while true; do
  date
  sleep 5
done
```

Alternative:

```bash
for ((;;)); do
  date
  sleep 5
done
```

### 7.14 Loop Control with Multiple Levels

```bash
for i in 1 2 3; do
  for j in a b c; do
    [[ $j == b ]] && break 2
    echo "$i $j"
  done
done
```

### 7.15 Looping Safely Over Filenames

Avoid this:

```bash
for f in $(ls); do
  echo "$f"
done
```

Prefer globbing or `find`.

```bash
for f in ./*; do
  printf '%s\n' "$f"
done
```

### 7.16 `find` with `while read`

```bash
find . -type f -name '*.log' -print0 |
while IFS= read -r -d '' file; do
  printf 'Found: %s\n' "$file"
done
```

### 7.17 Retry Loop Example

```bash
attempt=1
max_attempts=5

until curl -fsS https://example.com/health; do
  if (( attempt >= max_attempts )); then
    echo "Service unavailable"
    exit 1
  fi
  echo "Retry $attempt/$max_attempts"
  ((attempt++))
  sleep 2
done
```

### 7.18 Section Summary

Loops help automate repetition over:

- lists
- files
- lines
- arguments
- retry operations

---
