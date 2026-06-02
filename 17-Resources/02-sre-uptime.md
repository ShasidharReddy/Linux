# SRE, Uptime, and Observability

This guide collects SRE reading, uptime math, incident management, observability, and automation references.

## SRE & Uptime

<table>
  <thead>
    <tr>
      <th>Resource</th>
      <th>URL</th>
      <th>Purpose/Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>uptime.is</td>
      <td>https://uptime.is/</td>
      <td>A simple SLA calculator that translates availability targets into allowed downtime, making SLOs concrete.</td>
    </tr>
    <tr>
      <td>Google SRE Book</td>
      <td>https://sre.google/sre-book/table-of-contents/</td>
      <td>The foundational SRE text for SLIs, SLOs, toil reduction, alerting, and operational maturity.</td>
    </tr>
    <tr>
      <td>Google SRE Workbook</td>
      <td>https://sre.google/workbook/table-of-contents/</td>
      <td>Practical companion to the SRE Book with applied guidance for implementation and day-two operations.</td>
    </tr>
    <tr>
      <td>Building Secure and Reliable Systems</td>
      <td>https://sre.google/books/</td>
      <td>Connects reliability and security thinking; especially useful for production architecture reviews.</td>
    </tr>
    <tr>
      <td>Google Reliability Framework</td>
      <td>https://cloud.google.com/architecture/framework/reliability</td>
      <td>Provides a cloud-focused reliability checklist you can use during design and readiness reviews.</td>
    </tr>
    <tr>
      <td>PagerDuty Response Guide</td>
      <td>https://response.pagerduty.com/</td>
      <td>A strong reference for incident command, communications, escalation, and post-incident improvements.</td>
    </tr>
    <tr>
      <td>incident.io Guide</td>
      <td>https://incident.io/guide</td>
      <td>Concise operational advice for running incidents cleanly and communicating clearly under pressure.</td>
    </tr>
    <tr>
      <td>Atlassian Incident Management</td>
      <td>https://www.atlassian.com/incident-management</td>
      <td>Good background on incident lifecycles, postmortems, and stakeholder communication patterns.</td>
    </tr>
    <tr>
      <td>USENIX SREcon</td>
      <td>https://www.usenix.org/srecon</td>
      <td>Conference archive with real-world SRE talks that expose what works beyond textbook theory.</td>
    </tr>
    <tr>
      <td>OpenSLO</td>
      <td>https://github.com/OpenSLO/OpenSLO</td>
      <td>A vendor-neutral SLO specification that helps standardize reliability goals across teams and tools.</td>
    </tr>
    <tr>
      <td>Sloth</td>
      <td>https://sloth.dev/</td>
      <td>Generates Prometheus SLO rules, which reduces manual error when implementing error-budget alerting.</td>
    </tr>
  </tbody>
</table>


## Monitoring & Observability

