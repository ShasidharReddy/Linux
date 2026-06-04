# Question 7: Containers, Operations & Monitoring

By this stage, the platform can safely host workloads. The next step is to decide where containers belong, how to run them operationally, and how to know the stack is healthy before users tell you it is broken.

## Container Runtime Choice

| Runtime | Pros | Cons | Use When |
|---------|------|------|----------|
| Docker | Mature ecosystem, huge community, Compose | daemon-based, historically root-heavy | developer workflows and simple production |
| Podman | daemonless, rootless support, systemd-friendly, Docker-compatible CLI | slightly smaller ecosystem | security-focused Linux shops and standalone production containers |
| containerd | lightweight, Kubernetes-native CRI | less direct admin UX without nerdctl | Kubernetes nodes |

## Recommendation

- **Podman** for standalone containers on dedicated container-host VMs
- **containerd** for Kubernetes nodes

Why:

- Podman aligns well with systemd and rootless operation
- containerd is the standard boring choice in kubeadm-based clusters
- separating runtime choices by use case keeps each layer simpler

## Where containers should run

Containers should run on **dedicated container-host VMs**, not directly on the hypervisors.

Why:

- workload isolation from the virtualization control plane
- easier patching and reboot management
- resource boundaries are clearer
- compromise of a container host is not compromise of the hypervisor itself

### Example sizing for container-host VMs

| Role | vCPU | RAM | Disk |
|------|------|-----|------|
| General container host | 8 | 32 GB | 200 GB |
| Logging/registry host | 8 | 32-64 GB | 500 GB |
| Small utility host | 4 | 16 GB | 100 GB |

## Container networking across hosts

### Option 1 — reverse proxy to per-host containers

Simple and production-friendly at modest scale. Each host runs containers locally; a reverse proxy knows where to send traffic.

### Option 2 — overlay networking

Makes addressing more transparent, but adds operational complexity and more moving parts.

### Option 3 — Kubernetes with CNI

Best for self-healing, declarative operations, and multi-team growth.

## Container networking options

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[External Traffic] --> B[Reverse Proxy]
    B --> C[Host 1 Containers]
    B --> D[Host 2 Containers]
    E[Overlay Network] --> C
    E --> D
    F[Kubernetes CNI] --> G[K8s Worker 1 Pods]
    F --> H[K8s Worker 2 Pods]
~~~

## Container operations baseline

### Private image registry

Use **Harbor** for production image storage, RBAC, retention, and scanning integrations.

### Harbor deployment outline

1. provision dedicated VM or HA pair
2. place it in PROD or a shared services segment, not in DMZ unless it must be internet-exposed
3. back it with durable storage
4. integrate with internal TLS and identity provider

### Harbor install outline

~~~bash
wget https://github.com/goharbor/harbor/releases/download/v2.11.0/harbor-online-installer-v2.11.0.tgz
tar -xzf harbor-online-installer-v2.11.0.tgz
cd harbor
cp harbor.yml.tmpl harbor.yml
./install.sh --with-trivy
~~~

## Image scanning with Trivy

~~~bash
trivy image harbor.infra.example.com/apps/api:1.2.3
trivy fs --severity HIGH,CRITICAL /srv/app
~~~

Make scanning part of CI so vulnerable images are blocked before promotion.

## Logging approach

- standalone Podman containers: stdout/stderr to journald or a log shipper
- Kubernetes: stdout/stderr collected by daemonset agents
- structured JSON logging whenever possible

### Podman example with resource limits and health checks

~~~bash
podman run -d \
  --name inventory-api \
  --restart=always \
  --memory=1024m \
  --cpus=2 \
  --health-cmd='curl -f http://127.0.0.1:8080/health || exit 1' \
  --health-interval=30s \
  -p 8080:8080 \
  harbor.infra.example.com/apps/inventory-api:1.2.3
~~~

### Systemd integration for Podman

~~~bash
podman generate systemd --name inventory-api --files --new
mv container-inventory-api.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now container-inventory-api.service
~~~

## Layer-by-layer verification table

| Layer | Health Check | Tool | Healthy Signal |
|-------|-------------|------|---------------|
| Physical | Server reachable | IPMI ping | responds, sensors OK |
| Network | VLAN connectivity | ping, iperf3 | expected path and bandwidth |
| Hypervisor | Cluster quorum | `pvecm status` | all nodes online |
| Storage | Mount and I/O | `df`, `fio` | mounted, expected IOPS |
| VM | OS booted | SSH, `systemctl` | SSH reachable, services active |
| Container | App responding | `curl`, `podman ps` | HTTP 200, container healthy |
| K8s | Cluster healthy | `kubectl get nodes` | all nodes Ready |

## Layer verification workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Verify Physical] --> B[Verify Network]
    B --> C[Verify Hypervisor]
    C --> D[Verify Storage]
    D --> E[Verify VM]
    E --> F[Verify Container]
    F --> G[Verify Kubernetes]
~~~

## Monitoring stack — production setup

