# 06 Detailed Architecture Diagrams

Complete Visual Architecture Reference — Cloud plus 10-App System

This guide is the visual companion for the cloud-target architecture. It emphasizes large, reference-grade Mermaid diagrams that can be reused in design reviews, migration planning, operations, and security conversations.
Each section stays intentionally brief outside the diagrams: a concise explanation, a usage cue, the diagram itself, and a short set of takeaways that highlight the main design decisions.

---

## Diagram 1 — Complete 10-App System Architecture (The Big Picture)

This is the main system map for the cloud-native ecommerce platform, showing user entrypoints, the web-facing BFF, all ten application domains, their data stores, asynchronous messaging, and the shared observability layer.
Solid arrows indicate synchronous request paths such as REST or gRPC, while dashed arrows show asynchronous event publication and side-effect processing.

**When to use:** Use this as the primary reference when you need one diagram that explains the whole platform to engineering, operations, security, and product stakeholders.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    shoppers[Users / Browsers / Mobile Apps] --> cdn_edge[CDN + Edge Cache]
    cdn_edge --> api_gateway[API Gateway / WAF / Rate Limits]
    api_gateway --> web_bff[Web App BFF]

    subgraph app_domain[10 Application Domains]
        auth_service[User / Auth Service]
        catalog_service[Product Catalog Service]
        search_service[Search Service]
        cart_service[Cart Service]
        order_service[Order Service]
        inventory_service[Inventory Service]
        payment_service[Payments Service]
        notification_service[Notification Service]
        shipping_service[Shipping Service]
        admin_service[Admin / Backoffice Service]
    end

    subgraph data_domain[Databases and State]
        postgres_orders[PostgreSQL Orders DB]
        postgres_users[PostgreSQL Users DB]
        postgres_inventory[PostgreSQL Inventory DB]
        mongo_catalog[MongoDB Catalog DB]
        redis_cache_main[Redis Cache + Sessions]
        elastic_search_idx[Elasticsearch Search Index]
        kafka_bus[Kafka Event Bus]
        object_storage[S3 / Blob Storage]
    end

    subgraph integration_domain[External and Shared]
        payment_gateway_ext[Stripe / Razorpay]
        email_provider[Email Provider]
        sms_provider[SMS Provider]
        carrier_api[Shipping Carrier APIs]
        fraud_api[Fraud Scoring API]
    end

    subgraph observability_domain[Monitoring and Logging]
        prom_arch[Prometheus]
        graf_arch[Grafana]
        elk_arch[ELK Stack]
        otel_arch[OpenTelemetry Collector]
    end

    web_bff -->|REST| auth_service
    web_bff -->|REST| catalog_service
    web_bff -->|REST| search_service
    web_bff -->|REST| cart_service
    web_bff -->|REST| order_service
    web_bff -->|REST| admin_service
    order_service -->|gRPC| inventory_service
    order_service -->|REST| payment_service
    order_service -.->|order.created| notification_service
    order_service -.->|order.created| shipping_service
    catalog_service -.->|catalog.updated| search_service
    auth_service --> postgres_users
    catalog_service --> mongo_catalog
    search_service --> elastic_search_idx
    cart_service --> redis_cache_main
    order_service --> postgres_orders
    inventory_service --> postgres_inventory
    payment_service --> postgres_orders
    admin_service --> mongo_catalog
    admin_service --> object_storage
    catalog_service --> object_storage
    cart_service --> redis_cache_main
    order_service -.->|Kafka events| kafka_bus
    inventory_service -.->|stock.reserved| kafka_bus
    payment_service -.->|payment.captured| kafka_bus
    notification_service -.->|consume events| kafka_bus
    shipping_service -.->|consume events| kafka_bus
    search_service -.->|consume events| kafka_bus
    payment_service --> payment_gateway_ext
    payment_service --> fraud_api
    notification_service --> email_provider
    notification_service --> sms_provider
    shipping_service --> carrier_api
    auth_service -.-> prom_arch
    catalog_service -.-> prom_arch
    order_service -.-> prom_arch
    payment_service -.-> prom_arch
    inventory_service -.-> prom_arch
    auth_service -.-> otel_arch
    order_service -.-> otel_arch
    payment_service -.-> otel_arch
    web_bff -.-> elk_arch
    order_service -.-> elk_arch
    notification_service -.-> elk_arch
    prom_arch --> graf_arch
    recommendation_cache[Recommendation Cache]
    analytics_lake[Analytics Lakehouse]
    dead_letter_queue[Kafka DLQ / Retry Topics]
    auth_service --> redis_cache_main
    catalog_service --> redis_cache_main
    web_bff --> recommendation_cache
    search_service -.->|top search analytics| analytics_lake
    order_service -.->|checkout analytics| analytics_lake
    kafka_bus ==> dead_letter_queue
    kafka_bus ==> analytics_lake
