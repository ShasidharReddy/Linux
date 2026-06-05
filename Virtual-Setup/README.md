# Virtual Setup

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│           Virtual Environment Setup — E-Commerce            │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This track explains how to design, build, automate, and operate an e-commerce platform across virtual machines, containers, and Kubernetes. It starts with core virtualization concepts, moves into Docker-based packaging, then finishes with production Kubernetes, CI/CD, scaling, and hybrid architecture guidance.

Use this directory when you want a practical, end-to-end reference for:

- learning hypervisors and VM operations
- building a VM-based commerce stack
- packaging services with Docker and Compose
- orchestrating workloads with Kubernetes
- automating delivery with CI/CD and IaC
- preparing for production-grade scale and resilience

## Learning Path

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[VM Basics] --> B[KVM and Libvirt]
    B --> C[VM-Based E-Commerce]
    C --> D[Docker Fundamentals]
    D --> E[Docker Compose Deployment]
    E --> F[Kubernetes Fundamentals]
    F --> G[Kubernetes E-Commerce]
    G --> H[Production Kubernetes]
    H --> I[CI/CD and Automation]
    I --> J[Hybrid and Scaling]
~~~

## What This Track Covers

1. Infrastructure primitives for lab and production setups
2. VM lifecycle management with KVM, vSphere, and Proxmox
3. Multi-tier architecture design for web, app, cache, and database tiers
4. Container image creation, networking, storage, and security
5. Docker Compose deployments for development, staging, and mid-scale production
6. Kubernetes resource design, ingress, autoscaling, and stateful services
7. Production operations such as GitOps, observability, service mesh, DR, and cost control
8. Migration patterns from bare metal to VMs to containers to Kubernetes

## Table of Contents

- [01 - Virtualization Fundamentals](./01-virtualization-fundamentals.md)
- [02 - VM-Based E-Commerce Setup](./02-vm-based-ecommerce-setup.md)
- [03 - Docker Fundamentals](./03-docker-fundamentals.md)
- [04 - Docker Compose E-Commerce](./04-docker-compose-ecommerce.md)
- [05 - Kubernetes Fundamentals](./05-kubernetes-fundamentals.md)
- [06 - Kubernetes E-Commerce Deployment](./06-kubernetes-ecommerce-deployment.md)
- [07 - Production Kubernetes Setup](./07-production-kubernetes-setup.md)
- [08 - CI/CD and Automation](./08-ci-cd-and-automation.md)
- [09 - Hybrid and Scaling](./09-hybrid-and-scaling.md)
- [10 - Architecture Diagrams](./10-architecture-diagrams.md)

## Recommended Reading Order

| Stage | Goal | Primary File | Follow-Up |
| --- | --- | --- | --- |
| 1 | Understand virtual infrastructure | [01](./01-virtualization-fundamentals.md) | [02](./02-vm-based-ecommerce-setup.md) |
| 2 | Build a VM-hosted application stack | [02](./02-vm-based-ecommerce-setup.md) | [08](./08-ci-cd-and-automation.md) |
| 3 | Learn container packaging | [03](./03-docker-fundamentals.md) | [04](./04-docker-compose-ecommerce.md) |
| 4 | Run multi-service apps on a single host or small cluster | [04](./04-docker-compose-ecommerce.md) | [05](./05-kubernetes-fundamentals.md) |
| 5 | Learn Kubernetes objects and operations | [05](./05-kubernetes-fundamentals.md) | [06](./06-kubernetes-ecommerce-deployment.md) |
| 6 | Deploy production-style services on Kubernetes | [06](./06-kubernetes-ecommerce-deployment.md) | [07](./07-production-kubernetes-setup.md) |
| 7 | Add enterprise controls and reliability | [07](./07-production-kubernetes-setup.md) | [09](./09-hybrid-and-scaling.md) |
| 8 | Automate the full platform | [08](./08-ci-cd-and-automation.md) | [07](./07-production-kubernetes-setup.md) |
| 9 | Scale across sites and environments | [09](./09-hybrid-and-scaling.md) | Revisit all |

## VMs vs Containers vs Kubernetes

