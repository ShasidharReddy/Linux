# Question 11 (BONUS): Kubernetes Deployment on Top of This Infrastructure

This chapter assumes the lower platform is already correct. Kubernetes is not the foundation here; it is the platform layer that sits on top of stable hypervisors, network segmentation, shared storage, hardened Linux images, and observability.

## Dependencies Already in Place

1. Hypervisor cluster ✓
2. Network with dedicated K8S VLAN 60, DNS, NTP ✓
3. Firewall rules for 6443, 2379-2380, 10250, 30000-32767 ✓
4. Shared storage for persistent volumes ✓
5. Golden image with container runtime prerequisites ✓
6. Load balancer VMs for API VIP ✓

## K8s dependency chain

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Hypervisor Cluster] --> B[Node VMs]
    B --> C[K8S VLAN and Firewall]
    C --> D[API Load Balancer VIP]
    D --> E[kubeadm init]
    E --> F[Join Control Planes]
    F --> G[Join Workers]
    G --> H[Install CNI Ingress StorageClass]
~~~

## VM sizing for Kubernetes

| Role | Count | vCPU | RAM | Disk | Purpose |
|------|-------|------|-----|------|---------|
| Control plane | 3 | 4 | 8 GB | 100 GB | API server, etcd, scheduler, controller |
| Worker (general) | 3-5 | 8 | 32 GB | 200 GB | stateless workloads |
| Worker (data) | 2-3 | 8 | 64 GB | 500 GB | stateful and data-heavy workloads |
| HAProxy/LB | 2 | 2 | 4 GB | 50 GB | API VIP via keepalived |

## HA control plane design

Use **3 control plane nodes** with **stacked etcd** and an **HAProxy + keepalived** VIP in front of the API.

Why stacked etcd here:

- fewer nodes to manage at moderate scale
- standard kubeadm pattern
- sufficient for this on-prem starting point

Choose external etcd only when separation of duties or larger scale justifies the extra control plane complexity.

## HA control plane architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Admin[Operators kubectl] --> VIP[API VIP 10.10.60.10]
    VIP --> LB1[haproxy01]
    VIP --> LB2[haproxy02]
    LB1 --> CP1[cp01]
    LB1 --> CP2[cp02]
    LB2 --> CP2
    LB2 --> CP3[cp03]
    CP1 --- ETCD1[etcd]
    CP2 --- ETCD2[etcd]
    CP3 --- ETCD3[etcd]
    CP1 --> W1[workers]
    CP2 --> W1
    CP3 --> W1
~~~

## HAProxy config for API VIP

`/etc/haproxy/haproxy.cfg`

~~~bash
global
  log /dev/log local0
  maxconn 4096

defaults
  log global
  mode tcp
  timeout connect 5s
  timeout client  50s
  timeout server  50s

frontend k8s_api
  bind 10.10.60.10:6443
  default_backend k8s_control_plane

backend k8s_control_plane
  balance roundrobin
  option tcp-check
  server cp01 10.10.60.21:6443 check
  server cp02 10.10.60.22:6443 check
  server cp03 10.10.60.23:6443 check
~~~

## keepalived config

`/etc/keepalived/keepalived.conf`

~~~bash
vrrp_script chk_haproxy {
  script "pidof haproxy"
  interval 2
}

vrrp_instance VI_60 {
  state BACKUP
  interface eth0
  virtual_router_id 60
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass ReplaceMe123
  }
  virtual_ipaddress {
    10.10.60.10/24
  }
  track_script {
    chk_haproxy
  }
}
~~~

## Prepare nodes

On all K8s nodes:

~~~bash
swapoff -a
sed -i.bak '/ swap / s/^/#/' /etc/fstab
modprobe overlay
modprobe br_netfilter
cat >/etc/sysctl.d/99-kubernetes.conf <<'EOT'
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOT
sysctl --system
~~~

## Install containerd, kubelet, kubeadm, kubectl

### RHEL-family example

~~~bash
cat >/etc/yum.repos.d/kubernetes.repo <<'EOT'
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
EOT

dnf install -y containerd kubelet kubeadm kubectl
systemctl enable --now containerd kubelet
~~~

## kubeadm config example

~~~yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: v1.30.1
controlPlaneEndpoint: "10.10.60.10:6443"
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
apiServer:
  certSANs:
    - 10.10.60.10
    - k8s-api.infra.example.com
controllerManager: {}
scheduler: {}
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
systemReserved:
  cpu: 500m
  memory: 1Gi
~~~

## Cluster installation with kubeadm

### Initialize first control plane

~~~bash
kubeadm init --config kubeadm-config.yaml --upload-certs
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
~~~

### Join additional control plane nodes

Use the `kubeadm join ... --control-plane --certificate-key ...` command emitted by the init step.

### Join worker nodes

Use the worker join command emitted by `kubeadm init` or regenerate it:

~~~bash
kubeadm token create --print-join-command
~~~

## Install Calico CNI

Why Calico here:

- mature network policy support
- common on-prem choice
- can integrate with BGP if the network design evolves

~~~bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
kubectl get pods -n kube-system
~~~

## Ingress design

Use **MetalLB** for bare-metal service IP allocation and **NGINX Ingress Controller** for HTTP routing.

## Ingress traffic flow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    INET[Internet or Internal Clients] --> FW[Firewall]
    FW --> LBIP[MetalLB Service IP]
    LBIP --> NGINX[NGINX Ingress Controller]
    NGINX --> SVC[Kubernetes Service]
    SVC --> PODS[Application Pods]
~~~

## MetalLB config example

~~~yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: production-pool
  namespace: metallb-system
spec:
  addresses:
    - 10.10.50.100-10.10.50.120
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: production-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - production-pool
~~~

## Storage for Kubernetes