```

**Key takeaways**
- The BFF coordinates customer-facing workflows, but business capabilities remain separate behind it for scaling and ownership clarity.
- PostgreSQL, MongoDB, Redis, Elasticsearch, Kafka, and object storage each support different data access patterns in the same system.
- Async messaging reduces coupling for notifications, shipping, and search updates while preserving strong write paths for orders and payments.

## Diagram 2 — Cloud VPC and Network Architecture

This network diagram maps the cloud landing zone for the ecommerce platform, including public, app, database, and management subnets inside a single VPC.
It also includes route tables, security layers, and VPN attachment so readers can see how the cloud environment connects to external users and on-prem infrastructure.

**When to use:** Use this when defining cloud network boundaries, subnet placement, and security controls for a production deployment.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_cloud[Internet] --> waf_cloud[WAF + DDoS Protection]
    waf_cloud --> alb_public[Public ALB]
    onprem_site[On-Prem DC] --> vpn_gateway[VPN / Direct Connect]
    vpn_gateway --> vpc_main[VPC 10.0.0.0/16]

    subgraph public_subnet[Public Subnet 10.0.1.0/24]
        alb_node[ALB nodes]
        nat_gw[NAT Gateway]
        bastion_cloud[Bastion Host]
        rt_public[Route Table Public]
        nacl_public[NACL Public]
    end

    subgraph app_subnet[Private App Subnet 10.0.10.0/24]
        k8s_workers[K8s Worker Nodes]
        ingress_nginx_cloud[Ingress Controller]
        app_sg[Security Group App]
        rt_app[Route Table App via NAT]
        nacl_app[NACL App]
    end

    subgraph db_subnet[Private DB Subnet 10.0.20.0/24]
        rds_primary[RDS PostgreSQL]
        rds_replica[RDS Read Replica]
        elasticache[ElastiCache Redis]
        opensearch[Managed Search Cluster]
        db_sg[Security Group DB]
        rt_db[Route Table DB isolated]
        nacl_db[NACL DB]
    end

    subgraph mgmt_subnet[Private Mgmt Subnet 10.0.30.0/24]
        jenkins_host[Jenkins / Runners]
        prom_cloud[Prometheus]
        graf_cloud[Grafana]
        vault_cloud[Vault / Secret Mgmt]
        mgmt_sg[Security Group Mgmt]
        rt_mgmt[Route Table Mgmt via NAT/VPN]
        nacl_mgmt[NACL Mgmt]
    end

    alb_public --> alb_node
    alb_node --> ingress_nginx_cloud
    ingress_nginx_cloud --> k8s_workers
    k8s_workers --> rds_primary
    k8s_workers --> elasticache
    k8s_workers -.-> opensearch
    rds_primary ==> rds_replica
    jenkins_host -.-> k8s_workers
    prom_cloud -.-> k8s_workers
    prom_cloud -.-> rds_primary
    graf_cloud --> prom_cloud
    vault_cloud -.-> k8s_workers
    vpc_main --> public_subnet
    vpc_main --> app_subnet
    vpc_main --> db_subnet
    vpc_main --> mgmt_subnet
    rt_public --> nat_gw
    rt_app --> nat_gw
    rt_mgmt --> vpn_gateway
    bastion_cloud -.-> jenkins_host
    bastion_cloud -.-> prom_cloud
    igw[Internet Gateway]
    flow_logs[VPC Flow Logs]
    private_endpoints[Private Endpoints / service endpoints]
    dns_resolver[Private DNS Resolver]
    igw --> alb_node
    nat_gw --> private_endpoints
    k8s_workers --> dns_resolver
    vpc_main -.-> flow_logs
    db_sg -.-> rds_primary
    mgmt_sg -.-> jenkins_host
```

