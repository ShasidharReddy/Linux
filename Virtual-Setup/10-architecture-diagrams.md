# 10 Architecture Diagrams

Complete Visual Architecture Reference for Virtual, Container, and Kubernetes Ecommerce Setup

This guide complements the virtualization, Docker, and Kubernetes setup documents with detailed visual references that can be used during planning, implementation, and incident response.
The focus is on Mermaid diagrams first: large topologies, service relationships, scaling paths, and migration views, with only enough prose to explain intent and usage.

---

## Diagram 1 — KVM/VM-Based Ecommerce Architecture

This diagram shows an ecommerce platform split across KVM guests, with clear separation of frontend, backend, and data networks on top of a physical virtualization host layer.
Resource sizing is included directly in the topology so architects can see how compute, memory, storage, and network segmentation line up at the VM level.

**When to use:** Use this when modernizing a physical deployment into virtual machines while preserving clear tier boundaries and predictable resource reservations.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph host_cluster[Physical KVM Hosts]
        kvm_host_01[Host-01 32 vCPU / 128 GB RAM / 4 TB SSD]
        kvm_host_02[Host-02 32 vCPU / 128 GB RAM / 4 TB SSD]
        kvm_host_03[Host-03 24 vCPU / 96 GB RAM / 3 TB SSD]
        libvirt_mgr[libvirt + virt-manager + cloud-init]
    end

    subgraph networks[Virtual Bridges]
        br_frontend[br-frontend 192.168.10.0/24]
        br_backend[br-backend 192.168.20.0/24]
        br_data[br-data 192.168.30.0/24]
        br_mgmt[br-mgmt 192.168.40.0/24]
    end

    subgraph vm_tier[VM Inventory]
        haproxy_vm[HAProxy-VM<br/>2 vCPU / 4 GB / 40 GB]
        web_vm_01[Web-VM-01<br/>4 vCPU / 8 GB / 80 GB]
        web_vm_02[Web-VM-02<br/>4 vCPU / 8 GB / 80 GB]
        app_vm_01[App-VM-01<br/>6 vCPU / 12 GB / 120 GB]
        app_vm_02[App-VM-02<br/>6 vCPU / 12 GB / 120 GB]
        db_vm_pri[DB-Primary-VM<br/>8 vCPU / 32 GB / 500 GB]
        db_vm_rep[DB-Replica-VM<br/>8 vCPU / 32 GB / 500 GB]
        redis_vm[Redis-VM<br/>4 vCPU / 8 GB / 60 GB]
        utility_vm[Utility-VM Monitoring/CI<br/>4 vCPU / 8 GB / 100 GB]
    end

    kvm_host_01 --> br_frontend
    kvm_host_01 --> br_backend
    kvm_host_01 --> br_data
    kvm_host_01 --> br_mgmt
    kvm_host_02 --> br_frontend
    kvm_host_02 --> br_backend
    kvm_host_02 --> br_data
    kvm_host_02 --> br_mgmt
    kvm_host_03 --> br_frontend
    kvm_host_03 --> br_backend
    kvm_host_03 --> br_data
    kvm_host_03 --> br_mgmt
    libvirt_mgr -.-> kvm_host_01
    libvirt_mgr -.-> kvm_host_02
    libvirt_mgr -.-> kvm_host_03
    br_frontend --> haproxy_vm
    br_frontend --> web_vm_01
    br_frontend --> web_vm_02
    br_backend --> web_vm_01
    br_backend --> web_vm_02
    br_backend --> app_vm_01
    br_backend --> app_vm_02
    br_data --> db_vm_pri
    br_data --> db_vm_rep
    br_data --> redis_vm
    br_backend --> utility_vm
    br_mgmt --> haproxy_vm
    br_mgmt --> web_vm_01
    br_mgmt --> web_vm_02
    br_mgmt --> app_vm_01
    br_mgmt --> app_vm_02
    br_mgmt --> db_vm_pri
    br_mgmt --> db_vm_rep
    br_mgmt --> redis_vm
    br_mgmt --> utility_vm
    haproxy_vm --> web_vm_01
    haproxy_vm --> web_vm_02
    web_vm_01 --> app_vm_01
    web_vm_01 --> app_vm_02
    web_vm_02 --> app_vm_01
    web_vm_02 --> app_vm_02
    app_vm_01 --> db_vm_pri
    app_vm_02 --> db_vm_pri
    app_vm_01 --> redis_vm
    app_vm_02 --> redis_vm
    db_vm_pri ==> db_vm_rep
