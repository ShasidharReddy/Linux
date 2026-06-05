# Process Management and Configuration

[Back to guide index](README.md)

## 8.1 Why Process Management Matters

Production processes need:

- Automatic restarts
- Dependency ordering
- Startup on boot
- Log collection
- Resource controls
- Environment injection
- Health and readiness integration

## 8.2 systemd Overview

systemd is the standard service manager on most Linux distributions.

Capabilities:

- Unit-based service definitions
- Restart policies
- Resource limits
- Service dependencies
- Journald integration
- Environment files
- Timers and sockets

## 8.3 Common systemd Commands

```bash
# systemctl daemon-reload
# systemctl enable myapp
# systemctl start myapp
# systemctl restart myapp
# systemctl status myapp
# journalctl -u myapp -f
```

## 8.4 Generic systemd Unit Anatomy

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp/current
EnvironmentFile=/etc/myapp/myapp.env
ExecStart=/opt/myapp/current/bin/start.sh
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

## 8.5 systemd Service for Java

```ini
[Unit]
Description=Java Service
After=network.target

[Service]
Type=simple
User=javaapp
WorkingDirectory=/opt/javaapp/current
EnvironmentFile=/etc/javaapp/javaapp.env
ExecStart=/usr/bin/java -Xms512m -Xmx1024m -jar /opt/javaapp/current/app.jar
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.6 systemd Service for Python

```ini
[Unit]
Description=Python Gunicorn Service
After=network.target

[Service]
User=pythonapp
WorkingDirectory=/opt/pythonapp/current
EnvironmentFile=/etc/pythonapp/pythonapp.env
ExecStart=/opt/pythonapp/venv/bin/gunicorn --bind 127.0.0.1:8000 app:app
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.7 systemd Service for Node.js

```ini
[Unit]
Description=Node.js Service
After=network.target

[Service]
User=nodeapp
WorkingDirectory=/opt/nodeapp/current
EnvironmentFile=/etc/nodeapp/nodeapp.env
ExecStart=/usr/bin/node /opt/nodeapp/current/dist/server.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.8 systemd Service for Go

```ini
[Unit]
Description=Go Binary Service
After=network.target

[Service]
User=goapp
WorkingDirectory=/opt/goapp/current
EnvironmentFile=/etc/goapp/goapp.env
ExecStart=/opt/goapp/current/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.9 systemd Service for .NET

```ini
[Unit]
Description=.NET Service
After=network.target

[Service]
User=dotnetapp
WorkingDirectory=/opt/dotnetapp/current
EnvironmentFile=/etc/dotnetapp/dotnetapp.env
ExecStart=/usr/bin/dotnet /opt/dotnetapp/current/MyApp.dll
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## 8.10 Environment Variables Management

Options:

- Inline `Environment=` directives
- `EnvironmentFile=`
- Wrapper scripts
- Secret managers injecting env vars

Prefer `EnvironmentFile=` for maintainability.

Example:

```ini
EnvironmentFile=/etc/myapp/myapp.env
```

Example file:

```bash
APP_ENV=production
APP_PORT=8080
DATABASE_URL=postgresql://user:pass@db.internal/app
```

## 8.11 `.env` Files

`.env` files are common in development.

Production cautions:

- Restrict file permissions
- Avoid committing to source control
- Prefer centrally managed environment files or secret injection

Recommended permissions:

```bash
# chown root:myapp /etc/myapp/myapp.env
# chmod 640 /etc/myapp/myapp.env
```

## 8.12 Supervisor Overview

Supervisor is another process manager.

It is sometimes used when:

- systemd is unavailable
- Teams already standardized on it
- Multiple simple processes need orchestration in older environments

Install example:

```bash
# apt install -y supervisor
```

Program config example:

```ini
[program:pythonapp]
command=/opt/pythonapp/venv/bin/gunicorn --bind 127.0.0.1:8000 app:app
directory=/opt/pythonapp/current
autostart=true
autorestart=true
user=pythonapp
stdout_logfile=/var/log/pythonapp/stdout.log
stderr_logfile=/var/log/pythonapp/stderr.log
```

## 8.13 PM2 for Node.js

PM2 remains useful for:

- Clustered Node apps
- Developer-friendly rolling restarts
- Quick app dashboards

Example startup integration:

```bash
$ pm2 startup systemd
$ pm2 save
```

## 8.14 Resource Controls with systemd

Useful directives:

| Directive | Purpose |
|---|---|
| `MemoryMax=` | Limit memory usage |
| `CPUQuota=` | Limit CPU usage |
| `TasksMax=` | Cap total tasks |
| `LimitNOFILE=` | Max open files |
| `Restart=` | Restart behavior |

Example:

```ini
MemoryMax=1G
CPUQuota=200%
LimitNOFILE=65535
```

## 8.15 Journald Logging

View logs:

```bash
# journalctl -u myapp -f
```

Show last 100 lines:

```bash
# journalctl -u myapp -n 100 --no-pager
```

## 8.16 Graceful Shutdown Considerations

Applications should handle signals properly.

Important behaviors:

- Stop accepting new requests
- Drain active requests
- Close DB connections cleanly
- Flush logs/metrics if possible

systemd options that help:

```ini
TimeoutStopSec=30
KillSignal=SIGTERM
```

## 8.17 Common Process Management Mistakes

- Running services as root
- Missing restart policy
- Using relative paths in `ExecStart`
- Relying on shell-specific environment assumptions
- Not reloading systemd after unit changes
- Writing secrets directly into unit files with broad permissions

---

---

# 8. Configuration Management
## 8.1 Why Configuration Management Matters
Configuration separates deployable code from environment-specific behavior.

Examples of environment-specific settings:

- Database endpoints
- API credentials
- Feature flags
- Log levels
- Cache hosts
- Queue names
- Port bindings

## 8.2 Environment-Based Configuration
Typical environments:

- Development
- Test
- Staging
- Production

Best practice:

- Same code artifact across environments
- Different configuration injected per environment

## 8.3 Configuration Approaches
Common options:

- Environment variables
- YAML/JSON/TOML config files
- `.properties` files
- systemd `EnvironmentFile`
- Secret management systems
- Service discovery/config platforms

## 8.4 Configuration File Management
Recommended pattern:

```text
/opt/myapp/
├── current
├── releases/
└── shared/
    └── config/
        ├── app.env
        ├── application-prod.yml
        └── logging.yml