**Key takeaways**
- Public exposure is limited to the ALB, bastion, and NAT path; application and data services remain in private subnets.
- Security groups model workload relationships while NACLs provide an additional subnet-level filter layer.
- VPN or Direct Connect provides hybrid continuity for migration or shared operations with on-prem systems.

## Diagram 3 — Order Placement End-to-End Flow

This sequence diagram follows the most critical business transaction in the platform: placing an order and confirming payment while protecting stock accuracy.
It also captures the failure path where payment is rejected and inventory reservations must be rolled back cleanly.

**When to use:** Use this during domain modeling, incident review, or checkout latency analysis because it makes service responsibilities and rollback points explicit.

```mermaid
%%{init:{"theme":"neutral"}}%%
sequenceDiagram
    autonumber
    participant user as User
    participant cdn as CDN
    participant alb as ALB
    participant apigw as API Gateway
    participant web as Web App BFF
    participant auth as Auth Service
    participant catalog as Product Catalog
    participant inventory as Inventory Service
    participant order as Order Service
    participant payment as Payment Service
    participant gateway as Stripe / Razorpay
    participant notify as Notification Service
    user->>cdn: Checkout submit
    cdn->>alb: HTTPS request
    alb->>apigw: Forward /checkout
    apigw->>web: Authorised route request
    web->>auth: Validate JWT + customer state
    auth-->>web: token valid + customer id
    web->>catalog: Get price / promo snapshot
    catalog-->>web: price + tax + discount data
    web->>inventory: Reserve stock lines
    inventory-->>web: reservation ids + expiry
    web->>order: Create pending order
    order-->>web: order_id + status=pending_payment
    web->>payment: Initiate payment for order
    payment->>gateway: Authorize / capture payment
    alt payment success
        gateway-->>payment: success txn_id
        payment-->>web: payment confirmed
        web->>order: Confirm order paid
        order-->>web: status=confirmed
        web-.->>notify: emit order.confirmed
        web->>inventory: Commit reservation to deduction
        inventory-->>web: stock deducted
        notify-->>user: Email + SMS confirmation
        web-->>user: Confirmation page
    else payment failed
        gateway-->>payment: decline / timeout
        payment-->>web: payment failed
        web->>inventory: Release reservation
        inventory-->>web: reservation released
        web->>order: Mark order payment_failed
        order-->>user: Failure response + retry option
    end
    Note over user,web: Typical happy-path latency target under 2 seconds
    Note right of inventory: reservation TTL 10 minutes
    Note right of payment: gateway SLA 300-800 ms dominates latency
    Note over web,order: idempotency keys protect user retries
    order-.->>notify: publish audit + confirmation workflow
```

**Key takeaways**
- Inventory reservation must happen before payment capture to prevent overselling during spikes.
- Order confirmation is only final after both payment success and inventory commitment are complete.
- Rollback paths should be modeled explicitly, because checkout failures are business-critical flows too.

## Diagram 4 — Data Flow Architecture (CQRS plus Event Sourcing)

This diagram separates write and read responsibilities so the platform can keep transactional correctness on the command side while optimizing search and dashboards on the query side.
It also shows how events leave the write database through CDC and feed downstream projections such as Elasticsearch and Redis-backed read models.