```

**Key takeaways**
- KVM bridges model the same trust boundaries as physical VLANs while preserving VM mobility and resource overcommit controls.
- Database guests receive the heaviest RAM and disk allocations because transactional workloads dominate resource consumption.
- A management bridge keeps operator tooling separate from customer traffic and data traffic.

## Diagram 2 — Docker Compose Architecture (Development)

This development-oriented stack shows a local or single-host Compose deployment with realistic supporting services for an ecommerce application.
It keeps the topology simple enough for developer laptops or test VMs while still reflecting service boundaries, ports, networks, and persisted volumes.

**When to use:** Use this for local development, QA sandboxes, or small integration environments where ease of startup matters more than production-grade failover.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    developer[Developer Browser / Postman] --> compose_entry[docker-compose up]

    subgraph frontend_net[frontend network]
        nginx_ctr[nginx container :80/:443]
        mailhog_ctr[mailhog :1025/:8025]
    end

    subgraph backend_net[backend network]
        php_ctr[php-fpm container :9000]
        worker_ctr[queue-worker container]
        scheduler_ctr[scheduler container]
    end

    subgraph data_net[data network]
        mysql_ctr[mysql :3306]
        redis_ctr[redis :6379]
        es_ctr[elasticsearch :9200/:9300]
        rabbit_ctr[rabbitmq :5672/:15672]
        adminer_ctr[adminer :8080]
    end

    subgraph volumes[Named Volumes]
        mysql_vol[mysql-data]
        redis_vol[redis-data]
        es_vol[es-data]
        uploads_vol[uploads]
        logs_vol[app-logs]
    end

    compose_entry --> nginx_ctr
    compose_entry --> php_ctr
    compose_entry --> worker_ctr
    compose_entry --> scheduler_ctr
    compose_entry --> mysql_ctr
    compose_entry --> redis_ctr
    compose_entry --> es_ctr
    compose_entry --> rabbit_ctr
    compose_entry --> adminer_ctr
    developer -->|HTTP 80/443| nginx_ctr
    developer -->|DB GUI 8080| adminer_ctr
    developer -->|Mail UI 8025| mailhog_ctr
    nginx_ctr --> php_ctr
    php_ctr --> mysql_ctr
    php_ctr --> redis_ctr
    php_ctr -.-> rabbit_ctr
    php_ctr -.-> es_ctr
    worker_ctr --> mysql_ctr
    worker_ctr --> redis_ctr
    worker_ctr -.-> rabbit_ctr
    scheduler_ctr --> mysql_ctr
    mysql_ctr ==> mysql_vol
    redis_ctr ==> redis_vol
    es_ctr ==> es_vol
    nginx_ctr ==> uploads_vol
    php_ctr ==> uploads_vol
    php_ctr ==> logs_vol
    worker_ctr ==> logs_vol
```

**Key takeaways**
- Compose networks emulate frontend, backend, and data isolation without requiring a full orchestrator.
- Named volumes preserve state across container restarts, which matters even in development for realistic debugging.
- Supporting tools like Mailhog and Adminer reduce environment friction while keeping the core topology recognizable.

## Diagram 3 — Docker Compose Production with Traefik

This diagram expands the containerized model into a production-style deployment using Traefik, multiple replicas, clustered dependencies, and health-aware routing.
It frames Docker Swarm or Compose-on-multiple-hosts as a bridge step before Kubernetes, especially for teams adopting containers incrementally.

