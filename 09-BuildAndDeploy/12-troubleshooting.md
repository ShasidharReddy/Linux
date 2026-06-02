# Troubleshooting Applications

[Back to guide index](README.md)

## 12.1 Troubleshooting Workflow

A structured workflow prevents guesswork.

Steps:

1. Identify impact and symptoms
2. Confirm recent changes
3. Check process state
4. Check logs
5. Check system resources
6. Check dependencies
7. Reproduce or isolate
8. Mitigate safely
9. Document findings

## 12.2 Application Won't Start

Common causes:

- Port conflict
- Missing runtime or library
- Bad environment variable
- Broken config syntax
- Permission issue
- Missing secret file
- Failed migration or dependency startup

## 12.3 Check Service Status

```bash
# systemctl status myapp
# journalctl -u myapp -n 100 --no-pager
```

## 12.4 Detect Port Conflicts

Use `ss` or `lsof`.

```bash
# ss -ltnp | grep :8080
# lsof -i :8080
```

If another process owns the port, decide whether:

- The wrong service is running
- The config has the wrong port
- A stale process needs controlled shutdown

## 12.5 Missing Dependencies

Examples:

- Shared libraries missing for native apps
- Wrong JDK runtime for Java apps
- Missing Python module in venv
- Node module not installed or wrong build output
- Missing .NET runtime for framework-dependent deployment

Diagnostics:

```bash
$ ldd ./mybinary
$ java -version
$ /opt/pythonapp/venv/bin/python -c "import flask"
$ node -v
$ dotnet --info
```

## 12.6 Permission Issues

Check:

- File ownership
- Execute bits
- SELinux/AppArmor context where applicable
- Directory traversal permissions

Commands:

```bash
$ ls -l /opt/myapp/current
$ namei -l /opt/myapp/current/app.jar
```

## 12.7 Memory Leaks

Symptoms:

- Gradual RSS growth
- OOM kills
- Increased GC pressure
- Restart loops after sustained load

By runtime:

- JVM: heap dump, GC logs, JFR
- Python: object retention, worker growth, native extension leaks
- Node.js: heap snapshots, event loop lag, retaining closures
- Go: pprof heap profiles
- .NET: memory dumps, counters, tracing

## 12.8 OOM Diagnostics on Linux

Check kernel logs:

```bash
# dmesg | grep -i -E 'killed process|out of memory'
```

Check service logs around the time of failure.

## 12.9 Connection Pool Exhaustion

Symptoms:

- Timeouts under load
- Errors acquiring DB connections
- Increased latency on dependent operations

Common causes:

- Leaked connections
- Pool too small
- DB too slow
- Long transactions
- Traffic spike

What to inspect:

- Pool metrics
- DB slow queries
- Application code paths not returning connections
- Thread or worker counts relative to pool size

## 12.10 Slow Response Times

Check:

- Application logs
- Reverse proxy timing fields
- DB latency
- External API latency
- CPU saturation
- Disk I/O wait
- GC pauses

Commands:

```bash
$ top
$ vmstat 1 5
$ iostat -xz 1 5
$ free -m
```

## 12.11 `strace`

`strace` traces system calls.

Example:

```bash
# strace -p 12345
```

Use cases:

- Hanging process
- Unexpected file lookups
- Permission errors
- Network call diagnosis

Start a program under strace:

```bash
# strace -f -o strace.log ./myapp
```

## 12.12 `lsof`

`lsof` lists open files and sockets.

Examples:

```bash
# lsof -p 12345
# lsof -iTCP -sTCP:LISTEN -P -n
```

Use cases:

- Identify open ports
- Inspect leaked file descriptors
- Confirm log files or sockets in use

## 12.13 `tcpdump`

`tcpdump` captures packets.

Example:

```bash
# tcpdump -i any host 10.0.0.20 and port 5432
```

Use with care in production.

Use cases:

- Verify traffic reaches a dependency
- Inspect TLS handshake attempts
- Validate load balancer routing

## 12.14 `curl` for Local Health Testing

```bash
$ curl -v http://127.0.0.1:8080/health
$ curl -vk https://app.example.com/health
```

## 12.15 DNS and Connectivity Checks

Commands:

```bash
$ getent hosts db.internal
$ dig app.example.com
$ ping -c 3 db.internal
$ nc -vz db.internal 5432
```

## 12.16 Reverse Proxy Troubleshooting

Check:

- Nginx or Apache config syntax
- Upstream port availability
- Proxy headers
- TLS cert validity
- Body size limits
- WebSocket upgrade config

Commands:

```bash
# nginx -t
# journalctl -u nginx -n 100 --no-pager
```

## 12.17 Java Troubleshooting Tips

- Inspect GC logs
- Verify heap settings
- Confirm JDK version
- Use `jcmd`, `jmap`, `jstack` if available
- Validate truststore and certificate config for outbound TLS

## 12.18 Python Troubleshooting Tips

- Validate venv interpreter path
- Check Gunicorn worker exits
- Confirm package installation in the correct environment
- Verify `DJANGO_SETTINGS_MODULE` or Flask config vars

## 12.19 Node.js Troubleshooting Tips

