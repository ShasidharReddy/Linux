# Variables

> Variable declaration, scope, environment variables, and special parameters.

## 2. Variables

### 2.1 Variable Basics

Variables store values.

Shell variables are usually untyped strings.

Example:

```bash
name="Alice"
age=30
city="New York"
```

Access them with `$`:

```bash
echo "$name"
echo "$age"
```

### 2.2 Assignment Rules

- No spaces around `=`
- Variable names usually contain letters, digits, and underscores
- Names should not start with a digit

Valid:

```bash
user_name="john"
count=5
```

Invalid:

```bash
2name="bad"
my var="bad"
```

### 2.3 Quoted vs Unquoted Assignment

```bash
message="hello world"
path=/var/log
```

Quotes are recommended when the value may contain spaces or special characters.

### 2.4 Local Variables

Inside functions, use `local` in Bash:

```bash
my_func() {
  local temp="inside"
  echo "$temp"
}
```

`local` helps avoid overwriting global variables.

### 2.5 Global Variables

Variables defined outside functions are usually global to the script.

```bash
app_name="demo"

show_name() {
  echo "$app_name"
}
```

### 2.6 Environment Variables

Environment variables are exported so child processes can use them.

```bash
export APP_ENV="production"
export PATH="$PATH:/custom/bin"
```

Difference:

| Type | Visible in current shell | Visible in child processes |
| --- | --- | --- |
| Regular variable | Yes | No |
| Exported variable | Yes | Yes |

### 2.7 Exporting Variables

```bash
name="Alice"
export name
```

Or in one line:

```bash
export name="Alice"
```

### 2.8 Readonly Variables

Use `readonly` to prevent reassignment.

```bash
readonly VERSION="1.0.0"
```

Attempting to change it causes an error.

### 2.9 Unsetting Variables

Remove a variable with `unset`:

```bash
foo="bar"
unset foo
```

### 2.10 Variable Scope and Subshells

A subshell gets a copy of variables.

```bash
name="parent"
(
  name="child"
  echo "$name"
)
echo "$name"
```

Output:

```text
child
parent
```

### 2.11 Command Substitution

Store command output in a variable:

```bash
current_date=$(date +%F)
```

Older syntax:

```bash
current_date=`date +%F`
```

Prefer `$(...)` because it is easier to read and nest.

### 2.12 Arithmetic Assignment

```bash
count=5
count=$((count + 1))
```

### 2.13 Using Braces Around Variables

Braces clarify variable boundaries.

```bash
file="report"
echo "${file}.txt"
```

### 2.14 Default Values with Parameter Expansion

Use a default if unset or null:

```bash
echo "${name:-Guest}"
```

Assign default if unset:

```bash
name=${name:-Guest}
```

Or:

```bash
: "${name:=Guest}"
```

### 2.15 Required Variables

Stop if a variable is missing:

```bash
: "${API_KEY:?API_KEY is required}"
```

### 2.16 Alternate Value if Set

```bash
echo "${name:+value exists}"
```

### 2.17 String Length via Variable Expansion

```bash
name="Shell"
echo "${#name}"
```

### 2.18 Positional Parameters

Scripts receive arguments as positional parameters.

| Variable | Meaning |
| --- | --- |
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$#` | Number of arguments |
| `$@` | All arguments as separate words |
| `$*` | All arguments as one word when quoted |
| `$?` | Exit status of last command |
| `$$` | Current process ID |
| `$!` | PID of last background process |

Example:

```bash
#!/usr/bin/env bash

echo "Script: $0"
echo "First arg: $1"
echo "Arg count: $#"
```

### 2.19 Difference Between `$@` and `$*`

Quoted `$@` preserves individual arguments.

```bash
for arg in "$@"; do
  printf '[%s]\n' "$arg"
done
```

Quoted `$*` joins all arguments into one string.

```bash
printf '%s\n' "$*"
```

### 2.20 Exit Status Variable `$?`

```bash
grep "root" /etc/passwd
status=$?
echo "$status"
```

Convention:

- `0` means success
- non-zero means failure

### 2.21 Process ID Variables

```bash
echo "Current PID: $$"
sleep 10 &
echo "Background PID: $!"
```

### 2.22 Indirect Expansion

```bash
var_name="HOME"
echo "${!var_name}"
```

Bash-specific and useful in advanced scripts.

### 2.23 Listing Environment Variables

```bash
env
printenv
```

### 2.24 Common Environment Variables

| Variable | Meaning |
| --- | --- |
| `HOME` | User home directory |
| `PATH` | Command search path |
| `USER` | Current user |
| `PWD` | Current directory |
| `SHELL` | Login shell |
| `LANG` | Locale |
| `EDITOR` | Preferred editor |

### 2.25 Best Naming Practices

- Use lowercase for local script variables
- Use uppercase for exported or constant-like variables
- Choose descriptive names
- Avoid overriding special names like `PATH` accidentally

### 2.26 Example Variable-Heavy Script

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly SCRIPT_NAME=$(basename "$0")
readonly DEFAULT_ENV="dev"
env_name="${1:-$DEFAULT_ENV}"
export APP_ENV="$env_name"

echo "Script: $SCRIPT_NAME"
echo "Environment: $APP_ENV"
echo "PID: $$"
```

### 2.27 Variable Pitfalls

Unquoted variable:

```bash
file="my file.txt"
rm $file
```

Safer:

```bash
rm -- "$file"
```

### 2.28 Section Summary

You learned:

- Variable declaration
- Local and global scope
- Exporting variables
- Readonly and unset
- Special variables and positional parameters

---
