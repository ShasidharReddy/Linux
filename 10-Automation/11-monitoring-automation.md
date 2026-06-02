# Monitoring Automation

[Back to guide index](README.md)

Automation without observability is risky.

Monitoring systems validate whether automated changes produced healthy outcomes.

They can also trigger remediation workflows.

## 11.1 Why Monitoring Matters in Automation

Monitoring answers:

- Did the service come up after deployment?
- Did latency or error rates worsen?
- Did a configuration change break health checks?
- Are hosts conforming to expected package and service states?

## 11.2 Prometheus + Grafana Overview

Prometheus collects metrics.

Grafana visualizes them.

Common Linux monitoring signals:

- CPU usage
- Memory usage
- Disk space
- Filesystem inodes
- Network throughput
- Process counts
- Service availability
- HTTP response metrics

## 11.3 Node Exporter Setup Example

### Install node_exporter manually

```bash
sudo useradd -r -s /usr/sbin/nologin node_exporter
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xzf node_exporter-1.8.1.linux-amd64.tar.gz
sudo install node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/node_exporter
```

### systemd service

```ini
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

## 11.4 Ansible Example: Deploy Node Exporter

```yaml
- name: Install node exporter
  hosts: all
  become: true
  tasks:
    - name: Ensure node_exporter user exists
      ansible.builtin.user:
        name: node_exporter
        system: true
        shell: /usr/sbin/nologin
        create_home: false

    - name: Copy node_exporter binary
      ansible.builtin.copy:
        src: files/node_exporter
        dest: /usr/local/bin/node_exporter
        mode: '0755'
      notify: Restart node_exporter

    - name: Install service unit
      ansible.builtin.copy:
        dest: /etc/systemd/system/node_exporter.service
        content: |
          [Unit]
          Description=Prometheus Node Exporter
          After=network.target

          [Service]
          User=node_exporter
          Group=node_exporter
          ExecStart=/usr/local/bin/node_exporter
          Restart=always

          [Install]
          WantedBy=multi-user.target
        mode: '0644'
      notify:
        - Reload systemd
        - Restart node_exporter

    - name: Enable and start node_exporter
      ansible.builtin.service:
        name: node_exporter
        state: started
        enabled: true

  handlers:
    - name: Reload systemd
      ansible.builtin.command: systemctl daemon-reload
      changed_when: true

    - name: Restart node_exporter
      ansible.builtin.service:
        name: node_exporter
        state: restarted
```

## 11.5 Prometheus Scrape Config Example

```yaml
global:
  scrape_interval: 15s

auto_scrape:
  evaluation_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - web1.example.com:9100
          - web2.example.com:9100
          - db1.example.com:9100
```

## 11.6 Alerting Example

```yaml
groups:
  - name: linux-hosts
    rules:
      - alert: HostHighCpu
        expr: 100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"

      - alert: HostDiskSpaceLow
        expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"}) * 100 < 10
        for: 15m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
```

## 11.7 Grafana Dashboard Concepts

Dashboards should include:

- Host overview
- CPU/memory/disk panels
- Network I/O
- Service availability
- Deployment markers
- Alert state indicators

## 11.8 Alertmanager and Routing

Alertmanager can route alerts by:

- Severity
- Team ownership
- Environment
- Time windows
- Alert grouping

## 11.9 Auto-Remediation

Auto-remediation means automation responds to monitoring signals.

Examples:

- Restart a failed systemd service
- Rotate logs when disk consumption spikes
- Remove a node from load balancer pool after health failures
- Re-run configuration convergence on detected drift

## 11.10 Example Auto-Remediation Workflow

1. Prometheus fires an alert.
2. Alertmanager routes it to a webhook.
3. Automation platform validates context.
4. Safe remediation script or playbook runs.
5. Monitoring verifies recovery.
6. Incident ticket is updated.

## 11.11 Safety Controls for Auto-Remediation

| Control | Why It Matters |
|---|---|
| Scope limits | Avoid broad unintended actions |
| Rate limits | Prevent flapping loops |
| Approval gates for risky actions | Protect production |
| Idempotent remediation | Safe retries |
| Observability | Confirm actual outcome |
| Rollback criteria | Recover from failed remediation |

## 11.12 Monitoring-Driven Deployment Validation

After automation runs, monitoring should verify:

- Process is running
- Port is listening
- Health endpoint is green
- Error rate is not elevated
- Resource saturation is acceptable

## 11.13 Practical Example: Nginx Health Check with Auto-Fix

### Detection logic

- Alert when nginx is down on a host for 2 minutes.

### Automated action

- Run Ansible playbook to restart nginx.

### Escalation rule

- If service does not recover after one attempt, page the on-call engineer.

## 11.14 Monitoring Summary

Monitoring closes the loop between change execution and operational confidence.

---

---

# Appendix B: Troubleshooting Checklists

## B.1 Ansible Troubleshooting

- Verify SSH reachability.
- Verify Python interpreter on target hosts.
- Check inventory targeting.
- Run with `-vvv` for more verbosity.
- Validate sudo permissions.
- Confirm module dependencies exist.
- Test playbook syntax.

## B.2 Terraform Troubleshooting

- Run `terraform validate`.
- Confirm backend access.
- Review provider credentials.
- Inspect state locks.
- Review the exact plan output.
- Import existing resources if unmanaged.

## B.3 Puppet Troubleshooting

- Check agent logs.
- Validate manifests.
- Verify certificate trust.
- Inspect Hiera data resolution.
- Review catalog compilation errors.

## B.4 Chef Troubleshooting

- Inspect Chef Client logs.
- Validate cookbook dependencies.
- Review attribute precedence.
- Test recipes locally.
- Confirm Chef Server connectivity.

## B.5 Salt Troubleshooting

- Confirm minion connectivity.
- Check top file targeting.
- Review pillar resolution.
- Use test mode before broad apply.
- Inspect event bus behavior for reactor issues.

## B.6 Cloud-Init Troubleshooting

- Check cloud-init status.
- Inspect journal logs.
- Review user-data syntax.
- Confirm image compatibility.
- Re-test on clean instances.

## B.7 Packer Troubleshooting

- Run `packer validate`.
- Inspect builder credentials.
- Confirm SSH communicator settings.
- Check provisioner exit codes.
- Review generated manifests.

---