### Metrics

Use **Prometheus** scraping exporters from every layer.

- `node_exporter` on hypervisors and VMs
- `libvirt_exporter` or Proxmox exporter for virtualization metrics
- `cAdvisor` for container metrics
- storage metrics from appliance exporter, SNMP, or custom exporters
- `blackbox_exporter` for synthetic checks

### Logs

Use **Filebeat or Fluent Bit** shipping to **Elasticsearch/OpenSearch** and visualize in **Kibana/OpenSearch Dashboards**.

### Alerts

Use **Alertmanager** with severity routing:

- critical: page on-call
- warning: open ticket / Slack
- info: audit channel or dashboard only

## Monitoring architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    HOSTS[Hosts and VMs] --> EXP[Exporters and Log Shippers]
    EXP --> PROM[Prometheus]
    EXP --> LOGS[Elasticsearch or OpenSearch]
    PROM --> ALERT[Alertmanager]
    PROM --> GRAF[Grafana]
    LOGS --> KIB[Kibana]
    ALERT --> ONCALL[Pager Slack Email]
~~~

## Prometheus scrape config example

~~~yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - pve01.infra.example.com:9100
          - pve02.infra.example.com:9100
          - pve03.infra.example.com:9100
          - web01.infra.example.com:9100

  - job_name: blackbox-http
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://app.example.com/health
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.monitoring.svc:9115
~~~

## Alert rules example