**When to use:** Use this when the team wants container scheduling and SSL automation before investing in the operational complexity of a full Kubernetes platform.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_prod[Internet] --> traefik_edge[Traefik Edge<br/>Let's Encrypt + dashboard]

    subgraph overlay_front[Swarm Overlay - frontend]
        web_ctr_1[web-1 replica]
        web_ctr_2[web-2 replica]
        web_ctr_3[web-3 replica]
    end

    subgraph overlay_back[Swarm Overlay - backend]
        app_ctr_1[app-1 replica]
        app_ctr_2[app-2 replica]
        app_ctr_3[app-3 replica]
        worker_ctr_1[worker-1]
        worker_ctr_2[worker-2]
    end

    subgraph overlay_data[Swarm Overlay - data]
        mysql_pri_ctr[mysql-primary]
        mysql_rep_ctr[mysql-replica]
        redis_m_ctr[redis-master]
        redis_s_ctr[redis-slave]
        es1_ctr[es-1]
        es2_ctr[es-2]
        es3_ctr[es-3]
        rabbit1_ctr[rabbit-1]
        rabbit2_ctr[rabbit-2]
        rabbit3_ctr[rabbit-3]
    end

    subgraph ops[Operations]
        hc_engine[Docker health checks]
        registry_prod[Private Image Registry]
        watchtower[Update Automation / GitOps bridge]
    end

    traefik_edge -->|router websecure| web_ctr_1
    traefik_edge --> web_ctr_2
    traefik_edge --> web_ctr_3
    web_ctr_1 --> app_ctr_1
    web_ctr_1 --> app_ctr_2
    web_ctr_2 --> app_ctr_2
    web_ctr_2 --> app_ctr_3
    web_ctr_3 --> app_ctr_1
    web_ctr_3 --> app_ctr_3
    app_ctr_1 --> mysql_pri_ctr
    app_ctr_2 --> mysql_pri_ctr
    app_ctr_3 --> mysql_rep_ctr
    app_ctr_1 --> redis_m_ctr
    app_ctr_2 --> redis_m_ctr
    worker_ctr_1 -.-> rabbit1_ctr
    worker_ctr_2 -.-> rabbit2_ctr
    app_ctr_1 -.-> es1_ctr
    app_ctr_2 -.-> es2_ctr
    app_ctr_3 -.-> es3_ctr
    mysql_pri_ctr ==> mysql_rep_ctr
    redis_m_ctr ==> redis_s_ctr
    rabbit1_ctr ==> rabbit2_ctr
    rabbit2_ctr ==> rabbit3_ctr
    es1_ctr ==> es2_ctr
    es2_ctr ==> es3_ctr
    hc_engine -.-> web_ctr_1
    hc_engine -.-> web_ctr_2
    hc_engine -.-> web_ctr_3
    hc_engine -.-> app_ctr_1
    hc_engine -.-> app_ctr_2
    hc_engine -.-> app_ctr_3
    registry_prod -.-> watchtower
    watchtower -.-> traefik_edge
    watchtower -.-> app_ctr_1
    watchtower -.-> app_ctr_2
    watchtower -.-> app_ctr_3
```

**Key takeaways**
- Traefik centralizes ingress, certificate management, and dynamic service discovery for container replicas.
- Clustered stateful services remain possible, but they demand careful volume and placement planning compared with stateless containers.
- This architecture is often a practical transition point between simple Compose and Kubernetes.

## Diagram 4 — Kubernetes Cluster Architecture

This cluster-level view separates the control plane from worker pools and shows how infrastructure services coexist with ecommerce workloads.
The node pools are grouped by operational intent so readers can see which workloads belong on system, web, app, and data nodes.

**When to use:** Use this for platform design conversations where the question is how to shape the cluster, not just how to deploy one application.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    api_clients[Public Traffic] --> ingress_edge[Ingress Controller Service / External LB]

    subgraph control_plane[Control Plane x3]
        master1[Master-01<br/>API server + etcd + scheduler + controller-manager]
        master2[Master-02<br/>API server + etcd + scheduler + controller-manager]
        master3[Master-03<br/>API server + etcd + scheduler + controller-manager]
    end

    subgraph system_pool[System Pool]
        sys_node1[System Node-1]
        sys_node2[System Node-2]
        coredns[CoreDNS]
        kubeproxy[kube-proxy]
        cni[Calico / Cilium CNI]
        metrics_server[Metrics Server]
    end

    subgraph web_pool[Web Pool]
        web_node1[Web Node-1]
        web_node2[Web Node-2]
        nginx_ingress[Nginx Ingress Pods]
        frontend_pods[Frontend / Nginx Pods]
    end

    subgraph app_pool[App Pool]
        app_node1[App Node-1]
        app_node2[App Node-2]
        backend_pods[Backend API Pods]
        worker_pods[Worker Pods]
        cronjobs[Scheduled CronJobs]
    end

    subgraph data_pool[Data Pool]
        data_node1[Data Node-1]
        data_node2[Data Node-2]
        postgres_ss[PostgreSQL StatefulSet]
        redis_ss[Redis StatefulSet]
        elastic_ss[Elasticsearch StatefulSet]
    end

    ingress_edge --> nginx_ingress
    master1 --> sys_node1
    master1 --> web_node1
    master2 --> app_node1
    master2 --> data_node1
    master3 --> sys_node2
    master3 --> web_node2
    sys_node1 --> coredns
    sys_node1 --> kubeproxy
    sys_node2 --> cni
    sys_node2 --> metrics_server
    web_node1 --> frontend_pods
    web_node2 --> frontend_pods
    app_node1 --> backend_pods
    app_node2 --> backend_pods
    app_node1 --> worker_pods
    app_node2 --> cronjobs
    data_node1 --> postgres_ss
    data_node1 --> redis_ss
    data_node2 --> elastic_ss
    nginx_ingress --> frontend_pods
    frontend_pods --> backend_pods
    backend_pods --> postgres_ss
    backend_pods --> redis_ss
    backend_pods -.-> elastic_ss
```

**Key takeaways**
- Pool-based scheduling makes it easier to isolate noisy neighbors, security controls, and maintenance windows.
- Infrastructure add-ons such as DNS, proxying, metrics, and CNI are platform dependencies and belong in the system layer, not application namespaces.
- StatefulSets should be treated differently from web and app Deployments because storage and pod identity matter.

## Diagram 5 — Kubernetes Ecommerce Complete Deployment

This is the application-centric deployment map showing the full ecommerce workload as Kubernetes resources across production, monitoring, and ingress namespaces.
Replica counts, autoscaling ranges, resource envelopes, and PVC usage are included so platform and application teams can reason about scheduling and scaling together.

**When to use:** Use this when you want a single-page reference for what runs in the cluster, how much of it runs, and which workloads are stateful versus stateless.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph ingress_ns[Namespace: ingress-system]
        ext_lb[External LoadBalancer Service]
        ingress_ctrl[Ingress Controller Deployment x3<br/>HPA 3-6<br/>500m/1CPU 512Mi/1Gi]
        cert_mgr[cert-manager Deployment x2]
    end

    subgraph prod_ns[Namespace: ecommerce-prod]
        web_svc[web Service ClusterIP]
        web_dep[web Deployment replicas 4<br/>HPA 4-12<br/>250m/1CPU 256Mi/1Gi]
        bff_svc[bff Service ClusterIP]
        bff_dep[bff Deployment replicas 3<br/>HPA 3-10<br/>500m/2CPU 512Mi/2Gi]
        auth_dep[auth Deployment replicas 3<br/>HPA 3-8<br/>300m/1CPU 256Mi/1Gi]
        catalog_dep[catalog Deployment replicas 4<br/>HPA 4-10<br/>500m/2CPU 512Mi/2Gi]
        cart_dep[cart Deployment replicas 3<br/>HPA 3-8<br/>300m/1CPU 256Mi/1Gi]
        order_dep[order Deployment replicas 4<br/>HPA 4-12<br/>600m/2CPU 512Mi/2Gi]
        inventory_dep[inventory Deployment replicas 3<br/>HPA 3-9<br/>400m/2CPU 512Mi/2Gi]
        payment_dep[payment Deployment replicas 2<br/>HPA 2-6<br/>500m/2CPU 512Mi/1Gi]
        notification_dep[notification Deployment replicas 2<br/>HPA 2-6<br/>300m/1CPU 256Mi/512Mi]
        search_dep[search Deployment replicas 2<br/>HPA 2-5<br/>400m/1CPU 512Mi/1Gi]
        worker_dep[worker Deployment replicas 3<br/>KEDA 3-20<br/>500m/2CPU 512Mi/2Gi]
        postgres_ss_prod[postgres StatefulSet x3 + PVC 500Gi premium-ssd]
        redis_ss_prod[redis StatefulSet x3 + PVC 50Gi premium-ssd]
        elastic_ss_prod[elasticsearch StatefulSet x3 + PVC 200Gi standard-hdd]
        rabbit_ss_prod[rabbitmq StatefulSet x3 + PVC 50Gi premium-ssd]
        object_pvc[shared uploads PVC 200Gi nfs-shared]
    end

    subgraph monitoring_ns[Namespace: monitoring]
        prom_dep[Prometheus StatefulSet x2]
        graf_dep[Grafana Deployment x2]
        loki_dep[Loki StatefulSet x2]
        alert_dep[Alertmanager x2]
    end

    ext_lb --> ingress_ctrl
    ingress_ctrl --> cert_mgr
    ingress_ctrl --> web_svc
    web_svc --> web_dep
    web_dep --> bff_svc
    bff_svc --> bff_dep
    bff_dep --> auth_dep
    bff_dep --> catalog_dep
    bff_dep --> cart_dep
    bff_dep --> order_dep
    bff_dep --> search_dep
    order_dep --> inventory_dep
    order_dep --> payment_dep
    order_dep -.-> notification_dep
    catalog_dep -.-> elastic_ss_prod
    bff_dep --> redis_ss_prod
    auth_dep --> postgres_ss_prod
    cart_dep --> redis_ss_prod
    order_dep --> postgres_ss_prod
    inventory_dep --> postgres_ss_prod
    payment_dep --> postgres_ss_prod
    notification_dep -.-> rabbit_ss_prod
    worker_dep -.-> rabbit_ss_prod
    web_dep ==> object_pvc
    bff_dep ==> object_pvc
    prom_dep -.-> web_dep
    prom_dep -.-> bff_dep
    prom_dep -.-> order_dep
    prom_dep -.-> payment_dep
    graf_dep --> prom_dep
    alert_dep --> prom_dep
    loki_dep -.-> web_dep
    loki_dep -.-> bff_dep
```

**Key takeaways**
- Replica counts and HPA ranges should be documented near the service map so scaling expectations are explicit.
- Stateful workloads carry PVC dependencies that materially affect placement, backup, and recovery design.
- Separating ingress and monitoring namespaces reduces blast radius and improves operational ownership boundaries.

## Diagram 6 — Kubernetes Networking Deep Dive

This network-focused diagram explains how external traffic enters the cluster, how services resolve and route internally, and how policy boundaries constrain pod-to-pod communication.
It also layers in service mesh sidecars and DNS resolution so both platform engineers and application developers can trace a request fully.

**When to use:** Use this for debugging connectivity, service discovery, policy denials, or latency introduced by ingress, mesh, or CNI behavior.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    external_user[External User] --> cloud_lb[Cloud Load Balancer]
    cloud_lb --> nginx_ingress_net[Nginx Ingress Controller]

    subgraph service_layer[Service Networking]
        web_svc_net[web.ecommerce-prod.svc.cluster.local]
        api_svc_net[api.ecommerce-prod.svc.cluster.local]
        order_svc_net[order.ecommerce-prod.svc.cluster.local]
        redis_svc_net[redis.ecommerce-prod.svc.cluster.local]
        coredns_net[CoreDNS Service]
    end

    subgraph pod_layer[Pods with Calico / Envoy]
        web_pod_a[web pod A + Envoy]
        web_pod_b[web pod B + Envoy]
        api_pod_a[api pod A + Envoy]
        api_pod_b[api pod B + Envoy]
        order_pod_a[order pod A + Envoy]
        order_pod_b[order pod B + Envoy]
        redis_pod[redis pod]
        calico[Calico CNI dataplane]
    end

    subgraph policy_layer[Network Policies]
        ingress_policy[Allow ingress -> web only]
        api_policy[Allow web -> api 8080]
        order_policy[Allow api -> order 8080]
        redis_policy[Allow order/api -> redis 6379]
        deny_all[Default deny east-west]
    end

    nginx_ingress_net --> web_svc_net
    web_svc_net --> web_pod_a
    web_svc_net --> web_pod_b
    web_pod_a -->|HTTP 8080| api_svc_net
    web_pod_b -->|HTTP 8080| api_svc_net
    api_svc_net --> api_pod_a
    api_svc_net --> api_pod_b
    api_pod_a -->|gRPC 9090| order_svc_net
    api_pod_b -->|gRPC 9090| order_svc_net
    order_svc_net --> order_pod_a
    order_svc_net --> order_pod_b
    order_pod_a --> redis_svc_net
    order_pod_b --> redis_svc_net
    redis_svc_net --> redis_pod
    web_pod_a --> coredns_net
    api_pod_a --> coredns_net
    order_pod_a --> coredns_net
    coredns_net -.->|service-name.namespace.svc.cluster.local| api_svc_net
    coredns_net -.->|service-name.namespace.svc.cluster.local| order_svc_net
    coredns_net -.->|service-name.namespace.svc.cluster.local| redis_svc_net
    calico --> web_pod_a
    calico --> api_pod_a
    calico --> order_pod_a
    ingress_policy -.-> web_svc_net
    api_policy -.-> api_svc_net
    order_policy -.-> order_svc_net
    redis_policy -.-> redis_svc_net
    deny_all -.-> calico
```

**Key takeaways**
- Cluster networking is a combination of ingress, service abstraction, DNS, and CNI enforcement—not a single component.
- Default-deny policy plus explicit allow rules makes pod communication predictable and auditable.
- Envoy sidecars add observability and policy power, but they also add network hops that should be understood during debugging.

## Diagram 7 — CI/CD Pipeline to Kubernetes

This pipeline view connects source control to build, test, scan, packaging, GitOps sync, and progressive deployment across environments.
It is intentionally left-to-right so release managers can read it as a promotion path from developer commit to production rollout.

**When to use:** Use this when defining delivery controls, approval gates, and how Helm or GitOps artifacts move between environments.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph LR
    developer_ci[Developer] --> git_push[Git push / PR]
    git_push --> gh_actions[GitHub Actions or Jenkins]
    gh_actions --> build_step[Build image + unit tests]
    build_step --> integ_step[Integration + contract tests]
    integ_step --> scan_step[Trivy / SAST / dependency scan]
    scan_step --> registry_ci[Container Registry]
    registry_ci --> helm_pkg[Helm chart update]
    helm_pkg --> gitops_repo[GitOps config repo]
    gitops_repo --> argocd[ArgoCD detect drift + sync]
    argocd --> dev_env[Dev Cluster Namespace]
    dev_env --> qa_gate[QA / smoke tests]
    qa_gate --> staging_env[Staging Cluster Namespace]
    staging_env --> perf_gate[Performance + UAT gate]
    perf_gate --> approval_gate[Manual approval for prod]
    approval_gate --> prod_env[Production Cluster]
    prod_env --> rollout[Rolling update / canary]
    rollout --> verify[Post-deploy checks + rollback guard]
    gh_actions -.-> notify_chat[Slack / Teams notifications]
    scan_step -.-> vuln_ticket[Security findings / ticket]
    argocd -.-> drift_alert[Drift alerting]
```

**Key takeaways**
- The image build path and the environment promotion path should both be visible, because they are controlled by different tools and teams.
- Security scanning belongs before registry promotion so vulnerable images never become the default deploy artifact.
- GitOps turns cluster state into code, enabling auditability and safer rollback behavior.

## Diagram 8 — Production Kubernetes with Service Mesh (Istio)

This diagram adds Istio service mesh capabilities on top of the ecommerce deployment so routing, mTLS, telemetry, and canary releases are visible at pod level.
It shows how sidecars, ingress gateway, and mesh observability tools work together around normal application traffic.

**When to use:** Use this when you need fine-grained traffic policy, zero-trust service-to-service security, or progressive delivery beyond native ingress rules alone.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_mesh[Internet] --> istio_ingress[Istio Ingress Gateway]
    istio_ingress --> virtual_service[VirtualService orders.example.com]
    virtual_service -->|90% stable| stable_web[web-v1 pods + Envoy]
    virtual_service -.->|10% canary| canary_web[web-v2 pods + Envoy]
    stable_web --> bff_mesh[bff pods + Envoy]
    canary_web --> bff_mesh
    bff_mesh --> auth_mesh[auth pods + Envoy]
    bff_mesh --> catalog_mesh[catalog pods + Envoy]
    bff_mesh --> order_mesh[order pods + Envoy]
    order_mesh --> inventory_mesh[inventory pods + Envoy]
    order_mesh --> payment_mesh[payment pods + Envoy]
    order_mesh -.-> notification_mesh[notification pods + Envoy]
    auth_mesh --> redis_mesh[redis StatefulSet]
    order_mesh --> postgres_mesh[postgres StatefulSet]
    catalog_mesh -.-> elastic_mesh[elasticsearch StatefulSet]
    stable_web -.->|mTLS| bff_mesh
    bff_mesh -.->|mTLS| auth_mesh
    bff_mesh -.->|mTLS| catalog_mesh
    bff_mesh -.->|mTLS| order_mesh
    order_mesh -.->|mTLS| payment_mesh
    order_mesh -.->|mTLS| inventory_mesh
    subgraph mesh_ops[Mesh Observability]
        kiali[Kiali]
        jaeger_mesh[Jaeger]
        prom_mesh[Prometheus scrape sidecars]
        istiod[Istiod control plane]
    end
    istiod -.-> stable_web
    istiod -.-> canary_web
    istiod -.-> bff_mesh
    istiod -.-> order_mesh
    prom_mesh -.-> stable_web
    prom_mesh -.-> bff_mesh
    prom_mesh -.-> order_mesh
    jaeger_mesh -.-> bff_mesh
    jaeger_mesh -.-> payment_mesh
    kiali --> istio_ingress
    kiali --> virtual_service
```

**Key takeaways**
- Service mesh makes traffic policy and service identity explicit, which is especially valuable for critical checkout paths.
- Canary percentage splits belong in architecture references because rollout policy is part of production behavior, not only deployment automation.
- Kiali, Jaeger, and Prometheus provide the feedback loop needed to use mTLS and routing rules safely.

## Diagram 9 — Kubernetes Storage Architecture

This storage diagram ties StorageClasses to the workloads that consume them so persistence decisions are visible alongside the services that depend on them.
It also shows backup flow through Velero to object storage, which matters for cluster-level recovery and migration plans.

**When to use:** Use this when deciding which storage profile each service should use and how persistent state will be protected across upgrades or failures.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph storage_classes[StorageClasses]
        premium_ssd[premium-ssd<br/>low latency databases]
        standard_hdd[standard-hdd<br/>search and warm data]
        nfs_shared[nfs-shared<br/>RWX shared files]
        azure_file[azure-file / smb<br/>cross-node file share]
    end

    subgraph pv_layer[PersistentVolumes]
        pv_postgres[PV postgres-500Gi]
        pv_redis[PV redis-50Gi]
        pv_elastic[PV elastic-200Gi]
        pv_rabbit[PV rabbit-50Gi]
        pv_uploads[PV uploads-200Gi]
        pv_reports[PV reports-100Gi]
    end

    subgraph pvc_layer[PersistentVolumeClaims]
        pvc_postgres[postgres-data PVC]
        pvc_redis[redis-data PVC]
        pvc_elastic[elastic-data PVC]
        pvc_rabbit[rabbit-data PVC]
        pvc_uploads[uploads-shared PVC]
        pvc_reports[reports-share PVC]
    end

    subgraph workloads_storage[Pods / StatefulSets]
        postgres_pods_storage[PostgreSQL StatefulSet<br/>needs low latency writes]
        redis_pods_storage[Redis StatefulSet<br/>needs fast persistence]
        elastic_pods_storage[Elasticsearch StatefulSet<br/>needs capacity + throughput]
        rabbit_pods_storage[RabbitMQ StatefulSet<br/>needs durable queue storage]
        web_pods_storage[Web / BFF Pods<br/>need shared uploads]
        report_pods_storage[Report Jobs<br/>need shared export area]
    end

    subgraph backup_storage[Backup Flow]
        velero[Velero]
        snapshots[CSI snapshots]
        object_store[Object Storage Backup Bucket]
        restore_cluster[Restore Target Cluster]
    end

    premium_ssd --> pv_postgres
    premium_ssd --> pv_redis
    premium_ssd --> pv_rabbit
    standard_hdd --> pv_elastic
    nfs_shared --> pv_uploads
    azure_file --> pv_reports
    pv_postgres --> pvc_postgres
    pv_redis --> pvc_redis
    pv_elastic --> pvc_elastic
    pv_rabbit --> pvc_rabbit
    pv_uploads --> pvc_uploads
    pv_reports --> pvc_reports
    pvc_postgres ==> postgres_pods_storage
    pvc_redis ==> redis_pods_storage
    pvc_elastic ==> elastic_pods_storage
    pvc_rabbit ==> rabbit_pods_storage
    pvc_uploads ==> web_pods_storage
    pvc_reports ==> report_pods_storage
    velero ==> snapshots
    snapshots ==> object_store
    object_store ==> restore_cluster
```

**Key takeaways**
- Storage class choice should match workload behavior, not just capacity numbers.
- RWX shared file needs are distinct from RWO database needs and should be represented separately.
- Velero plus CSI snapshots gives both object-level and cluster-level recovery leverage.

## Diagram 10 — Migration Path: Physical to VM to Docker to Kubernetes

This progression chart places the four common maturity stages side by side so teams can see what changes structurally at each step.
It emphasizes separation of concerns, packaging shifts, and operational capabilities gained during each transition rather than focusing only on runtime technology names.

**When to use:** Use this for roadmap planning, stakeholder alignment, or explaining why intermediate modernization stages still deliver meaningful value.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph LR
    subgraph stage1[Stage 1 Physical]
        phy_user[Users] --> phy_nginx[Nginx + PHP + MySQL on one host]
        phy_nginx --> phy_disk[Local RAID storage]
        phy_notes[Manual patching / manual scaling]
    end
    subgraph stage2[Stage 2 VMs]
        vm_lb[HAProxy VM] --> vm_web[Web VMs]
        vm_web --> vm_app[App VMs]
        vm_app --> vm_db[DB VMs]
        vm_gain[Gain: role separation + snapshots]
    end
    subgraph stage3[Stage 3 Docker]
        docker_ing[Traefik] --> docker_web[Web containers]
        docker_web --> docker_app[App containers]
        docker_app --> docker_data[DB / cache containers]
        docker_gain[Gain: immutable packaging + faster deploys]
    end
    subgraph stage4[Stage 4 Kubernetes]
        k8s_ing[Ingress + HPA] --> k8s_svc[Services + Deployments]
        k8s_svc --> k8s_state[StatefulSets + PVCs]
        k8s_svc -.-> k8s_obs[GitOps + autoscaling + policies]
        k8s_gain[Gain: orchestration + self-healing + platform APIs]
    end
    phy_nginx --> vm_lb
    vm_db --> docker_ing
    docker_data --> k8s_ing
    phy_notes -.-> vm_gain
    vm_gain -.-> docker_gain
    docker_gain -.-> k8s_gain
```

**Key takeaways**
- Each stage should preserve business capabilities while improving packaging, isolation, and operations incrementally.
- Virtualization and containers are not redundant steps; they solve different transition problems.
- Kubernetes is most effective when teams have already separated tiers and containerized deployments consistently.

## Diagram 11 — Kubernetes Scaling Architecture

This diagram shows how different autoscaling controllers combine to adjust replicas, node count, and resource sizing based on both internal and external signals.
It clarifies that horizontal, vertical, event-driven, and cluster-level scaling are complementary controls rather than mutually exclusive choices.

**When to use:** Use this when designing scale behavior for web traffic, asynchronous queue workers, and fluctuating infrastructure demand in the same cluster.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    metrics_server_scale[Metrics Server] --> hpa_cpu[HPA CPU/Memory]
    prom_adapter[Prometheus Adapter] --> hpa_custom[HPA Custom Metrics]
    queue_metrics[RabbitMQ queue depth / HTTP RPS] --> keda[KEDA ScaledObjects]
    hpa_cpu --> web_dep_scale[web Deployment]
    hpa_cpu --> api_dep_scale[api Deployment]
    hpa_custom --> order_dep_scale[order Deployment]
    keda --> worker_dep_scale[worker Deployment]
    vpa[VPA Recommender / Updater] --> api_dep_scale
    vpa --> order_dep_scale
    cluster_autoscaler[Cluster Autoscaler] --> node_group_web[Web Node Group]
    cluster_autoscaler --> node_group_app[App Node Group]
    cluster_autoscaler --> node_group_data[Data Node Group]
    web_dep_scale --> pending_pods_web[Pending web pods?]
    api_dep_scale --> pending_pods_app[Pending api pods?]
    worker_dep_scale --> pending_pods_worker[Pending worker pods?]
    pending_pods_web -.-> cluster_autoscaler
    pending_pods_app -.-> cluster_autoscaler
    pending_pods_worker -.-> cluster_autoscaler
    node_group_web --> web_pods_scaled[Web replicas 4-30]
    node_group_app --> api_pods_scaled[API replicas 3-20]
    node_group_app --> worker_pods_scaled[Worker replicas 2-50]
    node_group_data --> stateful_pods_scaled[Stateful pods with headroom]
```

**Key takeaways**
- HPA handles fast replica changes, KEDA handles event-driven workloads, and Cluster Autoscaler handles underlying capacity.
- VPA is best treated carefully around stateful or latency-sensitive services but is valuable for recommendation loops.
- Scaling design should document triggers and bounds so cost and performance behavior stay predictable.

## Diagram 12 — Disaster Recovery: Multi-Region Kubernetes

This multi-region diagram maps primary and DR clusters, state replication, backup movement, and global entrypoint decisions for a Kubernetes-based ecommerce platform.
It makes the difference between warm standby capacity and data replication paths explicit so DR assumptions are visible before a real incident occurs.

**When to use:** Use this when the business needs a documented path from regional outage to restored service with measurable RPO and RTO expectations.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    gslb_dr[GSLB / GeoDNS with health checks] --> region1[Region 1 Primary]
    gslb_dr -.->|failover| region2[Region 2 DR]

    subgraph reg1_stack[Primary Region]
        k8s_primary[Primary K8s Cluster]
        db_primary_reg[Primary PostgreSQL]
        redis_primary_reg[Primary Redis]
        obj_primary[Primary Object Storage]
        velero_primary[Velero Backup]
    end

    subgraph reg2_stack[DR Region]
        k8s_dr[DR K8s Cluster]
        db_replica_reg[DB Replica]
        redis_replica_reg[Redis Replica]
        obj_dr[DR Object Storage]
        restore_ctrl[Restore Controller]
    end

    subgraph dr_objective[Recovery Targets]
        rpo_note[RPO target 5-15 min for DB]
        rto_note[RTO target 30-60 min for services]
        warm_note[Warm standby nodes kept ready]
    end

    region1 --> k8s_primary
    region2 --> k8s_dr
    k8s_primary --> db_primary_reg
    k8s_primary --> redis_primary_reg
    k8s_primary ==> obj_primary
    db_primary_reg ==> db_replica_reg
    redis_primary_reg ==> redis_replica_reg
    obj_primary ==> obj_dr
    velero_primary ==> obj_primary
    obj_primary ==> restore_ctrl
    restore_ctrl ==> k8s_dr
    restore_ctrl ==> obj_dr
    db_replica_reg --> rpo_note
    k8s_dr --> rto_note
    k8s_dr --> warm_note
```

**Key takeaways**
- Regional DR is a combination of traffic steering, application capacity, and replicated state—not just a second cluster.
- Object storage replication and Velero backups provide a second recovery path when live replication alone is not sufficient.
- Documenting RPO and RTO in the diagram keeps recovery design anchored to business expectations.