**When to use:** Use this when designing high-read ecommerce systems where order integrity and fast product discovery must coexist.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph LR
    client_api[Client API Request] --> command_api[Command API]
    command_api --> command_handler[Command Handler]
    command_handler --> domain_rules[Domain Validation + Aggregates]
    domain_rules --> write_db[PostgreSQL Write DB]
    write_db --> outbox_tbl[Outbox / Event Store]
    outbox_tbl -.-> debezium[Debezium CDC]
    debezium -.-> kafka_cqrs[Kafka Topics]
    kafka_cqrs -.-> projection_orders[Order Projection Service]
    kafka_cqrs -.-> projection_catalog[Catalog Projection Service]
    kafka_cqrs -.-> projection_customer[Customer Projection Service]
    projection_orders --> redis_read[Redis Read Cache]
    projection_catalog --> elastic_read[Elasticsearch Read Model]
    projection_customer --> mongo_read[MongoDB Read Model]
    client_query[Client Read Query] --> query_api[Query API]
    query_api --> query_handler[Query Handler]
    query_handler --> redis_read
    query_handler --> elastic_read
    query_handler --> mongo_read
    order_events[Order Event Stream] ==> event_archive[Immutable Event Archive]
    kafka_cqrs ==> order_events
    schema_registry[Schema Registry]
    analytics_sink[Analytics Warehouse]
    read_api_cache[Query API Response Cache]
    kafka_cqrs ==> schema_registry
    projection_orders --> analytics_sink
    projection_catalog --> analytics_sink
    query_handler --> read_api_cache
    read_api_cache --> redis_read
```

**Key takeaways**
- The command path prioritizes transactional correctness, while the query path prioritizes speed and tailored read models.
- CDC plus an outbox pattern avoids dual-write problems when publishing events from a relational write store.
- Event archives create a durable history that supports replay, audit, and new projection creation later.

## Diagram 5 — Cloud Security Architecture

This security architecture layers edge protection, cluster isolation, service identity, secret distribution, and encryption controls across the whole platform.
It connects IAM roles, Vault, External Secrets, mesh mTLS, and managed database encryption into one zero-trust oriented design.

**When to use:** Use this when documenting how cloud-native controls combine to protect internet-facing ecommerce workloads and sensitive payment-adjacent data.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_sec_cloud[Internet] --> waf_sec_cloud[WAF with OWASP rules]
    waf_sec_cloud --> ddos_guard[DDoS Protection]
    ddos_guard --> alb_tls[ALB TLS 1.3]
    alb_tls --> ingress_sec_cloud[K8s Ingress]

    subgraph cluster_sec[Cluster Security]
        netpols[Network Policies]
        pod_mesh[mTLS via Istio / Envoy]
        admission[Admission Policies / OPA]
        runtime_sec[Runtime Security / Falco]
    end

    subgraph secret_chain[Secret and Identity Flow]
        iam_root[IAM Role for Platform]
        vault_sec[Vault]
        ext_secrets[External Secrets Operator]
        k8s_secrets[K8s Secrets]
        pod_env[Pod Env / CSI Secret Mount]
    end

    subgraph data_sec[Data Protection]
        kms_keys[KMS Keys]
        rds_encrypt[Managed DB encrypted at rest]
        redis_encrypt[Managed Cache in-transit + at-rest]
        s3_encrypt[S3 / Blob SSE-KMS]
        cert_mgr_sec[Certificate Manager]
    end

    alb_tls --> ingress_sec_cloud
    ingress_sec_cloud --> netpols
    netpols --> pod_mesh
    pod_mesh --> admission
    admission --> runtime_sec
    iam_root --> vault_sec
    vault_sec --> ext_secrets
    ext_secrets --> k8s_secrets
    k8s_secrets --> pod_env
    kms_keys --> rds_encrypt
    kms_keys --> redis_encrypt
    kms_keys --> s3_encrypt
    cert_mgr_sec --> alb_tls
    cert_mgr_sec --> pod_mesh
    pod_env --> rds_encrypt
    pod_env --> redis_encrypt
    pod_env --> s3_encrypt
    image_signing[Image Signing / Provenance]
    audit_logs[Audit Logs / SIEM]
    workload_iam[IRSA / Workload Identity]
    security_hub[Security Hub]
    admission --> image_signing
    runtime_sec -.-> audit_logs
    iam_root --> workload_iam
    workload_iam --> pod_env
    audit_logs --> security_hub
    waf_sec_cloud -.-> audit_logs
```

**Key takeaways**
- Security is a chain from the edge to the pod, not a single WAF or firewall control.
- Secrets should move through managed operators and short-lived identity rather than manual secret injection.
- Encryption in transit and at rest must be paired with authorization boundaries such as IAM and network policy.

## Diagram 6 — Monitoring and Observability Stack

