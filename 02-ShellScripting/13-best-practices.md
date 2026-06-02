# Best Practices

> Best practices plus quick references and extended practice material from the original guide.

## 14. Best Practices

### 14.1 Use ShellCheck

ShellCheck is a static analysis tool for shell scripts.

```bash
shellcheck script.sh
```

Benefits:

- catches quoting issues
- finds unused variables
- flags portability problems
- suggests safer patterns

### 14.2 Quote Variables

Bad:

```bash
cp $src $dst
```

Good:

```bash
cp -- "$src" "$dst"
```

### 14.3 Use `set -euo pipefail` When Appropriate

This improves safety, but understand the behavior before applying it everywhere.

### 14.4 Prefer `printf` over `echo`

`printf` is more predictable across environments.

### 14.5 Avoid Parsing `ls`

Bad:

```bash
for f in $(ls *.txt); do
  echo "$f"
done
```

Better:

```bash
for f in ./*.txt; do
  [[ -e $f ]] || continue
  echo "$f"
done
```

### 14.6 Use `[[ ]]` in Bash Scripts

It is safer and more expressive than `[ ]` for many Bash-specific scripts.

### 14.7 Prefer Functions Over Repeated Blocks

Refactor repeated logic into reusable functions.

### 14.8 Separate stdout and stderr

Normal results should go to stdout.

Errors and diagnostics should go to stderr.

### 14.9 Validate Inputs Early

Examples:

- required arguments
- file existence
- numeric ranges
- command availability
- permissions

### 14.10 Check Dependencies

```bash
require_cmd() {
  command -v "$1" >/dev/null 2>&1 || {
    printf 'Missing required command: %s\n' "$1" >&2
    exit 1
  }
}
```

### 14.11 Use Portable Syntax When Needed

If targeting `/bin/sh`, avoid Bash-only features such as:

- arrays
- `[[ ]]`
- associative arrays
- process substitution
- `mapfile`

### 14.12 Security: Avoid `eval`

Bad:

```bash
eval "$user_input"
```

This can lead to command injection.

### 14.13 Security: Quote User Input

```bash
grep -- "$pattern" "$file"
```

### 14.14 Security: Use `--` for End of Options

```bash
rm -- "$file"
```

Helps when filenames begin with `-`.

### 14.15 Security: Restrict `IFS` Changes

If you change `IFS`, do it locally and carefully.

### 14.16 Security: Avoid World-Writable Temporary Paths

Use secure temp creation patterns.

### 14.17 Use Meaningful Exit Codes

- `0` for success
- `1` for general failure
- custom non-zero codes for domain-specific problems

### 14.18 Document Usage

Every production script should have:

- help text
- argument summary
- examples
- exit code notes when useful

### 14.19 Keep Scripts Small or Modular

Split large codebases into library files.

### 14.20 Test with Different Inputs

Test:

- empty input
- spaces in filenames
- missing files
- invalid flags
- large input sets
- concurrent runs

### 14.21 Portability Checklist

| Check | Why |
| --- | --- |
| Shebang matches syntax used | Prevent runtime failures |
| Avoid Bash-only features in `sh` scripts | Improve compatibility |
| Use `command -v` | Portable command detection |
| Prefer POSIX options when possible | Broader support |

### 14.22 Readability Guidelines

- indent consistently
- keep functions short
- name variables clearly
- comment intent, not obvious syntax
- group related logic

### 14.23 Security Checklist

- validate inputs
- avoid `eval`
- quote expansions
- use absolute paths for sensitive operations
- limit privileges
- use locks for critical sections
- avoid sourcing untrusted files

### 14.24 Section Summary

Best practices make scripts safer, clearer, and easier to maintain.

---

## 16. Appendix

### 16.1 Quick Reference: Safe Script Template

```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
  printf '%s [INFO] %s\n' "$(date '+%F %T')" "$*"
}

die() {
  printf '%s [ERROR] %s\n' "$(date '+%F %T')" "$*" >&2
  exit 1
}

cleanup() {
  :
}
trap cleanup EXIT

usage() {
  cat <<'EOF'
Usage: script.sh [options]
EOF
}

main() {
  log "Starting"
}

main "$@"
```

### 16.2 Quick Reference: Special Variables

| Variable | Meaning |
| --- | --- |
| `$0` | Script name |
| `$1`...`$9` | Positional parameters |
| `$#` | Number of arguments |
| `$@` | All arguments separately |
| `$*` | All arguments as one string when quoted |
| `$?` | Last exit status |
| `$$` | Current PID |
| `$!` | Last background PID |

### 16.3 Quick Reference: File Tests

