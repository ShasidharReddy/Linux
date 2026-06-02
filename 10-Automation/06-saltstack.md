# SaltStack

[Back to guide index](README.md)

SaltStack, often referred to as Salt, is an automation and configuration management platform that supports remote execution, desired-state configuration, event-driven automation, and large-scale orchestration.

## 6.1 Salt Architecture

Salt commonly uses a master-minion architecture.

| Component | Description |
|---|---|
| Master | Central coordinator |
| Minion | Managed node agent |
| State | Desired configuration definition |
| Pillar | Sensitive or targeted data |
| Grain | Static or discovered system metadata |
| Formula | Reusable Salt state package |
| Reactor | Event-driven automation engine |

## 6.2 Master-Minion Model

- Minions connect to the master.
- Commands can be executed remotely.
- States enforce desired configuration.
- Events can trigger automated reactions.

## 6.3 Remote Execution

### Ping all minions

```bash
salt '*' test.ping
```

### Run uptime on web minions

```bash
salt 'web*' cmd.run 'uptime'
```

## 6.4 States

States define desired configuration.

### nginx state example

```yaml
nginx:
  pkg.installed: []

nginx_service:
  service.running:
    - name: nginx
    - enable: True
    - require:
      - pkg: nginx
```

## 6.5 Applying States

```bash
salt 'web*' state.apply nginx
```

## 6.6 Top File

The top file maps minions to states.

```yaml
base:
  'web*':
    - nginx
  'db*':
    - mysql
```

## 6.7 Pillars

Pillars hold targeted data, often including sensitive configuration.

### pillar example

```yaml
app:
  env: production
  port: 9000
```

Use pillars for:

- Secrets
- Environment-specific variables
- Target-specific settings

## 6.8 Grains

Grains provide system information.

Examples:

- os
- roles
- ipv4
- cpuarch
- virtual

### Grain-based conditional state

```yaml
{% if grains['os_family'] == 'Debian' %}
nginx:
  pkg.installed:
    - name: nginx
{% endif %}
```

## 6.9 Jinja in States

Salt states often use Jinja templating.

```yaml
/etc/myapp/config.ini:
  file.managed:
    - source: salt://myapp/files/config.ini.j2
    - template: jinja
    - context:
        app_port: {{ pillar['app']['port'] }}
```

## 6.10 Formulas

Formulas are reusable Salt state collections similar to Ansible roles or Puppet modules.

## 6.11 Reactor System

Salt Reactor listens for events and triggers actions.

Use cases:

- Auto-remediation on service failure
- Security response actions
- Scaling workflows
- Pipeline integration

## 6.12 Orchestration

Salt can coordinate multi-node workflows.

Examples:

- Drain node from load balancer
- Apply state to node
- Verify health
- Rejoin load balancer

## 6.13 Example: User Management State

```yaml
adminops:
  group.present: []

deploy:
  user.present:
    - shell: /bin/bash
    - groups:
      - adminops
    - require:
      - group: adminops
```

## 6.14 Example: SSH Hardening State

```yaml
/etc/ssh/sshd_config:
  file.replace:
    - pattern: '^PermitRootLogin\s+.*'
    - repl: 'PermitRootLogin no'
    - append_if_not_found: True

sshd_service:
  service.running:
    - name: sshd
    - enable: True
    - watch:
      - file: /etc/ssh/sshd_config
```

## 6.15 Example: Package Baseline

```yaml
common_packages:
  pkg.installed:
    - pkgs:
      - curl
      - vim
      - git
      - rsync
```

## 6.16 Salt File Server

Salt can serve files from the master using `salt://` URLs.

Example:

```yaml
/opt/myapp/app.conf:
  file.managed:
    - source: salt://myapp/files/app.conf
```

## 6.17 Scheduling

Salt can schedule jobs on minions.

Use cases:

- Periodic state runs
- Log cleanup tasks
- Health checks

## 6.18 Salt SSH

Salt also supports an agentless mode with Salt SSH.

This is useful when installing minions is not desirable.

## 6.19 Best Practices for Salt

1. Keep states modular.
2. Use pillars for environment data.
3. Use grains for targeting.
4. Test top file targeting carefully.
5. Prefer formulas for reuse.
6. Use reactor carefully for controlled automation.

## 6.20 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| Overcomplicated Jinja | Hard debugging | Keep state logic simple |
| Too much secret data in plain files | Security risk | Use secured pillar handling |
| Broad target patterns | Large blast radius | Use precise targeting |
| Event loops in reactor | Unstable automation | Add filtering and guard conditions |

## 6.21 When Salt Fits Well

Salt is particularly effective when you need both configuration management and powerful event-driven remote execution at scale.

## 6.22 Salt Summary

SaltStack combines configuration enforcement, orchestration, and event-driven automation in a single platform.

---
