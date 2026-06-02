# 10. I/O Redirection and Piping

## 10.1 Standard Streams

Linux commands usually work with three standard streams.

| Stream | Number | Purpose |
|---|---|---|
| stdin | 0 | Input to a command |
| stdout | 1 | Normal output |
| stderr | 2 | Error output |

## 10.2 Basic Output Redirection

### `>`

Redirect stdout to a file and overwrite it.

```bash
echo "hello" > file.txt
ls > listing.txt
```

### `>>`

Append stdout to a file.

```bash
echo "next line" >> file.txt
```

## 10.3 Input Redirection

### `<`

Use a file as stdin.

```bash
wc -l < /etc/passwd
sort < names.txt
```

## 10.4 Error Redirection

### `2>`

Redirect stderr to a file.

```bash
find /root -name '*.conf' 2> errors.log
```

### `2>>`

Append stderr to a file.

```bash
command 2>> errors.log
```

## 10.5 Redirect Both stdout and stderr

### `2>&1`

Send stderr to the same place as stdout.

```bash
command > output.log 2>&1
```

Order matters.

This is correct:

```bash
command > output.log 2>&1
```

This means something else:

```bash
command 2>&1 > output.log
```

## 10.6 Discard Output

Use `/dev/null`.

```bash
command > /dev/null
command > /dev/null 2>&1
```

## 10.7 Pipes `|`

A pipe sends stdout of one command into stdin of another.

Examples:

```bash
ls -l | less
cat /etc/passwd | grep bash
ps aux | grep nginx
journalctl -b | tail -n 50
```

## 10.8 `tee`

`tee` writes output to both the screen and a file.

Examples:

```bash
echo "hello" | tee out.txt
ls -l | tee listing.txt
command 2>&1 | tee run.log
```

Useful for:

- logging command output
- preserving troubleshooting sessions

## 10.9 `xargs`

`xargs` builds command arguments from stdin.

Examples:

```bash
find . -name '*.log' | xargs ls -lh
find . -name '*.tmp' | xargs rm -f
printf 'a\nb\nc\n' | xargs -I{} echo file:{}
```

Safer form for unusual filenames:

```bash
find . -name '*.log' -print0 | xargs -0 ls -lh
```

## 10.10 Here Strings and Here Documents

Even though not required, they are useful to know.

### Here string

```bash
grep root <<< "root:x:0:0:root:/root:/bin/bash"
```

### Here document

```bash
cat <<EOF
line1
line2
EOF
```

## 10.11 Practical Examples

### Save command output for later review

```bash
df -h > disk-report.txt
```

### Capture only errors

```bash
find /etc -name '*.conf' 2> find-errors.txt
```

### Search logs through a pipeline

```bash
grep ERROR app.log | tail -n 20
```

### Count active TCP listeners

```bash
ss -tln | wc -l
```

### Log deployment output

```bash
./deploy.sh 2>&1 | tee deploy.log
```

## 10.12 Redirection Summary Table

| Operator | Meaning |
|---|---|
| `>` | Redirect stdout and overwrite |
| `>>` | Redirect stdout and append |
| `<` | Redirect stdin from file |
| `2>` | Redirect stderr |
| `2>>` | Append stderr |
| `2>&1` | Merge stderr into stdout |
| `|` | Pipe stdout to next command |

> Tip:
> If you are debugging a command, capture both stdout and stderr with `2>&1 | tee file.log`.

---

