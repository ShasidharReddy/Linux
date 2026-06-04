# 06 - Kubernetes E-Commerce Deployment
<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│             Kubernetes E-Commerce Deployment                │
└──────────────────────────────────────────────────────────────┘
</pre></div>
This document turns the Kubernetes concepts from [05-kubernetes-fundamentals.md](./05-kubernetes-fundamentals.md) into a deployable e-commerce platform. The examples assume a cluster with an ingress controller, a default StorageClass, metrics-server, and enough capacity for stateful workloads.
## Namespace Strategy
Use clear namespace boundaries:
- `ecommerce-prod` for production workloads
- `ecommerce-staging` for pre-production validation
- `monitoring` for Prometheus, Grafana, exporters, and logging agents
- `ingress` for the ingress controller and related edge services
Create namespaces:
~~~bash
kubectl create namespace ecommerce-prod
kubectl create namespace ecommerce-staging
kubectl create namespace monitoring
kubectl create namespace ingress
~~~
## Complete Architecture
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    User[Users] --> Ingress[Nginx Ingress]
    Ingress --> Frontend[Nginx frontend deployment]
    Frontend --> Backend[API / PHP-FPM / Node backend]
    Backend --> Redis[Redis StatefulSet]
    Backend --> MySQL[MySQL StatefulSet]
    Backend --> ES[Elasticsearch StatefulSet]
    Backend --> MQ[RabbitMQ StatefulSet]
    Monitoring[Prometheus / Grafana] --> Frontend
    Monitoring --> Backend
~~~
## Base Resources
### ConfigMap for application settings
~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config
  namespace: ecommerce-prod
data:
  APP_ENV: production
  APP_URL: https://shop.example.com
  LOG_CHANNEL: stderr
  CACHE_DRIVER: redis
  QUEUE_CONNECTION: rabbitmq
  REDIS_HOST: redis.ecommerce-prod.svc.cluster.local
  ELASTICSEARCH_HOST: http://elasticsearch:9200
~~~
### Secret for credentials
~~~yaml
apiVersion: v1
kind: Secret
metadata:
  name: ecommerce-secret
  namespace: ecommerce-prod
type: Opaque
stringData:
  DB_HOST: mysql-0.mysql.ecommerce-prod.svc.cluster.local
  DB_DATABASE: ecommerce
  DB_USERNAME: ecommerce
  DB_PASSWORD: StrongDbPassword123!
  MYSQL_ROOT_PASSWORD: StrongRootPassword123!
  RABBITMQ_DEFAULT_USER: ecommerce
  RABBITMQ_DEFAULT_PASS: StrongRabbitPassword123!
~~~
Tip:
- in GitOps environments, replace raw Secrets with Sealed Secrets or External Secrets Operator integration.
## Nginx Frontend
### Deployment
~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ecommerce-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
        - name: nginx
          image: registry.example.internal/ecommerce/frontend:1.4.0
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /healthz
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          startupProbe:
            httpGet:
              path: /healthz
              port: 80
            failureThreshold: 30
            periodSeconds: 5
~~~
### Service
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: ecommerce-prod
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
~~~
### HPA
~~~yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
  namespace: ecommerce-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 65
~~~
## Backend Deployment
### Deployment
~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: ecommerce-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: app
  template:
    metadata:
      labels:
        app: backend
        tier: app
    spec:
      serviceAccountName: backend-sa
      containers:
        - name: app
          image: registry.example.internal/ecommerce/backend:1.4.0
          ports:
            - containerPort: 9000
          envFrom:
            - configMapRef:
                name: ecommerce-config
            - secretRef:
                name: ecommerce-secret
          resources:
            requests:
              cpu: 500m
              memory: 512Mi
            limits:
              cpu: 2000m
              memory: 2Gi
          livenessProbe:
            tcpSocket:
              port: 9000
            initialDelaySeconds: 20
            periodSeconds: 20
          readinessProbe:
            tcpSocket:
              port: 9000
            initialDelaySeconds: 10
            periodSeconds: 10
          startupProbe:
            tcpSocket:
              port: 9000
            failureThreshold: 30
            periodSeconds: 5
