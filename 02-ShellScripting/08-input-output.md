# Input and Output

> Reading input, formatting output, redirection, and here documents.

## 9. Input/Output

### 9.1 `echo`

```bash
echo "Hello"
```

Be aware that `echo` behavior can vary between shells.

### 9.2 `printf`

Prefer `printf` for predictable formatting.

```bash
printf 'Name: %s\n' "$name"
```

### 9.3 Common `printf` Specifiers

| Specifier | Meaning |
| --- | --- |
| `%s` | String |
| `%d` | Integer |
| `%f` | Floating point |
| `%q` | Shell-escaped string in Bash |

### 9.4 Reading User Input with `read`

```bash
read -r name
printf 'Hello, %s\n' "$name"
```

### 9.5 Prompting Inline

```bash
read -r -p "Enter your name: " name
```

### 9.6 Reading Silent Input

```bash
read -r -s -p "Password: " password
echo
```

### 9.7 Reading Multiple Fields

```bash
read -r first last <<< "Ada Lovelace"
echo "$first"
echo "$last"
```

### 9.8 Here Documents

A here document feeds multiple lines to a command.

```bash
cat <<EOF
Line 1
Line 2
EOF
```

### 9.9 Quoted Here Documents

Quoted delimiter disables expansion.

```bash
cat <<'EOF'
Literal $HOME
Literal $(date)
EOF
```

### 9.10 Here Strings

A here string passes a single string.

```bash
grep "root" <<< "root:x:0:0"
```

### 9.11 Redirecting Output

| Syntax | Meaning |
| --- | --- |
| `>` | Redirect stdout |
| `>>` | Append stdout |
| `2>` | Redirect stderr |
| `2>>` | Append stderr |
| `&>` | Redirect stdout and stderr in Bash |
| `>/dev/null` | Discard stdout |

### 9.12 Standard Streams

| FD | Name |
| --- | --- |
| `0` | stdin |
| `1` | stdout |
| `2` | stderr |

### 9.13 Redirect stderr Separately

```bash
command >output.log 2>error.log
```

### 9.14 Redirect Both Streams

Portable style:

```bash
command >all.log 2>&1
```

### 9.15 Read from File Descriptor

```bash
exec 3< input.txt
read -r line <&3
echo "$line"
exec 3<&-
```

### 9.16 Write to Custom File Descriptor

```bash
exec 3> output.txt
printf 'hello\n' >&3
exec 3>&-
```

### 9.17 Piping Commands

```bash
ps aux | grep nginx
```

### 9.18 Safer Pipeline with `pipefail`

```bash
set -o pipefail
```

### 9.19 Capture stdout in Variable

```bash
hostname=$(hostname)
```

### 9.20 Capture Both stdout and stderr

```bash
output=$(command 2>&1)
```

### 9.21 Reading File into Variables

```bash
while IFS='=' read -r key value; do
  printf '%s -> %s\n' "$key" "$value"
done < config.env
```

### 9.22 Colored Output

```bash
red='\033[31m'
green='\033[32m'
reset='\033[0m'
printf '%bSuccess%b\n' "$green" "$reset"
```

### 9.23 Section Summary

For production scripts:

- prefer `printf`
- use `read -r`
- understand redirection
- use quoted here documents when needed

---