This observability view combines technical telemetry with business-facing dashboards so platform health and ecommerce outcomes are visible together.
Metrics, logs, traces, synthetic checks, and alert routing are connected as one operating system for the platform rather than isolated tools.

**When to use:** Use this when defining SLOs, dashboards, and incident response paths for production cloud workloads.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph svc_obs[All Services]
        web_obs[Web / BFF]
        order_obs[Order Service]
        payment_obs[Payment Service]
        inventory_obs[Inventory Service]
        notify_obs[Notification Service]
        ingress_obs[Ingress / Gateway]
    end

    subgraph metrics_obs[Metrics]
        prom_stack[Prometheus]
        custom_metrics[Custom business metrics]
        graf_dash[Grafana dashboards<br/>Orders/min Revenue Error rate P99 Cart abandonment]
    end

    subgraph logs_obs[Logs]
        fluentd[Fluentd / Fluent Bit]
        elastic_logs[Elasticsearch]
        kibana_logs[Kibana]
    end

    subgraph trace_obs[Tracing]
        otel_col[OpenTelemetry Collector]
        jaeger_obs[Jaeger]
    end

    subgraph alert_obs[Alerts and Uptime]
        alertmanager_obs[Alertmanager]
        pagerduty_obs[PagerDuty]
        slack_obs[Slack]
        email_obs[Email]
        synthetic_obs[Synthetic monitoring / uptime checks]
    end

    web_obs -.-> prom_stack
    order_obs -.-> prom_stack
    payment_obs -.-> prom_stack
    inventory_obs -.-> prom_stack
    ingress_obs -.-> prom_stack
    custom_metrics --> prom_stack
    prom_stack --> graf_dash
    prom_stack --> alertmanager_obs
    alertmanager_obs --> pagerduty_obs
    alertmanager_obs --> slack_obs
    alertmanager_obs --> email_obs
    web_obs --> fluentd
    order_obs --> fluentd
    payment_obs --> fluentd
    notify_obs --> fluentd
    fluentd --> elastic_logs
    elastic_logs --> kibana_logs
    web_obs -.-> otel_col
    order_obs -.-> otel_col
    payment_obs -.-> otel_col
    inventory_obs -.-> otel_col
    otel_col --> jaeger_obs
    synthetic_obs -.-> ingress_obs
    synthetic_obs -.-> web_obs
    slo_burn[SLO burn-rate alerts]
    blackbox_obs[Blackbox checkout probes]
    rum_obs[Real User Monitoring]
    synthetic_obs -.-> blackbox_obs
    blackbox_obs -.-> alertmanager_obs
    web_obs -.-> rum_obs
    rum_obs --> graf_dash
    jaeger_obs --> graf_dash
    prom_stack --> slo_burn
    slo_burn --> pagerduty_obs
```

**Key takeaways**
- Business metrics belong beside infrastructure metrics because incidents are often first seen as checkout or revenue anomalies.
- Tracing is essential for multi-service flows such as checkout where latency may span several internal and external calls.
- Synthetic checks catch customer-visible failures that can remain hidden when internal health endpoints still look green.

## Diagram 7 — On-Prem versus Cloud Architecture Comparison

This side-by-side comparison maps traditional on-prem components to their cloud-native equivalents so the migration target is easy to understand.
It makes the operational shift visible: from hardware-managed capacity and manual scaling to managed services, automation, and elastic infrastructure.

**When to use:** Use this when explaining migration rationale, target-state design, or platform operating model changes to mixed audiences.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph onprem_cmp[On-Prem]
        on_hw[Physical servers]
        on_lb[Hardware load balancer]
        on_scale[Manual scaling]
        on_storage[Local SAN / NAS]
        on_db[Self-managed MySQL/Postgres]
        on_mon[Local monitoring stack]
    end
    subgraph cloud_cmp[Cloud]
        cl_k8s[Managed Kubernetes]
        cl_alb[Managed ALB / API Gateway]
        cl_auto[HPA + Cluster Autoscaler]
        cl_obj[Object storage + managed volumes]
        cl_db[Managed DB / cache]
        cl_obs[Managed / cloud-integrated observability]
    end
    on_hw --> cl_k8s
    on_lb --> cl_alb
    on_scale --> cl_auto
    on_storage --> cl_obj
    on_db --> cl_db
    on_mon --> cl_obs
    subgraph onprem_ops[On-Prem Operations]
        on_fw[Perimeter firewalls]
        on_ci[Local Jenkins]
        on_secret[Manual secret distribution]
    end
    subgraph cloud_ops[Cloud Operations]
        cl_waf[Managed WAF / Shield]
        cl_gitops[GitOps + managed CI]
        cl_secret[Vault / workload identity]
    end
    on_fw --> cl_waf
    on_ci --> cl_gitops
    on_secret --> cl_secret
```