```

Keep mutable config in `shared/config/`.

Release directories should remain immutable.

## 8.5 Environment Variables vs Config Files
| Approach | Best For | Pros | Cons |
|---|---|---|---|
| Environment variables | Simple app settings, secrets injection | Standard, easy to automate | Can become unwieldy at scale |
| Config files | Larger structured configuration | Easier to read, version, template | Need secure distribution |
| Mixed model | Most real systems | Balanced flexibility | Needs discipline |

## 8.6 Secrets Management
Secrets include:

- Database passwords
- API tokens
- Private keys
- OAuth client secrets
- Encryption keys

Avoid:

- Hardcoding secrets in source
- Baking secrets into immutable artifacts
- Storing secrets in world-readable files

Prefer:

- Vault
- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager
- Kubernetes secrets with appropriate safeguards
- systemd EnvironmentFile with restricted permissions for simpler setups

## 8.7 Secret Injection Patterns
Patterns include:

- Env vars populated at service start
- Mounted files with strict permissions
- Sidecar or agent injection
- App runtime fetch with caching

## 8.8 Feature Flags
Feature flags enable controlled release behavior.

Use cases:

- Gradual rollout
- Emergency disablement
- A/B testing
- Decoupling deployment from release

Best practices:

- Name flags clearly
- Remove stale flags
- Monitor flag impact
- Separate operational kill switches from product experiments

## 8.9 Templating Configuration
Tools may include:

- `envsubst`
- Ansible templates
- Helm charts
- Jinja2 templates
- CI/CD variable substitution

Example with `envsubst`:

```bash
$ export APP_PORT=8080
$ envsubst < app.env.template > app.env
```

## 8.10 Config Validation
Validate before deployment.

Examples:

- Syntax validation for YAML/JSON
- Application startup dry-runs where possible
- Required variable checks in entrypoint scripts

Simple Bash pattern:

```bash
: "${DATABASE_URL:?DATABASE_URL is required}"
: "${APP_ENV:?APP_ENV is required}"
```

## 8.11 Multi-Environment Config Example
Development:

```bash
APP_ENV=development
LOG_LEVEL=debug
DATABASE_URL=postgresql://dev:dev@localhost/devdb
```

Staging:

```bash
APP_ENV=staging
LOG_LEVEL=info
DATABASE_URL=postgresql://stage:secret@staging-db.internal/stagedb
```

Production:

```bash
APP_ENV=production
LOG_LEVEL=warn
DATABASE_URL=postgresql://prod:secret@prod-db.internal/proddb
```

## 8.12 Configuration Drift Prevention
Prevent drift by:

- Storing config templates in version control
- Managing actual secret values securely outside git
- Automating host configuration
- Auditing live config regularly
- Avoiding manual hotfix changes on servers

## 8.13 Configuration Reloading
Some apps can reload config without full restart.

Patterns:

- `SIGHUP`-triggered reloads
- Reverse proxy reloads
- Dynamic config services

If live reload is unsupported, prefer controlled restart plus health checks.

## 8.14 Common Configuration Mistakes
- Mixing dev and prod credentials
- Using relative file paths
- Putting mutable config inside release directories
- Not documenting required variables
- No schema or validation for config structure
- Leaving stale feature flags enabled forever

## 8.15 Production Configuration Checklist
- Required variables documented
- Secrets stored securely
- Config separated from artifact
- File permissions restricted
- Environment naming consistent
- Validation performed before rollout
- Rollback config available

---

---

# Appendix G. End-to-End Sample Layouts

## G.1 Java Layout

```text
/opt/javaapp/
├── current
├── releases/
└── shared/
    ├── config/
    └── logs/
```

## G.2 Python Layout

```text
/opt/pythonapp/
├── current
├── releases/
├── shared/
│   ├── static/
│   └── logs/
└── venv/
```

## G.3 Node Layout

```text
/opt/nodeapp/
├── current
├── releases/
└── shared/
    └── logs/
```

## G.4 Go Layout

```text
/opt/goapp/
├── current
├── releases/
└── shared/
    └── config/
```

## G.5 .NET Layout

```text
/opt/dotnetapp/
├── current
├── releases/
└── shared/
    ├── config/
    └── logs/
```

---

---

# Appendix J. Security Considerations

## J.1 Principle of Least Privilege

Applications should:

- Run as dedicated users
- Have only required file permissions
- Bind to localhost unless public access is required
- Access only required secrets

## J.2 Artifact Integrity

Protect against tampering by:

- Using checksums
- Using signed packages where applicable
- Restricting artifact repository access

## J.3 Secret Hygiene

- Rotate secrets periodically
- Scope credentials narrowly
- Audit secret access
- Never log secret values

## J.4 Network Exposure

Only expose:

- Reverse proxy public ports
- Required internal service ports
- No unnecessary debug ports

## J.5 Log Sanitization

Ensure logs do not leak:

- Passwords
- Tokens
- Session IDs
- PII beyond approved policies

---