| Platform | Best Use Case | Strengths | Trade-Offs | Typical E-Commerce Use |
| --- | --- | --- | --- | --- |
| Virtual Machines | Strong isolation, legacy apps, mixed OS workloads | Hypervisor isolation, snapshots, live migration, mature tooling | Heavier than containers, slower provisioning | Database tiers, stateful middleware, legacy PHP or Java stacks |
| Containers | Fast packaging and portability for applications | Fast startup, small footprint, immutable images, repeatable builds | Single-host orchestration is limited without a scheduler | App services, workers, reverse proxies, local dev |
| Kubernetes | Large-scale orchestration and self-healing | Scheduling, autoscaling, rolling updates, service discovery | Higher complexity and operational overhead | Production web/app tiers, APIs, background jobs, multi-team platforms |

## Choosing the Right Model

- Start with **VMs** when you need isolation, simple recovery, or are migrating from physical servers.
- Use **Docker Compose** when you need a compact environment for development, staging, demos, or small traffic production.
- Use **Kubernetes** when you need autoscaling, declarative operations, GitOps, and reliable multi-service lifecycle management.
- Use a **hybrid approach** when databases or compliance-sensitive services stay on VMs while stateless services run in Kubernetes.

## Prerequisites

### Required Skills

- Linux shell basics
- Editing files with `vim`, `nano`, or VS Code remote workflows
- Package management on Ubuntu or RHEL-family systems
- TCP/IP fundamentals, DNS, and HTTP/HTTPS basics
- Basic SQL and web application architecture

### Recommended Host Resources

| Lab Type | CPU | RAM | Storage | Notes |
| --- | --- | --- | --- | --- |
| Lightweight VM lab | 8 cores | 16 GB | 200 GB SSD | Enough for KVM practice and a few small VMs |
| Docker + Compose lab | 8 cores | 16-24 GB | 200 GB SSD | Good for full-stack local testing |
| Kubernetes lab | 12 cores | 32 GB | 300 GB SSD | Better for 3-node clusters and observability tools |
| Production-like sandbox | 16+ cores | 64 GB | 500 GB SSD/NVMe | Useful for multi-node K8s and data services |

### Core Tools

~~~bash
sudo apt-get update
sudo apt-get install -y qemu-kvm libvirt-daemon-system virtinst bridge-utils curl git jq

sudo dnf install -y @virt virt-install libvirt bridge-utils qemu-kvm

docker --version
kubectl version --client
virsh --version
~~~

### Suggested Knowledge Progression

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Host OS Preparation] --> B[Hypervisors and VM Operations]
    B --> C[Multi-VM Application Design]
    C --> D[Container Images and Compose]
    D --> E[Kubernetes Resources]
    E --> F[Production Platform Patterns]
~~~

## How to Use This Directory

- Read [01](./01-virtualization-fundamentals.md) before provisioning infrastructure.
- Use [02](./02-vm-based-ecommerce-setup.md) if you want a full VM deployment flow.
- Use [03](./03-docker-fundamentals.md) and [04](./04-docker-compose-ecommerce.md) for container-based environments.
- Use [05](./05-kubernetes-fundamentals.md) before applying the manifests in [06](./06-kubernetes-ecommerce-deployment.md).
- Use [07](./07-production-kubernetes-setup.md) and [08](./08-ci-cd-and-automation.md) when hardening, automating, and scaling the platform.
- Finish with [09](./09-hybrid-and-scaling.md) for hybrid architecture and migration planning.

## Common Pitfalls

- Overcommitting host RAM when running multiple VMs and containers together
- Mixing dev and prod configuration patterns in the same Compose or Kubernetes manifests
- Storing secrets directly inside application images or Git repositories
- Skipping health checks and backup routines for MySQL, Redis, and Elasticsearch
- Underestimating DNS, TLS, and network segmentation requirements
- Scaling application pods without validating database, queue, and cache capacity

## Suggested Lab Progression

1. Build one VM manually with KVM.
2. Clone it into web, app, and DB roles.
3. Rebuild the same stack with Docker Compose.
4. Move stateless services into Kubernetes.
5. Add GitOps, monitoring, and production controls.
6. Test backup, restore, rollback, and scale events.

## Document Conventions

- Commands are shown as runnable Linux CLI examples.
- Configurations are complete enough to adapt directly.
- Mermaid diagrams provide topology and workflow context.
- Each file links back here for fast navigation.
- Related files are cross-referenced throughout the track.

## Next Step

Start with [01 - Virtualization Fundamentals](./01-virtualization-fundamentals.md) to learn the platform concepts that everything else builds on.
