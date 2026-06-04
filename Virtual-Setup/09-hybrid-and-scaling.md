# 09 - Hybrid and Scaling

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                   Hybrid and Scaling                        │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This guide covers hybrid deployment models, multi-site designs, performance testing, migration paths, and scale strategies for the e-commerce platform. It ties together the VM, Docker, and Kubernetes material from the earlier documents in this directory.

## Hybrid Architecture

A hybrid model is useful when:

- databases remain on-prem for latency, licensing, or compliance reasons
- burst traffic is handled in cloud Kubernetes clusters
- teams need a gradual migration rather than a full rewrite of operations
- monitoring and CI/CD must span both private and public infrastructure

## On-Prem + Cloud Burst Design

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Users[Users] --> GSLB[Global DNS / GSLB]
    GSLB --> OnPrem[On-Prem K8s / VM Stack]
    GSLB --> Cloud[Cloud K8s Burst Cluster]
    OnPrem --> VPN[VPN / Interconnect]
    Cloud --> VPN
    OnPrem --> Shared[Shared CI/CD and Observability]
    Cloud --> Shared
~~~

### Consistent Tooling

Use the same tools across both sides where possible:

- GitOps repo and promotion model
- container registry
- secrets workflow
- monitoring dashboards
- tracing and log pipelines
- deployment policy and admission checks

### Connectivity options

- site-to-site VPN
- direct cloud interconnect
- private load balancers plus DNS steering
- service mesh multi-cluster where applicable

## Multi-Site Setup

### Active-Active Pattern

Run traffic in two data centers at the same time.

Requirements:

- GSLB or geo-aware DNS
- synchronized application versions
- shared session strategy or stateless session design
- multi-site database replication design
- clear cache invalidation behavior

## Multi-Site Architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    G[Global Load Balancer] --> DC1[Data Center 1]
    G --> DC2[Data Center 2]
    DC1 --> DB1[Primary / regional DB]
    DC2 --> DB2[Replica / regional DB]
    DB1 --> DB2
    DC1 --> Cache1[Redis / CDN cache]
    DC2 --> Cache2[Redis / CDN cache]
~~~

### Database replication across sites

Options:

- asynchronous primary-replica across regions
- group replication or Galera-style clusters where latency allows
- application-level sharding by geography or tenant
- write-local/read-global patterns with careful consistency handling

### Cache invalidation strategies

- use short TTLs for catalog fragments
- publish invalidation events via RabbitMQ or Kafka
- keep cart and checkout state strongly consistent where required
- prefer stateless web tiers and durable shared session/cache backends

## Scaling Strategies

### Vertical scaling

Vertical scaling means bigger VMs, nodes, or container limits.

Use it when:

- the workload is stateful and hard to shard
- the bottleneck is memory for MySQL or Elasticsearch
- the service count is still small

Limits:

- hardware ceilings
- longer restart impact
- diminishing returns on single-node databases

### Horizontal scaling

Horizontal scaling means more replicas.

Per tier guidance:

- web tier: scale out aggressively behind load balancers
- app tier: scale with CPU, queue depth, and request rate
- cache tier: add replicas or clustering where supported
- search tier: add nodes and shard intentionally
- database tier: add read replicas and split write patterns carefully

### Read replicas and caching tiers

- offload product browsing reads to replicas when your consistency model permits it
- use Redis for sessions, carts, and hot catalog data
- use CDNs for static assets and cache-friendly content
- move heavy reports and exports onto worker queues

### Database sharding for e-commerce

Potential shard keys:

- tenant or merchant ID
- geographic region
- customer segment
- order ID range

Cautions:

- cross-shard transactions become harder
- reporting queries need aggregation layers
- inventory and payment consistency must be designed carefully

### Async processing

Use queues for:

- email sending
- stock synchronization
- recommendation generation
- search indexing
- image processing
- analytics export jobs

## Performance Testing

### Tools