| Test | Meaning |
| --- | --- |
| `-e` | Exists |
| `-f` | Regular file |
| `-d` | Directory |
| `-r` | Readable |
| `-w` | Writable |
| `-x` | Executable |
| `-s` | Non-empty |
| `-L` | Symbolic link |

### 16.4 Quick Reference: String Tests

| Test | Meaning |
| --- | --- |
| `-z "$x"` | Empty |
| `-n "$x"` | Not empty |
| `"$a" = "$b"` | Equal |
| `"$a" != "$b"` | Not equal |

### 16.5 Quick Reference: Numeric Tests

| Test | Meaning |
| --- | --- |
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater or equal |
| `-le` | Less or equal |

### 16.6 Quick Reference: Redirections

| Syntax | Meaning |
| --- | --- |
| `>` | Write stdout |
| `>>` | Append stdout |
| `2>` | Write stderr |
| `2>>` | Append stderr |
| `2>&1` | Redirect stderr to stdout |
| `<` | Read stdin |
| `<<EOF` | Here document |
| `<<<` | Here string |

### 16.7 Quick Reference: Useful Builtins

| Builtin | Purpose |
| --- | --- |
| `cd` | Change directory |
| `echo` | Print text |
| `printf` | Formatted output |
| `read` | Read input |
| `test` / `[` / `[[` | Conditions |
| `trap` | Signal handling |
| `exec` | Replace shell or manage file descriptors |
| `set` | Shell options |
| `shift` | Shift positional parameters |
| `source` / `.` | Load another file |

### 16.8 Troubleshooting Tips

- Run `bash -n script.sh` for syntax checking.
- Run `bash -x script.sh` for execution tracing.
- Use `shellcheck` for linting.
- Add `printf` statements to inspect values.
- Reproduce bugs with minimal inputs.

### 16.9 Common Anti-Patterns

- using `for x in $(command)` for arbitrary data
- leaving variables unquoted
- mixing `sh` shebang with Bash features
- ignoring return codes
- using `eval` on untrusted input
- assuming GNU tool behavior on all systems

### 16.10 Final Notes

Shell scripting is one of the most practical automation skills.

Start simple.

Write clear scripts.

Quote carefully.

Handle errors explicitly.

Prefer maintainable patterns.

Use shell where it fits best.

When complexity grows too large, consider pairing shell with languages like Python, Go, or Ruby for heavy parsing or application logic.

---

## 17. Extended Practice Notes

This final section intentionally provides additional compact practice material, reminders, and examples so the guide can serve as a long-form reference and workbook.

### 17.1 Practice Checklist

- Write a script that greets a user.
- Add argument parsing.
- Validate input.
- Log actions.
- Add error handling.
- Add cleanup with `trap`.
- Run ShellCheck.

### 17.2 Mini Exercise: Positional Parameters

```bash
#!/usr/bin/env bash
printf 'Script: %s\n' "$0"
printf 'Arg count: %s\n' "$#"
printf 'All args: %s\n' "$*"
```

### 17.3 Mini Exercise: File Check

```bash
#!/usr/bin/env bash
file=${1:-}
if [[ -z $file ]]; then
  echo "Provide a file" >&2
  exit 1
fi
[[ -f $file ]] && echo "Regular file"
```

### 17.4 Mini Exercise: Count Lines

```bash
#!/usr/bin/env bash
count=0
while IFS= read -r _; do
  ((count++))
done < "$1"
echo "$count"
```

### 17.5 Mini Exercise: Function Reuse

```bash
#!/usr/bin/env bash
log() {
  printf '[LOG] %s\n' "$*"
}
log "hello"
```

### 17.6 Mini Exercise: Regex Validation

```bash
#!/usr/bin/env bash
value=${1:-}
[[ $value =~ ^[A-Za-z_][A-Za-z0-9_]*$ ]] && echo ok || echo bad
```

### 17.7 Mini Exercise: Array Iteration

```bash
#!/usr/bin/env bash
items=(alpha beta gamma)
for item in "${items[@]}"; do
  echo "$item"
done
```

### 17.8 Mini Exercise: Menu with `select`

```bash
#!/usr/bin/env bash
select env in dev test prod quit; do
  echo "$env"
  [[ $env == quit ]] && break
done
```

### 17.9 Mini Exercise: `getopts`

```bash
#!/usr/bin/env bash
while getopts ":f:" opt; do
  case "$opt" in
    f) echo "file=$OPTARG" ;;
  esac
done
```

