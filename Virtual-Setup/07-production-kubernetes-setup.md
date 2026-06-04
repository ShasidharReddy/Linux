# 07 - Production Kubernetes Setup

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│               Production Kubernetes Setup                   │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This document covers production-grade Kubernetes operations for the e-commerce platform. It builds directly on [05-kubernetes-fundamentals.md](./05-kubernetes-fundamentals.md) and [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md), then layers service mesh, GitOps, CI/CD, observability, security, disaster recovery, and cost optimization.

## Production Cluster Architecture

### Design Principles

- isolate workloads by function and failure domain
- spread critical services across racks or availability zones
- keep system workloads separate from business applications
- plan for node failures, certificate rotation, and version upgrades
- automate everything that is repeated more than once

### Recommended Node Pools

| Pool | Purpose | Labels | Taints |
| --- | --- | --- | --- |
| system | ingress, DNS, metrics, controllers | `nodepool=system` | `CriticalAddonsOnly=true:NoSchedule` |
| web | edge and frontend workloads | `nodepool=web` | optional |
| app | APIs, workers, business services | `nodepool=app` | optional |
| data | MySQL, Redis, search, RabbitMQ | `nodepool=data` | `workload=data:NoSchedule` |

### Multi-AZ Layout

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    subgraph AZ1[AZ / Rack 1]
        CP1[Control Plane 1]
        W1[Web Nodes]
        A1[App Nodes]
        D1[Data Node]
    end
    subgraph AZ2[AZ / Rack 2]
        CP2[Control Plane 2]
        W2[Web Nodes]
        A2[App Nodes]
        D2[Data Node]
    end
    subgraph AZ3[AZ / Rack 3]
        CP3[Control Plane 3]
        W3[Web Nodes]
        A3[App Nodes]
        D3[Data Node]
    end
    LB[Global Load Balancer] --> W1
    LB --> W2
    LB --> W3
~~~

## Taints, Tolerations, and Spread Constraints

### Example data-node taint

~~~bash
kubectl taint nodes data-node-01 workload=data:NoSchedule
~~~

### Example toleration for MySQL

~~~yaml
spec:
  template:
    spec:
      tolerations:
        - key: workload
          operator: Equal
          value: data
          effect: NoSchedule
~~~

### Pod topology spread constraints

~~~yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: backend
~~~

## Service Mesh with Istio

### Install Istio

~~~bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.23.0
export PATH=$PWD/bin:$PATH
istioctl install --set profile=default -y
kubectl label namespace ecommerce-prod istio-injection=enabled
~~~

### mTLS between services

~~~yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: ecommerce-prod
spec:
  mtls:
    mode: STRICT
~~~

### DestinationRule for TLS

~~~yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: backend-dr
  namespace: ecommerce-prod
spec:
  host: backend.ecommerce-prod.svc.cluster.local
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 20
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 10s
      baseEjectionTime: 30s
~~~

### Canary deployment with VirtualService

~~~yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: backend
  namespace: ecommerce-prod
spec:
  hosts:
    - api.shop.example.com
  gateways:
    - ecommerce-gateway
  http:
    - route:
        - destination:
            host: backend
            subset: stable
          weight: 90
        - destination:
            host: backend
            subset: canary
          weight: 10
~~~

### Blue-green pattern

~~~yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: frontend-bluegreen
  namespace: ecommerce-prod
spec:
  hosts:
    - shop.example.com
  http:
    - route:
        - destination:
            host: frontend-blue
          weight: 100
        - destination:
            host: frontend-green
          weight: 0
~~~

### Retry and circuit-breaker patterns

~~~yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: checkout-api
  namespace: ecommerce-prod
spec:
  hosts:
    - checkout.ecommerce-prod.svc.cluster.local
  http:
    - retries:
        attempts: 3
        perTryTimeout: 2s
      route:
        - destination:
            host: checkout
~~~

## Service Mesh Traffic Flow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    User[User] --> Gateway[Istio Gateway]
    Gateway --> Sidecar1[Frontend pod + Envoy]
    Sidecar1 --> Sidecar2[Backend pod + Envoy]
    Sidecar2 --> Sidecar3[MySQL proxy path or data service]
~~~

### Observability add-ons