**Key takeaways**
- The biggest change is operational responsibility, not only runtime location.
- Cloud architecture replaces manual provisioning and appliance management with APIs, policies, and managed services.
- Mapping legacy components to cloud equivalents helps de-risk migration planning.

## Diagram 8 — 5-Wave Migration Timeline

This gantt chart breaks the modernization program into five waves so infrastructure, data, application, and cutover work can be sequenced realistically.
Parallel activities are shown inside each wave to reflect how platform and product workstreams typically overlap during migration.

**When to use:** Use this when coordinating teams, budgets, and delivery expectations across a multi-month migration effort.

```mermaid
%%{init:{"theme":"neutral"}}%%
gantt
    title Ecommerce Cloud Migration - 18 Week Plan
    dateFormat  YYYY-MM-DD
    axisFormat  %W
    section Wave 1 Foundation
    Landing zone, VPC, IAM, observability      :done,    w1a, 2026-01-05, 21d
    CI/CD baseline and registry               :done,    w1b, 2026-01-12, 14d
    Security controls and secrets foundation  :done,    w1c, 2026-01-15, 14d
    section Wave 2 Data Layer
    Managed DB provisioning                   :active,  w2a, 2026-02-02, 14d
    CDC and replication setup                 :active,  w2b, 2026-02-05, 14d
    Search and cache migration                :         w2c, 2026-02-10, 14d
    section Wave 3 Stateless Services
    Web BFF and auth migration                :         w3a, 2026-03-02, 14d
    Catalog, search, cart migration           :         w3b, 2026-03-05, 14d
    Observability tuning in staging           :         w3c, 2026-03-10, 10d
    section Wave 4 Stateful and Payments
    Orders and inventory migration            :         w4a, 2026-03-30, 14d
    Payments and notifications migration      :         w4b, 2026-04-02, 14d
    DR rehearsal and failover validation      :         w4c, 2026-04-08, 10d
    section Wave 5 Cutover
    Canary production traffic                 :         w5a, 2026-04-27, 7d
    Full cutover and rollback window          :         w5b, 2026-05-04, 7d
    Hypercare and optimization                :         w5c, 2026-05-11, 7d
    Platform policy as code                    :done,    w1d, 2026-01-20, 10d
    Data validation rehearsal                  :         w2d, 2026-02-18, 7d
    Frontend performance tuning                :         w3d, 2026-03-15, 7d
    Payment provider certification             :         w4d, 2026-04-10, 7d
    Executive go-live readiness                :         w5d, 2026-05-08, 5d
```

**Key takeaways**
- Foundation and data migration must start before stateless service movement can be low-risk.
- Stateful services and payments deserve a later wave because their failure cost is higher.
- Cutover should include both a canary phase and an explicit rollback window, not a single irreversible switch.

## Diagram 9 — Multi-Region DR Architecture

This diagram shows primary and disaster recovery regions with global traffic steering, replicated platform services, and cross-region data flows for all critical stateful components.
It makes the standby design and replication methods visible so teams can align failover procedures with actual platform topology.