### 17.10 Mini Exercise: Error Trap

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
trap 'echo "failed at line $LINENO"' ERR
false
```

### 17.11 Interview-Style Questions

1. What is the difference between `"$@"` and `"$*"`?
2. Why is `read -r` preferred?
3. When should you use `[[ ]]` instead of `[ ]`?
4. What does `set -o pipefail` solve?
5. Why is `eval` risky?
6. What does `trap cleanup EXIT` do?
7. Why avoid `for f in $(ls)`?
8. What is process substitution?
9. How do you parse options in Bash?
10. Why use `printf` over `echo`?

### 17.12 Study Table: Choose the Right Tool

| Problem | Shell Feature | Example |
| --- | --- | --- |
| One-time value store | Variable | `name=value` |
| Repeated action | Loop | `for` / `while` |
| Branching logic | Conditional | `if` / `case` |
| Reusable logic | Function | `log_info()` |
| Text validation | Regex | `[[ x =~ re ]]` |
| Multiple related values | Array | `items=(a b)` |
| Cleanup on exit | Trap | `trap cleanup EXIT` |
| Background execution | `&` and `wait` | `cmd & wait` |

### 17.13 Study Table: `[` vs `[[` vs `(( ))`

| Form | Best For | Notes |
| --- | --- | --- |
| `[ ]` | Portable conditions | POSIX-friendly |
| `[[ ]]` | Bash string/file/regex tests | Safer syntax |
| `(( ))` | Arithmetic evaluation | Numeric expressions |

### 17.14 Study Table: Common Expansion Forms

| Syntax | Purpose |
| --- | --- |
| `${var}` | Variable expansion |
| `${#var}` | Length |
| `${var:-default}` | Default if unset or null |
| `${var:=default}` | Assign default |
| `${var:?message}` | Error if missing |
| `${var:+alt}` | Alternate if set |
| `${var/pat/repl}` | Replace first match |
| `${var//pat/repl}` | Replace all matches |

### 17.15 Practice Notes on Quoting

Always ask:

- Could this contain spaces?
- Could this contain glob characters?
- Could this be empty?
- Could this begin with a dash?

If yes, quote it and often add `--` before path arguments.

### 17.16 Practice Notes on Portability

If a script starts with `#!/bin/sh`, verify every feature is POSIX.

If you need arrays, `[[ ]]`, or associative maps, switch to Bash explicitly.

### 17.17 Practice Notes on Maintainability

Large shell scripts become hard to test.

When scripts exceed comfortable complexity:

- extract functions
- move helpers into sourced files
- reduce global state
- document assumptions
- consider another language for complex data parsing

### 17.18 Practice Notes on Security

Security habits for shell authors:

- distrust input
- quote parameters
- use `command -v` to validate tools
- avoid `eval`
- prefer exact paths in privileged scripts
- do not source untrusted files
- use least privilege

### 17.19 Practice Notes on Debugging

When debugging a script:

1. Run `bash -n script.sh`
2. Run `shellcheck script.sh`
3. Run `bash -x script.sh`
4. Add `printf '%q\n' "$var"` to inspect tricky values
5. Reproduce with minimal inputs

### 17.20 Practice Notes on Logging

A production script should answer:

- What started?
- When did it start?
- What input was used?
- What succeeded?
- What failed?
- What exit code was returned?

### 17.21 Practice Notes on Exit Codes

Reserve different codes when useful.

Example:

- `1` general failure
- `2` bad arguments
- `3` missing dependency
- `4` permission denied
- `5` remote service unavailable

### 17.22 Practice Notes on Common Commands Used with Shell

Common helper tools:

- `grep`
- `sed`
- `awk`
- `cut`
- `sort`
- `uniq`
- `find`
- `xargs`
- `tar`
- `gzip`
- `curl`
- `jq`

### 17.23 Practice Notes on When Not to Use Shell

Shell is excellent for orchestration.

It is less ideal for:

- complex data structures
- large-scale JSON transformations without `jq`
- advanced CSV parsing
- long-running application logic
- multi-threaded computation

### 17.24 Practice Notes on CI/CD Usage

Shell scripts are common in:

- build pipelines
- deployment steps
- container entrypoints
- release packaging
- environment validation

In CI, always assume a clean environment and validate dependencies explicitly.

### 17.25 Practice Notes on Idempotency

An idempotent script can be run repeatedly without causing unintended side effects.

Examples:

- `mkdir -p`
- checking before creating users or directories
- using `ln -sfn` for symlink updates
- using declarative package managers when possible

### 17.26 Practice Notes on Defensive Patterns

Useful patterns:

```bash
: "${REQUIRED_VAR:?must be set}"
command -v jq >/dev/null 2>&1 || exit 1
[[ -d $dir ]] || mkdir -p "$dir"
```

### 17.27 Practice Notes on Safer Reads

```bash
while IFS= read -r line; do
  printf '%s\n' "$line"
done < input.txt
```

This avoids losing backslashes and preserves spacing better than a plain `read` loop.

### 17.28 Practice Notes on Safer File Discovery

```bash
find . -type f -name '*.txt' -print0 |
while IFS= read -r -d '' file; do
  printf '%s\n' "$file"
done
```

Use this pattern when filenames can contain spaces, tabs, or newlines.

### 17.29 Practice Notes on Shell Options

Common options:

- `set -e`
- `set -u`
- `set -x`
- `set -o pipefail`
- `set -f`

`set -f` disables globbing and can be useful in rare defensive cases.

### 17.30 Practice Notes on Testing Scripts

Testing ideas:

- run with no arguments
- run with valid arguments
- run with invalid arguments
- simulate missing commands
- simulate read-only destination
- simulate network failure

### 17.31 Practice Notes on Documentation

Document these clearly:

- purpose
- required tools
- arguments
- environment variables
- outputs
- exit codes
- examples

### 17.32 Practice Notes on Sourcing vs Executing

Executing:

```bash
./script.sh
```

Sourcing:

```bash
source script.sh
```

Sourcing runs commands in the current shell.

That means variables and directory changes affect the current session.

### 17.33 Practice Notes on `main`

A clean script layout often looks like:

1. strict mode
2. constants
3. helper functions
4. `usage`
5. `parse_args`
6. `main`
7. `main "$@"`

### 17.34 Practice Notes on Large Code Blocks

Prefer short focused functions over one huge block.

This makes scripts easier to:

- read
- debug
- test
- reuse

### 17.35 Practice Notes on External Commands

Every external command has cost.

Optimize by:

- using shell builtins when practical
- avoiding unnecessary subshells
- reducing repeated parsing
- batching operations

### 17.36 Practice Notes on Performance

Shell startup is cheap, but repeated external command calls can be slow.

For heavy loops over large data:

- prefer one `awk` over thousands of `grep` calls
- prefer one `find` pipeline over nested command substitutions
- measure before optimizing

### 17.37 Practice Notes on Readonly Constants

```bash
readonly APP_NAME="backup-tool"
readonly DEFAULT_PORT=8080
```

Use constants for values that should never change.

### 17.38 Practice Notes on Associative Arrays

Associative arrays are excellent for:

- lookup tables
- config maps
- command dispatch
- counters by key

Remember they are Bash-specific.

### 17.39 Practice Notes on `case` Dispatch

`case` is often cleaner than many `if/elif` blocks for command dispatch.

```bash
case "$action" in
  start) start_service ;;
  stop) stop_service ;;
  status) show_status ;;
  *) usage; exit 1 ;;
esac
```

### 17.40 Practice Notes on Input Trust Levels

Consider whether input comes from:

- a trusted operator
- a config file
- a CI variable
- an API response
- user-entered text
- filenames on disk

Less trust means more validation.

### 17.41 Practice Notes on Log Rotation Example Extensions

Possible enhancements:

- keep only N archives
- upload archives to remote storage
- skip active files currently in use
- compress with configurable level
- write a summary report

### 17.42 Practice Notes on Backup Example Extensions

Possible enhancements:

- incremental backups
- remote copy with `rsync`
- checksum manifest
- restore command
- exclusion file
- encryption

### 17.43 Practice Notes on Monitoring Example Extensions

Possible enhancements:

- email or Slack alerting
- multi-metric checks
- repeated polling
- JSON output
- status page integration

### 17.44 Practice Notes on Deployment Example Extensions

Possible enhancements:

- blue/green deployment
- health check rollback
- artifact version tagging
- maintenance window locks
- notification hooks

### 17.45 Practice Notes on CSV Parsing Limitations

CSV gets tricky when fields contain:

- commas
- quotes
- embedded newlines
- escaped characters

Use a dedicated CSV parser for full correctness.

### 17.46 Practice Notes on Regex Limits

Keep regex maintainable.

When patterns become hard to read:

- split validation into smaller checks
- add comments
- use helper functions
- test against examples

### 17.47 Practice Notes on Linters and Formatters

Useful tools around shell scripting:

- `shellcheck`
- `shfmt`
- `bats`

`shfmt` formats shell scripts consistently.

`bats` can be used for tests.

### 17.48 Practice Notes on Common Builtin Commands

More useful builtins:

- `type`
- `alias`
- `unalias`
- `export`
- `readonly`
- `umask`
- `times`
- `wait`

### 17.49 Practice Notes on Exit Trap Guarantees

`trap cleanup EXIT` is powerful, but be aware:

- abrupt termination like `kill -9` skips traps
- some failures in child processes may not trigger parent cleanup as expected
- test your cleanup flow under interruption scenarios

### 17.50 Closing Summary

You now have a broad reference spanning:

- fundamentals
- variables
- strings
- arrays
- operators
- conditions
- loops
- functions
- I/O
- error handling
- regex
- processes
- advanced patterns
- best practices
- real-world scripts

Use this guide as both a learning path and a production reference.
