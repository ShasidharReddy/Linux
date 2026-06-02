# Process Management

> Background jobs, wait, job control, subshells, and process substitution.

## 12. Process Management in Scripts

### 12.1 Background Processes

Run a command in the background with `&`.

```bash
sleep 30 &
pid=$!
echo "PID: $pid"
```

### 12.2 `wait`

Wait for a background job.

```bash
sleep 2 &
pid=$!
wait "$pid"
echo "Done"
```

### 12.3 Capture Exit Status from `wait`

```bash
some_command &
pid=$!
if wait "$pid"; then
  echo "Success"
else
  echo "Failed"
fi
```

### 12.4 Multiple Background Jobs

```bash
pids=()
for host in host1 host2 host3; do
  ping -c 1 "$host" >/dev/null 2>&1 &
  pids+=("$!")
done

for pid in "${pids[@]}"; do
  wait "$pid"
done
```

### 12.5 `jobs`

Lists current jobs in an interactive shell.

```bash
jobs
```

Note that job control is mostly relevant for interactive shells.

### 12.6 `fg` and `bg`

These are job-control builtins used interactively.

- `fg` moves a job to foreground
- `bg` resumes in background

### 12.7 Subshells

Commands in parentheses run in a subshell.

```bash
(
  cd /var/log || exit 1
  ls
)
```

Changes do not affect the parent shell.

### 12.8 Group Commands Without Subshell

Use braces:

```bash
{
  echo "one"
  echo "two"
} > output.txt
```

### 12.9 Process Substitution

Useful for comparing command output.

```bash
diff <(sort file1.txt) <(sort file2.txt)
```

### 12.10 Named Pipes vs Process Substitution

Process substitution often uses FIFOs or `/dev/fd` internally depending on the platform.

It is convenient for commands that expect filenames.

### 12.11 Disowning Jobs

Interactive shell feature:

```bash
long_running_command &
disown
```

### 12.12 Monitoring Child Processes

```bash
run_worker() {
  sleep 3
}

run_worker &
worker_pid=$!

if wait "$worker_pid"; then
  echo "Worker finished"
fi
```

### 12.13 Parallel Execution Pattern

```bash
pids=()
for file in *.log; do
  [[ -e $file ]] || continue
  gzip "$file" &
  pids+=("$!")
done

for pid in "${pids[@]}"; do
  wait "$pid"
done
```

### 12.14 Beware of Race Conditions

When multiple processes write to the same file or resource, you may get corruption or unexpected order.

Use locks when necessary.

### 12.15 Section Summary

Process management enables:

- concurrency
- waiting for work
- subshell isolation
- feeding commands with process substitution

---