~~~yaml
groups:
  - name: am-platform
    rules:
      - alert: HypervisorNodeDown
        expr: up{job="node", instance=~"pve.*"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Hypervisor node down"

      - alert: StorageLatencyHigh
        expr: histogram_quantile(0.99, sum(rate(storage_latency_seconds_bucket[5m])) by (le)) > 0.020
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Storage P99 latency above 20ms"

      - alert: DiskAlmostFull
        expr: (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) < 0.10
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Filesystem has less than 10 percent free"
~~~

## Minimal Grafana dashboard JSON fragment

~~~json
{
  "title": "AM Platform Overview",
  "panels": [
    {"type": "stat", "title": "Node Up", "targets": [{"expr": "sum(up{job='node'})"}]},
    {"type": "timeseries", "title": "CPU Steal", "targets": [{"expr": "avg(rate(node_cpu_seconds_total{mode='steal'}[5m])) by (instance)"}]}
  ]
}
~~~

## Alert flow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[Exporter Metrics] --> B[Prometheus Rule]
    B --> C[Alertmanager]
    C --> D[Pager for Critical]
    C --> E[Chat for Warning]
    C --> F[Dashboard for Info]
~~~

## Steady-state signals to watch

- CPU steal time on hypervisors and VMs
- storage latency P99 and queue depth
- memory pressure, reclaim, and swap behavior
- interface drops or CRC errors
- certificate expiry countdown
- backup success or failure
- config drift from `ansible-playbook --check --diff`

## Operational checks for standalone container hosts

~~~bash
podman ps --format '{{.Names}} {{.Status}}'
podman stats --no-stream
journalctl -u container-inventory-api.service -b
systemctl status podman
~~~

## Common trade-offs

| Choice | Benefit | Cost |
|--------|---------|------|
| Single shared monitoring cluster | simpler to operate | larger blast radius if it fails |
| Separate metrics and logs clusters | better isolation | more infra to maintain |
| Rootless Podman | reduced privilege exposure | some networking/storage patterns need more care |

## Exit criteria

You are ready for [08-troubleshooting-guide.md](./08-troubleshooting-guide.md) and [09-kubernetes-deployment.md](./09-kubernetes-deployment.md) when:

- container hosts are isolated from hypervisors
- registry and scanning are in place
- Prometheus and log shipping cover each layer
- alert routes and thresholds are tested


## Procurement & Cost Analysis

### Monitoring and platform cost comparison

| Option | Cost Guidance | Best Fit | Watch-outs |
|--------|---------------|----------|------------|
| Prometheus + Grafana + Alertmanager | Free software | cost-efficient self-managed observability | requires infra and retention planning |
| Datadog | roughly $15+/host/month before add-ons | fast SaaS adoption | cost scales quickly with logs/APM |
| New Relic | usage-based / host-based | teams wanting managed observability | budgeting can be harder |
| Podman | Free | standalone container hosts | less opinionated platform tooling |
| OpenShift | $$$ enterprise | large regulated estates wanting opinionated platform | high license and skill cost |

### What to procure or budget

- VM capacity for monitoring stack, registry, and log storage.
- Object storage or additional disks if long-term metrics/log retention is required.
- TLS certs, SSO/MFA for Grafana/Harbor, and backup for dashboards/rules.
- Training budget for PromQL, alert design, container runtime troubleshooting, and on-call workflow.

## Resource Planning

### Monitoring sizing rules

| Dimension | Rule of Thumb | Example |
|-----------|---------------|---------|
| Prometheus retention | `samples_per_second x bytes_per_sample x retention_days` | 20k samples/s at 2 bytes compressed over 15 days ≈ plan 50-100 GB+ with headroom |
| Exporter coverage | every host plus storage/network/firewall targets | avoid blind spots on shared dependencies |
| Alert volume | <5 actionable pages/engineer/week | tune early to prevent fatigue |
| Growth target | 2x hosts in 18 months | size storage and scrape fan-out now |

### Staffing and alert fatigue management

- One platform engineer can operate a modest self-managed stack, but 24x7 paging needs shared ownership and documented runbooks.
- Use severity, deduplication, inhibition, and maintenance silences to keep pages actionable.
- Track noisy alerts by count, ack time, and false-positive rate monthly.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | Podman for standalone hosts | lightweight, rootless-friendly | running app containers on hypervisors rejected |
| ADR-02 | Prometheus-based monitoring | open, flexible, widely understood | SaaS-only observability rejected as default |
| ADR-03 | Harbor + Trivy | private registry plus scanning | public registry dependence rejected |
| ADR-04 | dedicated monitoring/logging VMs | keeps observability independent of workloads | fully co-located stack rejected |

### Integration points

- Linux host policies from [06-linux-os-layer.md](./06-linux-os-layer.md) define service, package, and security defaults.
- Alerting feeds [08-troubleshooting-guide.md](./08-troubleshooting-guide.md).
- Kubernetes monitoring extends this baseline in [09-kubernetes-deployment.md](./09-kubernetes-deployment.md).

## Planning & Timeline

### Rollout plan

| Week | Goal | Deliverables |
|------|------|--------------|
| Week 1 | runtime and registry | Podman host baseline, Harbor, image scanning policy |
| Week 2 | metrics and dashboards | Prometheus, Grafana, core dashboards, exporters |
| Week 3 | alerting and logs | Alertmanager routes, log shipping, on-call test |
| Week 4 | tuning | retention, capacity review, noise reduction |

### Prerequisites checklist

- Dedicated container hosts or VMs sized and hardened.
- DNS, TLS certs, and object/log storage destinations ready.
- Escalation paths and paging targets defined.
- Backup policy for dashboards, rules, and registry metadata.

### Risk register and rollback

| Risk | Impact | Mitigation | Rollback |
|------|--------|------------|----------|
| under-sized Prometheus disk | lost metrics | retention sizing and alerts | reduce retention temporarily, expand storage |
| alert storm | operator fatigue | inhibition, grouping, maintenance silences | disable bad rule group and review |
| registry outage | deployment delays | HA Harbor or backup registry strategy | use cached images / fail over |
| rootful container misuse | security exposure | rootless by default, policy review | redeploy service as managed unit |

## Advanced Production Configurations

- Add Thanos, Mimir, or Cortex when retention, HA, or multi-site metrics justify the extra complexity.
- For DR, replicate registry data and back up alert rules, Grafana dashboards, and Prometheus configs; restore testing matters more than YAML backups alone.
- Compliance impact: log retention, tamper resistance, SSO/MFA, and audit trails become mandatory under SOC 2/PCI/HIPAA.
- Capacity alerts: Prometheus TSDB >75%, scrape failures, Harbor storage growth, log ingest backlog, Alertmanager queue issues.
- Auto-remediation can restart a failed exporter or rotate log buffers, but paging policy and retention changes should stay reviewed.
- Manage alert fatigue with SLO-based alerts, symptom-over-cause hierarchy, and monthly deletion of low-value rules.

## Building & Deployment Runbook

1. Prepare dedicated container hosts and verify Podman baseline.
2. Deploy Harbor and image scanning, then validate access:
   ~~~bash
   podman login harbor.infra.example.com
   trivy image harbor.infra.example.com/platform/base:latest
   ~~~
3. Stand up Prometheus, Grafana, and Alertmanager; test configs before reload:
   ~~~bash
   promtool check config /etc/prometheus/prometheus.yml
   promtool test rules /etc/prometheus/alerts.yml
   amtool check-config /etc/alertmanager/alertmanager.yml
   ~~~
4. Install exporters on every layer and confirm targets are up.
5. Wire critical alerts to paging and warnings to chat/email.
6. Validation gate: break one test service, verify metrics, alert, dashboard, and log trail all line up end to end.

### Common mistakes to avoid during build

- Adopting SaaS monitoring by default without modeling 18-month cost.
- Paging on every threshold breach instead of on actionable symptoms.
- Co-locating monitoring with the very workloads it must diagnose.
- Ignoring Prometheus retention math until disks fill.


## Cross-references

- Linux host logic: [06-linux-os-layer.md](./06-linux-os-layer.md)
- Troubleshooting method: [08-troubleshooting-guide.md](./08-troubleshooting-guide.md)
- Kubernetes next: [09-kubernetes-deployment.md](./09-kubernetes-deployment.md)
- Related repo reference: [../Virtual-Setup/07-production-kubernetes-setup.md](../Virtual-Setup/07-production-kubernetes-setup.md)