- `k6` for scriptable HTTP load tests
- Locust for Python-based user behavior modeling
- JMeter for established enterprise testing patterns

### k6 example - checkout flow

~~~javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 50,
  duration: '5m',
  thresholds: {
    http_req_duration: ['p(95)<800'],
    http_req_failed: ['rate<0.01']
  }
};

export default function () {
  let res = http.get('https://shop.example.com/healthz');
  check(res, { 'healthz ok': (r) => r.status === 200 });

  res = http.get('https://shop.example.com/products');
  check(res, { 'catalog ok': (r) => r.status === 200 });

  sleep(1);
}
~~~

Run it:

~~~bash
k6 run checkout.js
~~~

### Locust example

~~~python
from locust import HttpUser, task, between

class Shopper(HttpUser):
    wait_time = between(1, 3)

    @task(3)
    def browse(self):
        self.client.get('/products')

    @task(1)
    def health(self):
        self.client.get('/healthz')
~~~

Run it:

~~~bash
locust -f locustfile.py --host=https://shop.example.com
~~~

### Capacity planning workflow

1. establish a baseline with one replica count
2. measure CPU, memory, DB latency, and queue depth
3. increase load gradually
4. identify the first bottleneck tier
5. scale that tier and repeat the test
6. record safe operating ranges and alert thresholds

### Regression testing

- keep standard load profiles in version control
- run smaller performance suites on every major release
- compare P95 and error-rate deltas between releases
- fail promotion if performance regression exceeds tolerance

## Migration Paths

### Physical to VM migration

1. inventory the current host OS, packages, data paths, ports, and dependencies
2. create a VM template with the target OS and matching packages
3. rsync application files and databases
4. switch DNS or load balancer traffic
5. keep rollback by preserving the physical host until validation completes

### VM to container migration

1. isolate application dependencies from the VM OS
2. build Dockerfiles for web and app services
3. externalize configuration via env vars or config files
4. move data to volumes or managed storage
5. validate with Docker Compose before cutting over
6. keep the VM deployment available for rollback

### Compose to Kubernetes migration

1. inventory services, ports, volumes, and environment variables
2. convert stateless services to Deployments and Services
3. convert data services to StatefulSets or managed services
4. create Ingress, ConfigMaps, and Secrets
5. add readiness and liveness probes
6. test in staging before production cutover

## Migration Path Diagram

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    P[Physical Servers] --> V[Virtual Machines]
    V --> C[Containers / Compose]
    C --> K[Kubernetes]
~~~

### Rollback plans

- keep DNS TTL low before migrations
- snapshot VMs before risky changes
- preserve old Compose files and image tags
- keep database backups and schema rollback scripts ready
- practice rollback under load, not only in idle windows

## Practical Scaling Playbook

### When to scale web tier

Indicators:

- rising ingress latency
- CPU saturation on frontend pods or VMs
- increasing TLS handshake load

Actions:

- add replicas
- enable keepalive and gzip/brotli appropriately
- offload static content to CDN

### When to scale app tier

Indicators:

- request queueing
- rising response time with stable DB latency
- worker backlog growth

Actions:

- add app replicas
- separate sync and async workloads
- cache read-heavy catalog endpoints

### When to scale database tier

Indicators:

- slow query growth
- high disk latency
- replication lag
- connection saturation

Actions:

- tune queries and indexes first
- add read replicas
- increase buffer pool and storage performance
- evaluate sharding only after simpler optimizations

## Hybrid Operations Checklist

- one source of truth for manifests and infrastructure code
- one image registry or mirrored registry model
- one observability standard across environments
- environment-specific but policy-consistent secrets handling
- documented failover decision points between on-prem and cloud capacity


