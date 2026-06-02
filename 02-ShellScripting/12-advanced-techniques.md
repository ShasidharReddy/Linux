# Advanced Techniques

> getopts, signals, temporary files, locking, debugging, and coprocesses.

## 13. Advanced Techniques

### 13.1 Argument Parsing with `getopts`

Use `getopts` for short options.

```bash
#!/usr/bin/env bash
set -euo pipefail

verbose=false
output=""

while getopts ":vo:" opt; do
  case "$opt" in
    v) verbose=true ;;
    o) output=$OPTARG ;;
    :) echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
    \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
  esac
done
shift $((OPTIND - 1))
```

### 13.2 Long Options Pattern

`getopts` does not support long options directly.

A common manual pattern:

```bash
while [[ $# -gt 0 ]]; do
  case "$1" in
    --help)
      echo "Help"
      exit 0
      ;;
    --env)
      env_name=$2
      shift 2
      ;;
    *)
      echo "Unknown option: $1" >&2
      exit 1
      ;;
  esac
done
```

### 13.3 Signal Handling

```bash
handle_int() {
  echo "Caught SIGINT"
  exit 130
}

trap handle_int INT
```

### 13.4 Temporary Files with `mktemp`

```bash
temp_file=$(mktemp)
trap 'rm -f -- "$temp_file"' EXIT
```

For temporary directories:

```bash
temp_dir=$(mktemp -d)
trap 'rm -rf -- "$temp_dir"' EXIT
```

### 13.5 Lock Files with `flock`

Prevent concurrent runs.

```bash
exec 9>/var/lock/my_script.lock
flock -n 9 || {
  echo "Another instance is running" >&2
  exit 1
}
```

### 13.6 Debugging with `set -x`

```bash
set -x
```

Disable later:

```bash
set +x
```

### 13.7 Customize Debug Output with `PS4`

```bash
export PS4='+ ${BASH_SOURCE}:${LINENO}:${FUNCNAME[0]}: '
set -x
```

### 13.8 Logging to File and Console

```bash
log_file="app.log"
log() {
  printf '%s %s\n' "$(date '+%F %T')" "$*" | tee -a "$log_file"
}
```

### 13.9 Structured Logging Levels

```bash
log_msg() {
  local level=$1
  shift
  printf '%s [%s] %s\n' "$(date '+%F %T')" "$level" "$*"
}
```

### 13.10 Command-Line Usage Function

```bash
usage() {
  cat <<'EOF'
Usage: script.sh [-v] [-o file]
  -v        Verbose mode
  -o FILE   Output file
EOF
}
```

### 13.11 Config File Loading

Example `.env`-style loading:

```bash
set -a
source .env
set +a
```

Only do this for trusted files.

### 13.12 Safe Directory Changes

```bash
script_dir=$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)
cd "$script_dir"
```

### 13.13 Detect OS

```bash
case "$(uname -s)" in
  Linux) echo "Linux" ;;
  Darwin) echo "macOS" ;;
  *) echo "Other" ;;
esac
```

### 13.14 Timeout Pattern

```bash
if ! timeout 5s curl -fsS https://example.com; then
  echo "Timed out or failed" >&2
fi
```

### 13.15 Reading Script Directory Reliably

```bash
SCRIPT_DIR=$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)
```

### 13.16 Advanced Redirection with `exec`

Redirect all output:

```bash
exec >script.out 2>script.err
```

Or both together:

```bash
exec > >(tee -a script.log) 2>&1
```

### 13.17 Coprocesses

Bash supports `coproc` for advanced two-way communication.

```bash
coproc MYCOPROC { tr '[:lower:]' '[:upper:]'; }
printf 'hello\n' >&"${MYCOPROC[1]}"
read -r line <&"${MYCOPROC[0]}"
echo "$line"
```

### 13.18 Null-Delimited Safety

Use null delimiters for filenames.

```bash
find . -type f -print0 |
while IFS= read -r -d '' file; do
  printf '%s\n' "$file"
done
```

### 13.19 Rate Limiting Loops

```bash
for item in "${items[@]}"; do
  process "$item"
  sleep 0.2
done
```

### 13.20 Retry Helper with Backoff

```bash
retry_backoff() {
  local max_attempts=$1
  shift
  local delay=1
  local attempt=1

  until "$@"; do
    if (( attempt >= max_attempts )); then
      return 1
    fi
    sleep "$delay"
    delay=$((delay * 2))
    ((attempt++))
  done
}
```

### 13.21 Section Summary

Advanced techniques improve:

- robustness
- observability
- safety
- concurrency control
- maintainability

---
