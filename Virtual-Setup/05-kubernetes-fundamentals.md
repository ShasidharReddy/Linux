# 05 - Kubernetes Fundamentals

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                 Kubernetes Fundamentals                     │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This document introduces the Kubernetes primitives needed for the e-commerce deployment in [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md). It covers architecture, cluster setup options, core resources, networking, storage, and RBAC.

## Kubernetes Architecture

Kubernetes separates control plane components from worker nodes.

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    API[API Server] --> ETCD[etcd]
    API --> Scheduler[Scheduler]
    API --> Controller[Controller Manager]
    API --> CCM[Cloud Controller Manager]
    Scheduler --> Worker1[Worker Node 1]
    Scheduler --> Worker2[Worker Node 2]
    Scheduler --> Worker3[Worker Node 3]
    Worker1 --> Kubelet1[kubelet]
    Worker2 --> Kubelet2[kubelet]
    Worker3 --> Kubelet3[kubelet]
~~~

### Control Plane Components

- API server: central entry point
- etcd: state store
- scheduler: assigns pods to nodes
- controller manager: reconciliation loops
- cloud controller manager: cloud integrations when applicable

### Worker Node Components

- kubelet
- kube-proxy or eBPF-based replacement
- container runtime such as containerd
- CNI plugin for pod networking

## Cluster Setup Options

### kubeadm - 3-node cluster

Use kubeadm when you want a standard upstream-style cluster on VMs or bare metal.

#### Install prerequisites on all nodes

~~~bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
~~~

#### Install containerd

~~~bash
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
~~~

#### Install Kubernetes packages

~~~bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
~~~

#### Initialize the control plane

~~~bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

#### Join worker nodes

Use the token printed by `kubeadm init`, for example:

~~~bash
sudo kubeadm join 192.168.56.10:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:1111111111111111111111111111111111111111111111111111111111111111
~~~

#### Install a CNI plugin

Flannel example:

~~~bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubectl get nodes -o wide
~~~

### k3s

k3s is excellent for labs, edge nodes, and lightweight clusters.

~~~bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
~~~

Join an agent:

~~~bash
curl -sfL https://get.k3s.io | K3S_URL=https://k3s-server:6443 K3S_TOKEN=SECRET sh -
~~~

### Kind and Minikube

Use these for local development.

Kind example:

~~~bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind create cluster --name ecommerce
kubectl cluster-info --context kind-ecommerce
~~~

Minikube example:

~~~bash
minikube start --cpus=4 --memory=8192 --driver=docker
kubectl get nodes
~~~

## Core Resources

### Pod

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
    - name: nginx
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
~~~

Create it:

~~~bash
kubectl apply -f pod.yaml
kubectl get pods -o wide
~~~

### Deployment

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
~~~

### Service

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
~~~

### Ingress

~~~yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: shop.example.internal
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
~~~

### ConfigMap

~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_URL: https://shop.example.internal
~~~

### Secret

~~~yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  DB_USERNAME: ecommerce
  DB_PASSWORD: StrongPassword123!
~~~

### PersistentVolume and PVC

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/mysql
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
~~~

### StatefulSet

~~~yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
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
    spec:
      containers:
        - name: redis
          image: redis:7.4-alpine
          ports:
            - containerPort: 6379
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
~~~

### DaemonSet

~~~yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:v1.8.2
          ports:
            - containerPort: 9100
~~~

### Job and CronJob

~~~yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: registry.example.internal/ecommerce/app:1.4.0
          command: ["php", "artisan", "migrate", "--force"]
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-report
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: report
              image: registry.example.internal/ecommerce/app:1.4.0
              command: ["php", "artisan", "reports:generate"]
~~~

## Pod Networking

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    PodA[Pod A] --> Service[ClusterIP Service]
    PodB[Pod B] --> Service
    Service --> PodC[Backend Pod C]
    Service --> PodD[Backend Pod D]
~~~

## Networking Concepts

### CNI Plugins

| Plugin | Strengths | Typical Fit |
| --- | --- | --- |
| Calico | Network policy, BGP support, mature operations | Production clusters needing policy control |
| Flannel | Simpler overlay networking | Labs and smaller clusters |
| Cilium | eBPF-based networking, advanced observability and policy | High-performance or security-focused clusters |

### Service Types

| Type | Description | Common Use |
| --- | --- | --- |
| ClusterIP | Internal-only service | App-to-app communication |
| NodePort | Exposes a port on each node | Simple testing or edge integration |
| LoadBalancer | External LB integration | Cloud or MetalLB-backed ingress |

### Ingress Controllers

| Controller | Strength | Typical Use |
| --- | --- | --- |
| Nginx Ingress | Widely used, rich annotations | General web workloads |
| Traefik | Dynamic routing, easy CRDs | Labs and modern edge routing |
| HAProxy Ingress | Strong performance and control | High-throughput HTTP environments |

### NetworkPolicy example

~~~yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-db
  namespace: ecommerce-prod
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: app
      ports:
        - protocol: TCP
          port: 3306
~~~

## Storage

### StorageClasses

| StorageClass | Example Backend | Best For |
| --- | --- | --- |
| local-path | local disk | labs and single-node clusters |
| nfs-client | NFS provisioner | shared file storage |
| ceph-rbd | Ceph block | resilient production clusters |
| gp3 / pd-ssd / managed-premium | cloud block volumes | managed cloud storage |

### Dynamic provisioning

~~~yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-rwo
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
~~~

PVC using a StorageClass:

~~~yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-data
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard-rwo
  resources:
    requests:
      storage: 10Gi
~~~

## Storage Provisioning Flow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    PVC[PersistentVolumeClaim] --> SC[StorageClass]
    SC --> Provisioner[Dynamic Provisioner]
    Provisioner --> PV[PersistentVolume]
    PV --> Pod[Pod Mounts Volume]
~~~

## RBAC

### Role

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: ecommerce-staging
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list"]
~~~

### RoleBinding

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-reader-binding
  namespace: ecommerce-staging
subjects:
  - kind: ServiceAccount
    name: app-sa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-reader
~~~

### ClusterRole

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
~~~

### Service account example

~~~yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: ecommerce-prod
~~~

Validation commands:

~~~bash
kubectl auth can-i get secrets --as=system:serviceaccount:ecommerce-prod:app-sa -n ecommerce-prod
kubectl get role,rolebinding -n ecommerce-staging
~~~

## Operational Tips

- prefer Deployments for stateless services and StatefulSets for persistent identities
- use namespaces to separate staging, production, monitoring, and ingress
- start with ClusterIP services internally and expose only through Ingress
- use ConfigMaps for non-secret config and Secrets for sensitive values
- apply NetworkPolicies early so default-open networking does not become permanent
- standardize labels for app, tier, environment, and team ownership

## Common Pitfalls

- assuming a Pod is durable like a VM
- using hostPath volumes in production without understanding node coupling
- putting secrets in ConfigMaps
- skipping readiness probes and sending traffic too early
- using NodePort as a long-term internet edge for serious production traffic
- ignoring resource requests and limits

## Where to Go Next

- Use these concepts in [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md).
- Compare service packaging in [03-docker-fundamentals.md](./03-docker-fundamentals.md).
- Study production hardening in [07-production-kubernetes-setup.md](./07-production-kubernetes-setup.md).

## Summary

Kubernetes provides the declarative control plane that turns container packaging into a resilient application platform. Once you understand pods, services, persistent storage, ingress, and RBAC, you can deploy realistic multi-tier commerce platforms with confidence.

[← Back to Virtual Setup](./README.md)