Install Kiali, Jaeger, and metrics support:

~~~bash
kubectl apply -f samples/addons/prometheus.yaml
kubectl apply -f samples/addons/grafana.yaml
kubectl apply -f samples/addons/jaeger.yaml
kubectl apply -f samples/addons/kiali.yaml
~~~

## GitOps with Argo CD

### Install Argo CD

~~~bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
~~~

### Example Argo CD Application

~~~yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example-org/ecommerce-platform.git
    targetRevision: main
    path: deploy/k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: ecommerce-prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
~~~

### Rollback with Argo CD

~~~bash
argocd app history ecommerce-prod
argocd app rollback ecommerce-prod 12
~~~

Best practices:

- separate base manifests from environment overlays
- sign commits or protect branches
- use sync waves for data dependencies
- keep secrets sealed or externally sourced

## CI/CD Pipeline

The production pipeline should build, test, scan, package, and promote images rather than deploying arbitrary mutable tags.

### Example Jenkins pipeline outline

~~~groovy
pipeline {
  agent any
  environment {
    REGISTRY = 'registry.example.internal'
    IMAGE = 'ecommerce/backend'
    VERSION = "${env.BUILD_NUMBER}"
  }
  stages {
    stage('Build') {
      steps {
        sh 'docker build -t $REGISTRY/$IMAGE:$VERSION .'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run --rm $REGISTRY/$IMAGE:$VERSION php artisan test'
      }
    }
    stage('Scan') {
      steps {
        sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $REGISTRY/$IMAGE:$VERSION'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push $REGISTRY/$IMAGE:$VERSION'
      }
    }
    stage('Promote') {
      steps {
        sh "sed -i 's/tag:.*/tag: ${VERSION}/' deploy/helm/prod-values.yaml"
      }
    }
  }
}
~~~

### Helm chart packaging

~~~bash
helm lint deploy/helm/ecommerce
helm package deploy/helm/ecommerce
helm repo index .
~~~

### Promotion flow

- build image in CI
- scan and sign image
- push to registry
- promote tag to staging values
- verify smoke tests
- promote same immutable tag to production

## CI/CD Pipeline Diagram

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Commit[Git Commit] --> Build[Build Image]
    Build --> Test[Test Suite]
    Test --> Scan[Security Scan]
    Scan --> Package[Helm Package]
    Package --> GitOps[Update GitOps Repo]
    GitOps --> Argo[Argo CD Sync]
    Argo --> Prod[Production Cluster]
~~~

## Monitoring and Observability

### Prometheus Operator

Install the kube-prometheus-stack Helm chart:

~~~bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
~~~

### Sample ServiceMonitor

~~~yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: frontend
  namespace: monitoring
spec:
  namespaceSelector:
    matchNames: ["ecommerce-prod"]
  selector:
    matchLabels:
      app: frontend
  endpoints:
    - port: http
      path: /metrics
      interval: 30s
~~~

### Grafana dashboards

Create dashboards for:

- orders/minute
- checkout success rate
- cart abandonment proxy metrics
- latency P50, P95, P99
- pod restarts by deployment
- queue backlog and worker throughput
- MySQL connections and replication health

### Alerting rules

~~~yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: ecommerce-alerts
  namespace: monitoring
spec:
  groups:
    - name: ecommerce.rules
      rules:
        - alert: PodRestartHigh
          expr: increase(kube_pod_container_status_restarts_total[15m]) > 3
          for: 10m
          labels:
            severity: warning
          annotations:
            summary: Excessive pod restarts detected
        - alert: ErrorRateHigh
          expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: HTTP 5xx error rate above 5 percent
        - alert: LatencyP99High
          expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 1.5
          for: 15m
          labels:
            severity: critical
          annotations:
            summary: P99 latency above 1.5 seconds
~~~

### Centralized logging with EFK

Suggested components:

- Fluent Bit DaemonSet on every node
- Elasticsearch or OpenSearch storage cluster
- Kibana or OpenSearch Dashboards

### Distributed tracing with Jaeger

Instrument:

- frontend checkout routes
- backend API calls
- payment gateway integration
- order fulfillment worker jobs

## Security

### Pod Security Standards