~~~
### Service
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: ecommerce-prod
spec:
  selector:
    app: backend
  ports:
    - port: 9000
      targetPort: 9000
  type: ClusterIP
~~~
### HPA with CPU and memory
~~~yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: ecommerce-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 3
  maxReplicas: 12
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 75
~~~
## MySQL StatefulSet
### Headless service
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: ecommerce-prod
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
~~~
### ConfigMap for MySQL
~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: ecommerce-prod
data:
  primary.cnf: |
    [mysqld]
    server-id=1
    log_bin=mysql-bin
    binlog_format=ROW
    gtid_mode=ON
    enforce_gtid_consistency=ON
    bind-address=0.0.0.0
  replica.cnf: |
    [mysqld]
    log_bin=mysql-bin
    relay_log=relay-bin
    read_only=ON
    super_read_only=ON
~~~
### StatefulSet with init containers
~~~yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: ecommerce-prod
spec:
  serviceName: mysql
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
        tier: data
    spec:
      initContainers:
        - name: init-mysql
          image: mysql:8.4
          command:
            - bash
            - -c
            - |
              ordinal=$(hostname | awk -F'-' '{print $NF}')
              if [ "$ordinal" = "0" ]; then
                cp /config/primary.cnf /mnt/conf.d/server-id.cnf
              else
                cp /config/replica.cnf /mnt/conf.d/server-id.cnf
                echo "server-id=$((100 + ordinal))" >> /mnt/conf.d/server-id.cnf
              fi
          volumeMounts:
            - name: config
              mountPath: /config
            - name: conf
              mountPath: /mnt/conf.d
      containers:
        - name: mysql
          image: mysql:8.4
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ecommerce-secret
                  key: MYSQL_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              value: ecommerce
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
            - name: conf
              mountPath: /etc/mysql/conf.d
          resources:
            requests:
              cpu: 1000m
              memory: 4Gi
            limits:
              cpu: 4000m
              memory: 8Gi
          livenessProbe:
            exec:
              command: ["sh", "-c", "mysqladmin ping -uroot -p$MYSQL_ROOT_PASSWORD"]
            initialDelaySeconds: 30
            periodSeconds: 20
          readinessProbe:
            exec:
              command: ["sh", "-c", "mysqladmin ping -uroot -p$MYSQL_ROOT_PASSWORD"]
            initialDelaySeconds: 15
            periodSeconds: 10
      volumes:
        - name: config
          configMap:
            name: mysql-config
        - name: conf
          emptyDir: {}
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 100Gi
~~~
Replication bootstrapping note:
- primary/replica automation usually needs additional logic or operators; for labs, use init containers plus post-start scripts or an operator like Oracle MySQL Operator.
## Redis StatefulSet
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: ecommerce-prod
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: ecommerce-prod
spec:
  serviceName: redis
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        tier: cache
    spec:
      containers:
        - name: redis
          image: redis:7.4-alpine
          args: ["redis-server", "--appendonly", "yes"]
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
          volumeMounts:
            - name: data
              mountPath: /data
          livenessProbe:
            exec:
              command: ["redis-cli", "ping"]
            initialDelaySeconds: 10
            periodSeconds: 20
          readinessProbe:
            exec:
              command: ["redis-cli", "ping"]
            initialDelaySeconds: 5
            periodSeconds: 10
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi
~~~
## Elasticsearch StatefulSet
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: ecommerce-prod
spec:
  clusterIP: None
  selector:
    app: elasticsearch
  ports:
    - port: 9200
      targetPort: 9200
    - port: 9300
      targetPort: 9300
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: ecommerce-prod
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
        tier: search
    spec:
      containers:
        - name: elasticsearch
          image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
          env:
            - name: discovery.seed_hosts
              value: elasticsearch-0.elasticsearch,easticsearch-1.elasticsearch,elasticsearch-2.elasticsearch
            - name: cluster.initial_master_nodes
              value: elasticsearch-0,elasticsearch-1,elasticsearch-2
            - name: cluster.name
              value: ecommerce-search
            - name: node.store.allow_mmap
              value: "false"
            - name: ES_JAVA_OPTS
              value: "-Xms1g -Xmx1g"
            - name: xpack.security.enabled
              value: "false"
          ports:
            - containerPort: 9200
            - containerPort: 9300
          resources:
            requests:
              cpu: 1000m
              memory: 2Gi
            limits:
              cpu: 2000m
              memory: 4Gi
          volumeMounts:
            - name: data
              mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 50Gi