**When to use:** Use this when you need a clear production DR reference that spans compute, databases, cache, and object storage across regions.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    route53[Route53 / Traffic Manager] --> region1_arch[Region 1 Primary]
    route53 -.->|health-check failover| region2_arch[Region 2 DR]

    subgraph primary_region_arch[Primary Region]
        k8s_primary_arch[Full K8s Cluster]
        rds_primary_arch[Managed PostgreSQL Primary]
        mongo_primary_arch[Managed Mongo Primary]
        redis_primary_arch[Redis Primary]
        search_primary_arch[OpenSearch Primary]
        blob_primary_arch[Primary Object Storage]
    end

    subgraph dr_region_arch[DR Region]
        k8s_dr_arch[Standby K8s Cluster]
        rds_dr_arch[Read Replica / promoted DB]
        mongo_dr_arch[Replica Set Secondary]
        redis_dr_arch[Redis Replica]
        search_dr_arch[Warm Search Cluster]
        blob_dr_arch[Replicated Object Storage]
    end

    subgraph dr_targets_arch[Recovery Targets]
        rpo_arch[RPO 5-15 min core data]
        rto_arch[RTO 30-60 min app failover]
    end

    region1_arch --> k8s_primary_arch
    region2_arch --> k8s_dr_arch
    k8s_primary_arch --> rds_primary_arch
    k8s_primary_arch --> mongo_primary_arch
    k8s_primary_arch --> redis_primary_arch
    k8s_primary_arch -.-> search_primary_arch
    k8s_primary_arch ==> blob_primary_arch
    rds_primary_arch ==> rds_dr_arch
    mongo_primary_arch ==> mongo_dr_arch
    redis_primary_arch ==> redis_dr_arch
    search_primary_arch ==> search_dr_arch
    blob_primary_arch ==> blob_dr_arch
    rds_dr_arch --> rpo_arch
    k8s_dr_arch --> rto_arch
    velero_primary_arch[Velero Backups]
    secret_replication[Global secret replication]
    dr_runbook[Failover Orchestrator]
    k8s_primary_arch -.-> velero_primary_arch
    velero_primary_arch ==> blob_dr_arch
    secret_replication ==> k8s_dr_arch
    route53 -.-> dr_runbook
    dr_runbook -.-> k8s_dr_arch
```

**Key takeaways**
- DR requires both replicated data and enough standby compute to actually serve traffic after promotion.
- Different data stores replicate differently, so a single DR statement rarely fits the whole platform.
- Health-checked global DNS is the final control point that makes recovery reachable by customers.

## Diagram 10 — Cost Architecture Where Money Goes

This pie chart gives a simplified view of cost concentration areas so platform decisions can be discussed in economic as well as technical terms.
It is intentionally high-level, making it useful for governance reviews, budget planning, and optimization prioritization.

**When to use:** Use this when communicating cost distribution to stakeholders deciding where optimization work will have the biggest impact.

```mermaid
%%{init:{"theme":"neutral"}}%%
pie
    title Ecommerce Platform Cost Distribution
    "Compute" : 40
    "Database" : 25
    "Storage" : 10
    "Network/CDN" : 10
    "Monitoring" : 5
    "Security" : 5
    "Other" : 5
```

**Key takeaways**
- Compute and database spend usually dominate the bill for a multi-service ecommerce platform.
- Storage and network costs look smaller individually but grow quickly with search, backups, media, and cross-region traffic.
- A cost diagram helps teams prioritize autoscaling, data retention, and architecture simplification work.

## Diagram 11 — Technology Decision Tree

This decision tree turns common architecture choices into explicit branching logic so the team can explain why particular technologies fit specific problem shapes.
It covers persistence, messaging, ingress, and orchestration choices rather than presenting one stack as universally correct.

**When to use:** Use this when standardizing technology selection or reviewing whether a new workload should follow the platform defaults.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TD
    start_choice[New workload or capability?] --> consistency_q{Need strong consistency?}
    consistency_q -->|Yes| postgres_choice[Choose PostgreSQL]
    consistency_q -->|No| schema_q{Need flexible schema?}
    schema_q -->|Yes| mongo_choice[Choose MongoDB]
    schema_q -->|No| kv_q{Need key-value cache or sessions?}
    kv_q -->|Yes| redis_choice[Choose Redis]
    kv_q -->|No| search_q{Need full-text / faceted search?}
    search_q -->|Yes| elastic_choice[Choose Elasticsearch / OpenSearch]
    search_q -->|No| object_q{Need large binary/object storage?}
    object_q -->|Yes| object_choice[Choose S3 / Blob Storage]
    object_q -->|No| messaging_q{Need asynchronous messaging?}
    messaging_q -->|Yes| throughput_q{High throughput event stream?}
    throughput_q -->|Yes| kafka_choice[Choose Kafka]
    throughput_q -->|No| rabbit_choice[Choose RabbitMQ]
    messaging_q -->|No| ingress_q{Public HTTP traffic?}
    ingress_q -->|Yes| ingress_choice[Choose API Gateway + Ingress]
    ingress_q -->|No| orchestration_q{Need multi-service scheduling?}
    orchestration_q -->|Yes| k8s_choice[Choose Kubernetes]
    orchestration_q -->|No| vm_choice[Choose VM / simple container host]
    ingress_choice --> mesh_q{Need service-to-service policy and mTLS?}
    mesh_q -->|Yes| istio_choice[Choose Istio service mesh]
    mesh_q -->|No| nginx_choice[Choose Nginx Ingress only]
    kafka_choice --> ordering_q{Need ordered retries and replay?}
    ordering_q -->|Yes| kafka_choice_final[Kafka remains fit]
    ordering_q -->|No| rabbit_choice_alt[RabbitMQ simpler fit]
    vm_choice --> packaging_q{Need immutable releases soon?}
    packaging_q -->|Yes| container_host[Use Docker on VMs]
```