### NFS CSI example

~~~yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: cluster.local/nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "true"
reclaimPolicy: Delete
volumeBindingMode: Immediate
~~~

### High-performance local storage note

For data-heavy worker nodes, local-path provisioners or direct-attached performance volumes can complement NFS.

## Networking notes

- node IPs sit on VLAN 60
- Pod CIDR and Service CIDR should not overlap with any routed VLANs
- CoreDNS provides service discovery inside the cluster
- default-deny namespace policies should be standard

### Example default deny NetworkPolicy

~~~yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
~~~

## Security model

### RBAC guidance

- cluster-admin only for core platform operators
- namespace-admin for application teams in their own namespaces
- read-only roles for auditors or support teams

### Pod security

Use **Pod Security Standards: restricted** by default where workloads permit it.

### Secrets

Prefer **sealed-secrets** or **external-secrets** integrated with an enterprise secret store.

## Day-2 operations

### Upgrade strategy

1. upgrade one control plane node at a time
2. upgrade kubeadm first
3. run `kubeadm upgrade apply` or `node`
4. drain and upgrade workers one by one
5. verify workloads and DaemonSets after each step

### etcd backup schedule

~~~bash
export ETCDCTL_API=3
etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --endpoints=https://127.0.0.1:2379 \
  snapshot save /var/backups/etcd-$(date +%F-%H%M).db
~~~

### Maintenance commands

~~~bash
kubectl cordon k8s-w01
kubectl drain k8s-w01 --ignore-daemonsets --delete-emptydir-data
kubectl uncordon k8s-w01
kubeadm certs check-expiration
~~~

## Upgrade workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[Upgrade Window] --> B[Control Plane 1]
    B --> C[Control Plane 2]
    C --> D[Control Plane 3]
    D --> E[Worker Nodes One by One]
    E --> F[Post-Upgrade Validation]
~~~

## Verification checklist

| Check | Command | Healthy Signal |
|-------|---------|----------------|
| Control plane | `kubectl get nodes` | all Ready |
| System pods | `kubectl get pods -A` | core pods healthy |
| API VIP | `curl -k https://10.10.60.10:6443/healthz` | `ok` |
| CNI | `kubectl get pods -n kube-system | grep calico` | running |
| Ingress | curl test URL | expected 200 or app response |
| Storage | PVC bind test | bound and writable |

## Common mistakes

- deploying K8s before lower-layer monitoring exists
- overlapping pod/service CIDRs with routed infrastructure networks
- exposing the API server directly without an HA VIP
- treating every worker as identical when some are data-heavy
- skipping etcd backups because the cluster is “small”


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | kubeadm HA with stacked etcd | lowest software cost and standard upstream pattern | managed distro as default rejected |
| ADR-02 | 3 control planes + API VIP | control-plane resilience | single control plane rejected |
| ADR-03 | Calico + MetalLB + NGINX Ingress | proven bare-metal combo | custom LB/CNI mix rejected |
| ADR-04 | NFS CSI first | simplest PV path for general workloads | complex SDS on day one rejected |

### Integration points

- Node images and OS policy come from [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md) and [06-linux-os-layer.md](./06-linux-os-layer.md).
- Alerting, logging, and incident response rely on [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) and [08-troubleshooting-guide.md](./08-troubleshooting-guide.md).
- Network segmentation and load balancer reachability depend on [02-network-design.md](./02-network-design.md).

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

- Multi-cluster strategy: separate clusters by environment, regulatory boundary, or business unit before you separate by minor technical preference.
- DR options: etcd snapshot restore into warm standby, GitOps-driven cluster rebuild, or active/active multi-cluster for critical apps.
- Compliance impact: PCI/HIPAA usually require stricter namespace isolation, secret handling, audit logging, and image provenance.
- Capacity alerts: pending pods, API latency, etcd DB size, cert expiry, worker disk pressure, controller restart loops.
- Auto-remediation can cordon an unhealthy worker or recycle a failed DaemonSet pod, but control plane surgery stays manual.
- Reassess kubeadm versus commercial distributions when team size, compliance, or multi-cluster sprawl outgrows the DIY operational model.

## Building & Deployment Runbook

1. Build load balancer VMs and validate the API VIP.
2. Prepare all nodes with containerd, kernel settings, and kube packages.
3. Initialize the first control plane and join the remaining nodes.
4. Install networking, load balancer, ingress, and storage components:
   ~~~bash
   kubeadm init --config kubeadm-config.yaml --upload-certs
   kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
   kubectl apply -f metallb-config.yaml
   kubectl get nodes
   ~~~
5. Configure RBAC, Pod Security Standards, sealed-secrets/external-secrets, quotas, and backup jobs.
6. Validate etcd and API health:
   ~~~bash
   ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379 endpoint status --write-out=table
   curl -k https://10.10.60.10:6443/healthz
   kubectl get pods -A
   ~~~
7. Validation gate: run a test deployment, PVC claim, ingress route, and a worker drain/uncordon before declaring the cluster ready.

### Common mistakes to avoid during build

- Thinking “free kubeadm” means low total cost.
- Putting etcd on slow shared disks or overloaded workers.
- Building one cluster for every environment before the team can operate one well.
- Skipping upgrade rehearsals until the first emergency patch.


## Cross-references

- VM factory: [05-vm-provisioning-and-hardening.md](./05-vm-provisioning-and-hardening.md)
- Monitoring and alerting: [07-containers-and-monitoring.md](./07-containers-and-monitoring.md)
- Troubleshooting method: [08-troubleshooting-guide.md](./08-troubleshooting-guide.md)
- Related repo reference: [../Virtual-Setup/07-production-kubernetes-setup.md](../Virtual-Setup/07-production-kubernetes-setup.md)