~~~
Important fix when applying this manifest in real clusters:
- correct the `discovery.seed_hosts` value if you copy it; each DNS name must match the headless service and StatefulSet naming pattern exactly.
## RabbitMQ StatefulSet
~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq
  namespace: ecommerce-prod
spec:
  clusterIP: None
  selector:
    app: rabbitmq
  ports:
    - port: 5672
      targetPort: 5672
    - port: 15672
      targetPort: 15672
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: rabbitmq
  namespace: ecommerce-prod
spec:
  serviceName: rabbitmq
  replicas: 1
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
        tier: queue
    spec:
      containers:
        - name: rabbitmq
          image: rabbitmq:3.13-management
          env:
            - name: RABBITMQ_DEFAULT_USER
              valueFrom:
                secretKeyRef:
                  name: ecommerce-secret
                  key: RABBITMQ_DEFAULT_USER
            - name: RABBITMQ_DEFAULT_PASS
              valueFrom:
                secretKeyRef:
                  name: ecommerce-secret
                  key: RABBITMQ_DEFAULT_PASS
          ports:
            - containerPort: 5672
            - containerPort: 15672
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
          volumeMounts:
            - name: data
              mountPath: /var/lib/rabbitmq
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi
~~~
## Ingress Configuration
### Nginx Ingress with TLS and path routing
~~~yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce-prod
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 32m
    nginx.ingress.kubernetes.io/limit-rps: "20"
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://shop.example.com"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "Authorization,Content-Type,X-Requested-With"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - shop.example.com
      secretName: ecommerce-tls
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 9000
          - path: /admin
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 9000
          - path: /static
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
~~~
## Traffic Flow
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    U[User] --> I[Ingress TLS termination]
    I --> F[Frontend service]
    I --> B[Backend service for /api and /admin]
    B --> D[(Redis MySQL RabbitMQ Elasticsearch)]
~~~
## Custom Metrics HPA Example
Queue-depth or request-rate based scaling usually requires Prometheus Adapter or KEDA.
~~~yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-custom-hpa
  namespace: ecommerce-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 3
  maxReplicas: 15
  metrics:
    - type: Pods
      pods:
        metric:
          name: requests_per_second
        target:
          type: AverageValue
          averageValue: "20"
~~~
## LimitRange and ResourceQuota
### LimitRange
~~~yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: ecommerce-defaults
  namespace: ecommerce-prod
spec:
  limits:
    - default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      type: Container
~~~
### ResourceQuota
~~~yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ecommerce-quota
  namespace: ecommerce-prod
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "20"
~~~
## CronJobs
### Order cleanup
~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: order-cleanup
  namespace: ecommerce-prod
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: registry.example.internal/ecommerce/backend:1.4.0
              command: ["php", "artisan", "orders:cleanup"]
~~~
### Report generation
~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report-generation
  namespace: ecommerce-prod
spec:
  schedule: "30 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: report
              image: registry.example.internal/ecommerce/backend:1.4.0
              command: ["php", "artisan", "reports:generate"]
~~~
### Inventory sync
~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: inventory-sync
  namespace: ecommerce-prod
spec:
  schedule: "*/15 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: inventory
              image: registry.example.internal/ecommerce/backend:1.4.0
              command: ["php", "artisan", "inventory:sync"]
~~~
## Scaling Workflow
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    Metric[CPU Memory Custom Metric] --> HPA[HorizontalPodAutoscaler]
    HPA --> Deploy[Deployment replica change]
    Deploy --> Scheduler[Scheduler places pods]
    Scheduler --> Nodes[Worker nodes]
~~~
## Health Check Strategy
Apply:
- startup probes to slower boot services such as PHP-FPM or Java apps
- readiness probes to avoid routing traffic before the app is ready
- liveness probes only when you are confident failure means restart is the correct action
Examples already shown above cover frontend, backend, MySQL, and Redis.
[← Back to Virtual Setup](./README.md)