Label namespaces for restricted policy when supported:

~~~bash
kubectl label namespace ecommerce-prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
~~~

### Example restricted security context

~~~yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
~~~

### OPA Gatekeeper

Install Gatekeeper:

~~~bash
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper gatekeeper/gatekeeper -n gatekeeper-system --create-namespace
~~~

Example ConstraintTemplate:

~~~yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {"app", "team", "environment"}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("missing required labels: %v", [missing])
        }
~~~

### Zero-trust NetworkPolicies

- default deny ingress and egress per namespace
- explicitly allow frontend to backend
- explicitly allow backend to Redis, MySQL, RabbitMQ, and Elasticsearch
- deny east-west traffic that is not required

### Image scanning in CI

~~~bash
trivy image --exit-code 1 --severity HIGH,CRITICAL registry.example.internal/ecommerce/backend:1.4.0
~~~

### Runtime security with Falco

~~~bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco -n falco --create-namespace
~~~

## Disaster Recovery

### Velero for cluster backup

~~~bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.10.0 \
  --bucket velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1
~~~

### Backup namespaces and volumes

~~~bash
velero backup create ecommerce-prod-backup --include-namespaces ecommerce-prod,monitoring
velero backup describe ecommerce-prod-backup --details
~~~

### Database backup CronJob

~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup
  namespace: ecommerce-prod
spec:
  schedule: "0 */6 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: dump
              image: mysql:8.4
              command:
                - sh
                - -c
                - |
                  mysqldump -h mysql-0.mysql -uroot -p$MYSQL_ROOT_PASSWORD --all-databases > /backup/backup.sql
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: ecommerce-secret
                      key: MYSQL_ROOT_PASSWORD
              volumeMounts:
                - name: backup
                  mountPath: /backup
          volumes:
            - name: backup
              persistentVolumeClaim:
                claimName: mysql-backup-pvc
~~~

### DR drill procedure

1. simulate namespace loss in staging
2. restore manifests from GitOps source
3. restore data from Velero or database backups
4. validate checkout flow and order visibility
5. document recovery time and data loss window

## DR Architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Prod[Primary Cluster] --> Backup[Velero Backups]
    Prod --> DBBackup[Database Dumps]
    Backup --> DR[DR Cluster]
    DBBackup --> DR
    GitOps[GitOps Repository] --> DR
~~~

## Cost Optimization

### Right-sizing with VPA

~~~bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler-crd.yaml
~~~

### Example VPA

~~~yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
  namespace: ecommerce-prod
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  updatePolicy:
    updateMode: Off
~~~

### Spot or preemptible nodes

Use for:

- batch jobs
- CI runners
- non-critical search reindexing
- asynchronous analytics workloads

Avoid using them for:

- MySQL primaries
- ingress controllers without redundancy
- latency-sensitive checkout APIs

### Quota enforcement

- require requests and limits on all workloads
- use cost dashboards based on namespaces and labels
- review idle PVCs and orphaned load balancers regularly

## Upgrade and Maintenance Practices

- upgrade control plane first, then node pools
- keep one minor version skew within supported limits
- rehearse ingress, CNI, and CSI upgrades in staging first
- use PodDisruptionBudgets for critical stateless services
- test backup and restore before any major version upgrade

## Common Pitfalls

- running stateful data services without node isolation
- mixing GitOps and manual kubectl edits in production
- skipping mTLS and network policy in a multi-team cluster
- storing dashboards, alerts, and policies outside version control
- failing to rehearse disaster recovery under time pressure
- treating observability as optional instead of foundational

## Next Steps

- Automate delivery with [08-ci-cd-and-automation.md](./08-ci-cd-and-automation.md).
- Extend hybrid and geo-distributed strategies with [09-hybrid-and-scaling.md](./09-hybrid-and-scaling.md).
- Revisit deployment manifests in [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md) to add service-mesh and policy hooks.

## Summary

Production Kubernetes for e-commerce is much more than deploying pods. It requires controlled node placement, GitOps discipline, secure service-to-service communication, strong observability, tested recovery workflows, and continuous cost management. When those pieces are in place, Kubernetes becomes a stable platform for high-growth online systems.

[← Back to Virtual Setup](./README.md)