## Detailed Hybrid Runbook
When you operate one platform across on-prem and cloud, document exact failover and burst procedures.
### Burst-to-cloud checklist
- confirm container image parity between sites
- confirm database replication health and lag thresholds
- pre-warm caches or accept an initial cache miss period
- verify DNS, CDN, or GSLB health checks
- confirm secrets and certificates are already present in the burst cluster
### Example verification commands
~~~bash
kubectl get nodes -A
kubectl get pods -n ecommerce-prod
kubectl top pods -n ecommerce-prod
mysql -h db-replica.example.internal -u readonly -p -e 'SHOW REPLICA STATUS\G'
redis-cli -h redis-cache.example.internal INFO replication
~~~
### Failback checklist
1. drain burst traffic gradually using weighted DNS
2. verify queue depth is near zero before shutting down workers
3. reconcile inventory and order state
4. disable cloud-only write paths
5. archive incident notes and capacity metrics
## Multi-site operational patterns
### Session strategy options
| Pattern | Best Fit | Risk |
| --- | --- | --- |
| Sticky sessions | simple legacy web stacks | poor failover flexibility |
| Redis shared session store | standard web apps | cross-site latency if central only |
| Stateless tokens | APIs and headless commerce | token revocation and key rotation planning |
### Search and catalog sync
- schedule full reindex windows during low-traffic periods
- use event-driven partial updates for product changes
- monitor index lag separately from database lag
- treat search relevance tuning as a release artifact
### Payment and checkout concerns
- keep idempotency keys for payment retries
- store checkout state durably before calling gateways
- never let active-active routing create duplicate charge paths
- rehearse partial regional outage handling with sandbox payment providers
## More load and stress testing examples
### JMeter CLI example
~~~bash
jmeter -n -t ecommerce-checkout.jmx -l results.jtl -e -o reports/html
~~~
### k6 ramping example
~~~javascript
import http from 'k6/http';
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 300 },
    { duration: '2m', target: 0 }
  ]
};
export default function () {
  http.get('https://shop.example.com/products');
}
~~~
### What to measure during tests
- frontend latency by route group
- backend queue wait time
- MySQL slow queries and replication lag
- Redis evictions and memory fragmentation
- Elasticsearch heap usage and search latency
## Capacity planning notes
A rough starting model for commerce traffic:
- browsing traffic is usually far higher than checkout traffic
- search bursts can dominate CPU even when orders remain stable
- promotions create read spikes first and write spikes later
- background jobs often become the hidden bottleneck after the frontends scale
### Example planning table
| Visitors/Day | Web Replicas | App Replicas | DB Pattern | Cache Pattern |
| --- | --- | --- | --- | --- |
| 1,000 | 2 | 2 | single primary | single Redis |
| 10,000 | 3-4 | 3-4 | primary + replica | Redis with persistence |
| 100,000 | 6-8 | 6-10 | primary + multiple replicas or shard plan | Redis cluster |
## Migration decision triggers
Move from one platform stage to the next when you observe:
- VM management effort rising faster than application changes
- slow environment rebuilds blocking releases
- frequent manual scaling events during campaigns
- inconsistent host configuration causing deployment drift
- a need for policy-driven deployment and self-healing

## Common Pitfalls

- stretching synchronous database clustering across high-latency links
- assuming cloud burst works without image, DNS, and secret parity
- scaling stateless services while ignoring shared bottlenecks like MySQL
- running performance tests without realistic user journeys
- migrating multiple layers at once without rollback checkpoints

## Next Steps

- revisit [07-production-kubernetes-setup.md](./07-production-kubernetes-setup.md) for multi-cluster resilience and policy controls
- use [08-ci-cd-and-automation.md](./08-ci-cd-and-automation.md) to automate migration and promotion workflows
- compare with the VM-first path in [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md) for gradual adoption

## Summary

Scaling and hybrid operations succeed when you treat architecture, automation, data consistency, and observability as one problem rather than separate projects. Build the simplest platform that meets today’s traffic, then evolve it step by step using tested migration paths and measurable performance data.

[← Back to Virtual Setup](./README.md)
