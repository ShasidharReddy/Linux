# 10. Container Orchestration Basics

## 10.1 Why Orchestration Exists

Running one container on one host is simple.
Running many containers across many hosts requires automation.
Orchestration platforms help with:

- Scheduling
- Scaling
- Service discovery
- Rolling updates
- Health-based replacement
- Secret/config management
- Declarative desired state

## 10.2 Docker Swarm Overview

Docker Swarm is Docker's native clustering/orchestration mode.
It offers:

- Service abstraction
- Overlay networking
- Built-in load balancing
- Secrets/configs
- Rolling updates

## 10.3 Swarm Concepts

| Concept | Meaning |
|---|---|
| Node | Machine in the swarm |
| Manager | Control-plane node |
| Worker | Executes tasks |
| Service | Desired state definition |
| Task | Running unit of a service |

## 10.4 Swarm Quick Example

Initialize swarm:

```bash
docker swarm init
```

Create service:

```bash
docker service create --name web --replicas 3 -p 8080:80 nginx:stable
```

Inspect services:

```bash
docker service ls
```

## 10.5 Swarm Strengths and Limitations

Strengths:

- Simpler than Kubernetes for some teams
- Built into Docker ecosystem
- Good for smaller clusters

Limitations:

- Smaller ecosystem
- Less feature depth than Kubernetes
- Less dominant in industry adoption

## 10.6 Kubernetes Overview

Kubernetes is the dominant container orchestration platform.
It provides declarative management for containerized workloads at scale.

## 10.7 Core Kubernetes Concepts

| Concept | Meaning |
|---|---|
| Pod | Smallest deployable unit, one or more containers |
| Deployment | Manages stateless replica rollout |
| Service | Stable network endpoint/load balancing |
| ConfigMap | Non-secret configuration |
| Secret | Sensitive configuration data |
| StatefulSet | Stateful workload management |
| DaemonSet | One pod per node pattern |
| Ingress | HTTP/S routing into cluster |
| Node | Worker machine |
| Namespace | Logical cluster grouping |

## 10.8 Pod Concept

A Pod can contain:

- One primary container
- Optionally helper sidecars/init containers
- Shared network namespace
- Shared volumes

This is a key difference from plain Docker single-container mental models.

## 10.9 Deployment Concept

A Deployment manages replica sets for stateless apps.
It handles:

- Desired replica count
- Rolling updates
- Rollbacks

## 10.10 Service Concept

A Service provides stable discovery/load balancing for pods whose IPs may change.
Common types:

- ClusterIP
- NodePort
- LoadBalancer

## 10.11 ConfigMaps and Secrets

These externalize configuration from container images.
That supports immutable image patterns.

## 10.12 Ingress Basics

Ingress provides HTTP/S routing into cluster services.
It is typically backed by an ingress controller such as NGINX Ingress or Traefik.

## 10.13 Kubernetes Scheduling

The scheduler decides where pods run based on:

- Resource requests/limits
- Affinity/anti-affinity
- Taints/tolerations
- Node selectors
- Topology constraints

## 10.14 Requests and Limits

Resource requests influence scheduling.
Limits constrain runtime usage.
These concepts extend the cgroup principles discussed earlier.

## 10.15 Rolling Updates

Orchestrators can update workloads gradually.
Typical goals:

- Minimize downtime
- Replace unhealthy instances automatically
- Roll back bad releases

## 10.16 Self-Healing

A big advantage of orchestration is self-healing.
If a container or node fails, the platform tries to restore desired state automatically.

## 10.17 Basic Kubernetes YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
```

## 10.18 Kubernetes Learning Path for Docker Users

If you know Docker, learn Kubernetes in this order:

1. Pods
2. Deployments
3. Services
4. ConfigMaps/Secrets
5. Probes
6. Volumes/PVCs
7. Ingress
8. RBAC and security contexts

## 10.19 Compose vs Kubernetes

| Area | Compose | Kubernetes |
|---|---|---|
| Scope | Single host mostly | Cluster orchestration |
| Complexity | Lower | Higher |
| Scheduling | Minimal | Advanced |
| Self-healing | Limited | Strong |
| Ecosystem | Developer-centric | Production platform-centric |

## 10.20 Summary

Orchestration automates the lifecycle of many containers across one or more hosts.
Swarm offers approachable clustering; Kubernetes offers a broader, deeper platform with more operational power.

---

## B.10 Orchestration Q&A

### Q181. Why use orchestration?
A181. To manage many containers declaratively across hosts.

### Q182. What is Docker Swarm?
A182. Docker's native orchestration/clustering mode.

### Q183. What is a Swarm service?
A183. A desired state definition for replicated tasks.

### Q184. What is Kubernetes' smallest deployable unit?
A184. A Pod.

### Q185. What does a Deployment manage?
A185. Replica rollout and desired state for stateless pods.

### Q186. What provides a stable network endpoint to changing pods?
A186. A Kubernetes Service.

### Q187. What is a ConfigMap?
A187. Non-secret configuration object in Kubernetes.

### Q188. What is a Secret in Kubernetes?
A188. Sensitive configuration data object.

### Q189. What is a StatefulSet for?
A189. Stateful workloads with stable identities/storage patterns.

### Q190. What is a DaemonSet for?
A190. Running one pod per node.

### Q191. What is Ingress for?
A191. HTTP/S routing into cluster services.

### Q192. What is a node?
A192. A worker machine in the cluster.

### Q193. Why are resource requests and limits important in Kubernetes?
A193. Scheduling and runtime control.

### Q194. What is self-healing in orchestration?
A194. Replacing failed workloads automatically to restore desired state.

### Q195. What is a rolling update?
A195. Gradual replacement of old instances with new ones.

### Q196. Is Compose equivalent to Kubernetes?
A196. No.

### Q197. Why do pods often contain sidecars?
A197. For tightly coupled helper functions like logging or proxies.

### Q198. Why externalize config in orchestration platforms?
A198. To keep images immutable and portable.

### Q199. Why is service discovery essential in clusters?
A199. Pod IPs are dynamic.

### Q200. Why does orchestration complexity increase with scale?
A200. Scheduling, networking, security, and observability all become multi-node problems.