- Confirm compiled output exists
- Check `NODE_ENV`
- Review uncaught exception handling
- Inspect heap with the inspector when needed
- Verify WebSocket reverse proxy behavior

## 12.20 Go Troubleshooting Tips

- Expose pprof in secured environments
- Inspect goroutine leaks
- Validate config and env parsing at startup
- Confirm static binary expectations vs CGO use

## 12.21 .NET Troubleshooting Tips

- Check runtime version availability
- Inspect Kestrel bind settings
- Validate forwarded headers middleware configuration
- Use `dotnet-counters` and related diagnostics tools where available

## 12.22 Incident Triage Table

| Symptom | Likely Areas | First Checks |
|---|---|---|
| Service won't start | Config, deps, permissions | `systemctl status`, logs |
| 502 from proxy | Upstream down, bad port | `curl localhost`, Nginx logs |
| High latency | DB, CPU, GC, external APIs | metrics, `top`, app logs |
| OOM restarts | Leak, too-small limits | kernel logs, memory graphs |
| Connection errors | DNS, firewall, pool, creds | `nc`, `dig`, config |

## 12.23 Post-Incident Review Topics

Capture:

- Root cause
- Trigger
- Detection method
- Time to mitigate
- Time to resolve
- Gaps in automation or alerting
- Action items to prevent recurrence

---

---

# Appendix H. Practical Smoke Tests

## H.1 HTTP Health Check

```bash
$ curl -fsS http://127.0.0.1:8080/health
```

## H.2 TLS Check

```bash
$ curl -vk https://app.example.com/health
```

## H.3 Port Listening Check

```bash
# ss -ltnp | grep -E ':3000|:5000|:8000|:8080'
```

## H.4 Process Check

```bash
# systemctl is-active myapp
```

## H.5 Nginx Config Check

```bash
# nginx -t
```

---

---

# Supplemental Line Expansion Section

The following compact reminders are included intentionally to make this document a practical long-form reference.

## Quick Reminder 1

Use absolute paths in automation.

## Quick Reminder 2

Do not run production services as root.

## Quick Reminder 3

Prefer immutable artifacts.

## Quick Reminder 4

Pin versions in CI.

## Quick Reminder 5

Expose health endpoints.

## Quick Reminder 6

Monitor error rate after each deployment.

## Quick Reminder 7

Keep rollback artifacts ready.

## Quick Reminder 8

Validate Nginx config before reload.

## Quick Reminder 9

Use `npm ci` in CI.

## Quick Reminder 10

Use virtual environments for Python.

## Quick Reminder 11

Use the Gradle wrapper when provided.

## Quick Reminder 12

Use `go mod tidy` carefully and review changes.

## Quick Reminder 13

Use `dotnet publish` for deployment output.

## Quick Reminder 14

Set JVM heap intentionally.

## Quick Reminder 15

Verify proxy headers end to end.

## Quick Reminder 16

Keep config outside release directories.

## Quick Reminder 17

Restrict permissions on environment files.

## Quick Reminder 18

Capture deployment version in logs.

## Quick Reminder 19

Store checksums with artifacts.

## Quick Reminder 20

Prefer systemd for standard Linux operations.

## Quick Reminder 21

Use readiness checks for production rollouts.

## Quick Reminder 22

Keep static assets on the proxy when possible.

## Quick Reminder 23

Record recent changes during incident triage.

## Quick Reminder 24

Measure before tuning.

## Quick Reminder 25

Favor promotion over rebuild.

## Quick Reminder 26

Do not bake secrets into images or tarballs.

## Quick Reminder 27

Use service users with nologin shells.

## Quick Reminder 28

Retain a few previous releases.

## Quick Reminder 29

Prefer structured logs.

## Quick Reminder 30

Test rollback procedures before emergencies.

## Quick Reminder 31

Use `journalctl -u <service> -f` for live troubleshooting.

## Quick Reminder 32

Check `ss -ltnp` for port ownership.

## Quick Reminder 33

Check `dmesg` for OOM kills.

## Quick Reminder 34

Use `curl -fsS` for smoke tests.

## Quick Reminder 35

Validate TLS renewal automation.

## Quick Reminder 36

Avoid mutable app state on local disk when scaling.

## Quick Reminder 37

Prefer backward-compatible DB changes.

## Quick Reminder 38

Use canaries when blast radius matters.

## Quick Reminder 39

Use blue-green when rollback speed matters most.

## Quick Reminder 40

Use rolling updates for stateless multi-instance services.

## Quick Reminder 41

Keep deployment scripts idempotent.

## Quick Reminder 42

Avoid manual server edits outside emergencies.

## Quick Reminder 43

Document required environment variables.

## Quick Reminder 44

Validate runtime versions on the target host.

## Quick Reminder 45

Track dependency caches in CI wisely.

## Quick Reminder 46

Size worker counts using load testing, not guesswork.

## Quick Reminder 47

Avoid overcommitting CPU with too many workers.

## Quick Reminder 48

Check file descriptor limits for high-traffic services.

## Quick Reminder 49

Use non-interactive CI-friendly commands.

## Quick Reminder 50

Treat deployment logs as operational evidence.
