# Functions

> Function declaration, arguments, return codes, and reusable libraries.

## 7. Functions
### 7.1 Why Functions Matter
Functions let you:

- Reuse code
- Organize logic
- Improve readability
- Reduce duplication
- Build script libraries

### 7.2 Function Declaration Styles
Portable style:

```bash
say_hello() {
  echo "Hello"
}
```

Bash also accepts:

```bash
function say_hello {
  echo "Hello"
}
```

Prefer the first style for portability.

### 7.3 Calling a Function
```bash
say_hello
```

### 7.4 Function Arguments
Functions use positional parameters too.

```bash
greet() {
  echo "Hello, $1"
}

greet "Alice"
```

### 7.5 Multiple Arguments
```bash
sum_two() {
  local a=$1
  local b=$2
  echo $((a + b))
}
```

### 7.6 Returning Values
Shell functions return an exit status, not arbitrary data.

```bash
is_even() {
  local n=$1
  (( n % 2 == 0 ))
}

if is_even 4; then
  echo "Even"
fi
```

To return data, print it and capture output.

```bash
get_date() {
  date +%F
}

today=$(get_date)
```

### 7.7 Local Variables in Functions
```bash
build_path() {
  local base=$1
  local name=$2
  echo "$base/$name"
}
```

### 7.8 Exit Status from Functions
```bash
check_file() {
  [[ -f $1 ]]
}

check_file /etc/passwd
status=$?
```

### 7.9 Guard Functions
```bash
require_file() {
  local file=$1
  [[ -f $file ]] || {
    echo "Missing file: $file" >&2
    return 1
  }
}
```

### 7.10 Logging Functions
```bash
log_info() {
  printf '[INFO] %s\n' "$*"
}

log_error() {
  printf '[ERROR] %s\n' "$*" >&2
}
```

### 7.11 Error Handling Function Pattern
```bash
die() {
  printf 'ERROR: %s\n' "$*" >&2
  exit 1
}
```

### 7.12 Function Libraries with `source`
Create a reusable library.

`lib.sh`:

```bash
log() {
  printf '[LOG] %s\n' "$*"
}
```

`main.sh`:

```bash
#!/usr/bin/env bash
source ./lib.sh
log "Started"
```

Portable alternative:

```sh
. ./lib.sh
```

### 7.13 Recursion
Bash supports recursion, but use it carefully.

```bash
factorial() {
  local n=$1
  if (( n <= 1 )); then
    echo 1
  else
    local prev
    prev=$(factorial $((n - 1)))
    echo $((n * prev))
  fi
}
```

### 7.14 Function Naming Tips
- Use lowercase with underscores
- Prefix library functions when needed
- Keep names descriptive

Examples:

- `parse_args`
- `load_config`
- `backup_directory`
- `log_info`

### 7.15 Using `return`
`return` sets a numeric exit status between 0 and 255.

```bash
check_port() {
  local port=$1
  (( port > 0 && port < 65536 ))
  return $?
}
```

### 7.16 Passing Arrays to Functions
Usually pass values or use namerefs in modern Bash.

Simple approach:

```bash
print_items() {
  local item
  for item in "$@"; do
    echo "$item"
  done
}

print_items "${fruits[@]}"
```

### 7.17 Using Namerefs
Bash 4.3+ supports `local -n`.

```bash
print_array() {
  local -n arr_ref=$1
  local item
  for item in "${arr_ref[@]}"; do
    echo "$item"
  done
}
```

### 7.18 Section Summary
Functions are the foundation of maintainable shell scripts.

---