**Key takeaways**
- Decision trees are useful because they encode platform rationale, not just preferences.
- Different storage and messaging systems are selected for different access and consistency needs.
- Standard branching logic reduces architecture drift across teams.

## Diagram 12 — Deployment Strategy Decision

This decision graph explains when rolling, canary, or blue-green deployment strategies are appropriate and how traffic moves during each one.
It links release criticality and blast radius to the actual deployment pattern instead of leaving rollout choice to ad hoc judgment.

**When to use:** Use this when defining release policy for different classes of services such as payments, customer-facing web, and major version upgrades.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TD
    deploy_start[New release ready] --> critical_q{Is it payments or business-critical?}
    critical_q -->|Yes| canary_path[Canary deployment<br/>1% -> 5% -> 25% -> 100%]
    critical_q -->|No| stateless_q{Is it stateless web or API?}
    stateless_q -->|Yes| rolling_path[Rolling update<br/>maxUnavailable 1 / maxSurge 1]
    stateless_q -->|No| major_q{Is it major version or schema-risky?}
    major_q -->|Yes| bluegreen_path[Blue-Green deployment<br/>full parallel environment]
    major_q -->|No| rolling_path
    canary_path --> traffic_canary[Traffic split via mesh / ingress]
    rolling_path --> traffic_rolling[Pod-by-pod replacement]
    bluegreen_path --> traffic_bluegreen[Switch all traffic after validation]
    traffic_canary --> observe_canary[Observe error rate latency business KPIs]
    traffic_rolling --> observe_roll[Observe readiness and saturation]
    traffic_bluegreen --> observe_bg[Run smoke tests on green before cutover]
    observe_canary --> decision_canary{Healthy?}
    observe_roll --> decision_roll{Healthy?}
    observe_bg --> decision_bg{Healthy?}
    decision_canary -->|Yes| promote_canary[Promote to 100%]
    decision_canary -->|No| rollback_canary[Rollback to previous stable]
    decision_roll -->|Yes| finish_roll[Finish rollout]
    decision_roll -->|No| rollback_roll[Pause and rollback]
    decision_bg -->|Yes| cutover_bg[Switch router to green]
    decision_bg -->|No| keep_blue[Keep blue live and fix green]
    promote_canary --> post_canary[Post-canary verification window]
    cutover_bg --> post_bg[Green soak monitoring]
    finish_roll --> post_roll[Capacity and error check]
    rollback_canary --> incident_review[Incident review + fix forward]
    rollback_roll --> incident_review
    keep_blue --> incident_review
    post_canary --> close_release[Close release]
    post_bg --> close_release
    post_roll --> close_release
```

**Key takeaways**
- Criticality and change risk should drive rollout strategy selection, not habit alone.
- Canary is ideal for high-risk but live-traffic-tolerant services, while blue-green suits large compatibility jumps.
- Every strategy needs an explicit observation and rollback step to be operationally safe.