<table>
  <thead>
    <tr>
      <th>Resource</th>
      <th>URL</th>
      <th>Purpose/Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Prometheus</td>
      <td>https://prometheus.io/</td>
      <td>The core open-source metrics and alerting stack for modern Linux and cloud-native systems.</td>
    </tr>
    <tr>
      <td>Alertmanager</td>
      <td>https://prometheus.io/docs/alerting/latest/alertmanager/</td>
      <td>Explains grouping, routing, deduplication, and silence management for actionable alerting.</td>
    </tr>
    <tr>
      <td>node_exporter</td>
      <td>https://github.com/prometheus/node_exporter</td>
      <td>Essential for exposing host-level metrics such as CPU, memory, filesystem, and network counters.</td>
    </tr>
    <tr>
      <td>blackbox_exporter</td>
      <td>https://github.com/prometheus/blackbox_exporter</td>
      <td>Useful for probing HTTP, TCP, ICMP, and DNS endpoints from the outside in.</td>
    </tr>
    <tr>
      <td>Grafana</td>
      <td>https://grafana.com/</td>
      <td>The standard dashboard and visualization layer for metrics, logs, and traces.</td>
    </tr>
    <tr>
      <td>Loki</td>
      <td>https://grafana.com/oss/loki/</td>
      <td>A label-based log system that pairs well with Prometheus-style operational workflows.</td>
    </tr>
    <tr>
      <td>Jaeger</td>
      <td>https://www.jaegertracing.io/</td>
      <td>Distributed tracing system that helps uncover latency bottlenecks across service boundaries.</td>
    </tr>
    <tr>
      <td>OpenTelemetry</td>
      <td>https://opentelemetry.io/</td>
      <td>The open standard for collecting metrics, logs, and traces without vendor lock-in.</td>
    </tr>
    <tr>
      <td>Elastic Stack</td>
      <td>https://www.elastic.co/elastic-stack</td>
      <td>Widely used for log aggregation, search, visualization, and general observability pipelines.</td>
    </tr>
    <tr>
      <td>Netdata</td>
      <td>https://www.netdata.cloud/</td>
      <td>Excellent for instant, host-centric dashboards with very low setup effort.</td>
    </tr>
    <tr>
      <td>Datadog</td>
      <td>https://www.datadoghq.com/</td>
      <td>Popular commercial observability platform with deep integrations and fast time to value.</td>
    </tr>
    <tr>
      <td>VictoriaMetrics</td>
      <td>https://victoriametrics.com/</td>
      <td>High-performance metrics storage that is especially useful at larger retention scales.</td>
    </tr>
    <tr>
      <td>Thanos</td>
      <td>https://thanos.io/</td>
      <td>Adds long-term storage, deduplication, and global query capabilities to Prometheus deployments.</td>
    </tr>
    <tr>
      <td>Fluent Bit</td>
      <td>https://fluentbit.io/</td>
      <td>A lightweight log and metrics forwarder that fits well on Linux hosts and Kubernetes nodes.</td>
    </tr>
  </tbody>
</table>


## DevOps & Automation

<table>
  <thead>
    <tr>
      <th>Resource</th>
      <th>URL</th>
      <th>Purpose/Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Ansible Galaxy</td>
      <td>https://galaxy.ansible.com/</td>
      <td>Find reusable roles and collections instead of rebuilding automation from scratch.</td>
    </tr>
    <tr>
      <td>Ansible Documentation</td>
      <td>https://docs.ansible.com/</td>
      <td>Authoritative playbook, module, and inventory guidance for configuration management work.</td>
    </tr>
    <tr>
      <td>Terraform Registry</td>
      <td>https://registry.terraform.io/</td>
      <td>The main source for providers and modules used in infrastructure-as-code workflows.</td>
    </tr>
    <tr>
      <td>Terraform Documentation</td>
      <td>https://developer.hashicorp.com/terraform/docs</td>
      <td>Essential for state, plans, modules, and expression syntax when building reliable IaC.</td>
    </tr>
    <tr>
      <td>OpenTofu Documentation</td>
      <td>https://opentofu.org/docs/</td>
      <td>Useful if you prefer an open governance alternative with Terraform-compatible workflows.</td>
    </tr>
    <tr>
      <td>Puppet Forge</td>
      <td>https://forge.puppet.com/</td>
      <td>Finds maintained Puppet modules and common implementation patterns quickly.</td>
    </tr>
    <tr>
      <td>Chef Supermarket</td>
      <td>https://supermarket.chef.io/</td>
      <td>Repository of Chef cookbooks that accelerates bootstrapping and configuration reuse.</td>
    </tr>
    <tr>
      <td>Vagrant Cloud</td>
      <td>https://app.vagrantup.com/</td>
      <td>Useful for browsing box images and building repeatable local lab environments.</td>
    </tr>
    <tr>
      <td>Packer Documentation</td>
      <td>https://developer.hashicorp.com/packer/docs</td>
      <td>Important for golden-image pipelines and reproducible machine image creation.</td>
    </tr>
    <tr>
      <td>GitHub Actions Marketplace</td>
      <td>https://github.com/marketplace?type=actions</td>
      <td>Lets you find reusable CI/CD actions quickly while comparing popularity and maintenance signals.</td>
    </tr>
    <tr>
      <td>Jenkins Documentation</td>
      <td>https://www.jenkins.io/doc/</td>
      <td>Still valuable in environments with established Jenkins pipelines and plugin ecosystems.</td>
    </tr>
    <tr>
      <td>Argo CD</td>
      <td>https://argo-cd.readthedocs.io/</td>
      <td>Key GitOps reference for syncing Kubernetes state from version control safely.</td>
    </tr>
  </tbody>
</table>
