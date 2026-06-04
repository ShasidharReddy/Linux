# 08 - CI/CD and Automation

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                 CI/CD and Automation                        │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This guide shows how to automate the e-commerce platform across VMs, containers, and Kubernetes. It connects the deployment patterns in [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md), [04-docker-compose-ecommerce.md](./04-docker-compose-ecommerce.md), and [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md).

## CI/CD Overview

A mature delivery pipeline should:

- build artifacts predictably
- run tests early
- scan images and dependencies
- package deployment manifests or Helm charts
- promote immutable versions through environments
- support rollback without rebuilding old versions

## Pipeline Stages

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Code[Commit] --> Build[Build]
    Build --> Test[Test]
    Test --> Scan[Security Scan]
    Scan --> Package[Package]
    Package --> Deploy[Deploy]
    Deploy --> Verify[Smoke Test]
~~~

## Jenkins Pipeline

### `Jenkinsfile`

~~~groovy
pipeline {
  agent any
  environment {
    REGISTRY = 'registry.example.internal'
    APP_IMAGE = 'ecommerce/backend'
    WEB_IMAGE = 'ecommerce/frontend'
    VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
    KUBECONFIG_CRED = credentials('kubeconfig-prod')
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t $REGISTRY/$APP_IMAGE:$VERSION -f app/Dockerfile app'
        sh 'docker build -t $REGISTRY/$WEB_IMAGE:$VERSION -f web/Dockerfile web'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run --rm $REGISTRY/$APP_IMAGE:$VERSION php artisan test'
      }
    }
    stage('Security Scan') {
      steps {
        sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $REGISTRY/$APP_IMAGE:$VERSION'
        sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $REGISTRY/$WEB_IMAGE:$VERSION'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push $REGISTRY/$APP_IMAGE:$VERSION'
        sh 'docker push $REGISTRY/$WEB_IMAGE:$VERSION'
      }
    }
    stage('Helm Package') {
      steps {
        sh 'helm dependency update deploy/helm/ecommerce'
        sh 'helm lint deploy/helm/ecommerce'
        sh 'helm package deploy/helm/ecommerce --destination dist/'
      }
    }
    stage('Deploy') {
      steps {
        sh "helm upgrade --install ecommerce deploy/helm/ecommerce -n ecommerce-prod --create-namespace --set image.tag=$VERSION --kubeconfig $KUBECONFIG_CRED"
      }
    }
    stage('Verify') {
      steps {
        sh 'kubectl --kubeconfig $KUBECONFIG_CRED rollout status deploy/backend -n ecommerce-prod --timeout=180s'
        sh 'kubectl --kubeconfig $KUBECONFIG_CRED get ingress -n ecommerce-prod'
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'dist/*.tgz', fingerprint: true
    }
  }
}
~~~

## GitLab CI

### `.gitlab-ci.yml`

~~~yaml
stages:
  - build
  - test
  - scan
  - deploy

variables:
  REGISTRY: registry.example.internal
  APP_IMAGE: $REGISTRY/ecommerce/backend
  WEB_IMAGE: $REGISTRY/ecommerce/frontend
  VERSION: $CI_COMMIT_SHORT_SHA

build:
  stage: build
  script:
    - docker build -t $APP_IMAGE:$VERSION -f app/Dockerfile app
    - docker build -t $WEB_IMAGE:$VERSION -f web/Dockerfile web
    - docker push $APP_IMAGE:$VERSION
    - docker push $WEB_IMAGE:$VERSION

unit_test:
  stage: test
  script:
    - docker run --rm $APP_IMAGE:$VERSION php artisan test

trivy_scan:
  stage: scan
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $APP_IMAGE:$VERSION
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $WEB_IMAGE:$VERSION

deploy_prod:
  stage: deploy
  when: manual
  script:
    - helm upgrade --install ecommerce deploy/helm/ecommerce -n ecommerce-prod --create-namespace --set image.tag=$VERSION
~~~

## GitHub Actions Workflow

### `.github/workflows/ecommerce.yml`

~~~yaml
name: ecommerce-ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-test-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: registry.example.internal
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Build images
        run: |
          docker build -t registry.example.internal/ecommerce/backend:${GITHUB_SHA::7} -f app/Dockerfile app
          docker build -t registry.example.internal/ecommerce/frontend:${GITHUB_SHA::7} -f web/Dockerfile web
      - name: Test backend
        run: docker run --rm registry.example.internal/ecommerce/backend:${GITHUB_SHA::7} php artisan test
      - name: Scan images
        run: |
          trivy image --exit-code 1 --severity HIGH,CRITICAL registry.example.internal/ecommerce/backend:${GITHUB_SHA::7}
          trivy image --exit-code 1 --severity HIGH,CRITICAL registry.example.internal/ecommerce/frontend:${GITHUB_SHA::7}
      - name: Push images
        run: |
          docker push registry.example.internal/ecommerce/backend:${GITHUB_SHA::7}
          docker push registry.example.internal/ecommerce/frontend:${GITHUB_SHA::7}
      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: |
          helm upgrade --install ecommerce deploy/helm/ecommerce \
            -n ecommerce-prod --create-namespace \
            --set image.tag=${GITHUB_SHA::7}
~~~

## Infrastructure as Code

## Terraform for VM Provisioning

### Example `main.tf`

~~~hcl
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
      version = "0.7.6"
    }
  }
}

provider "libvirt" {
  uri = "qemu:///system"
}

resource "libvirt_pool" "vm_pool" {
  name = "ecommerce"
  type = "dir"
  path = "/vmstore/ecommerce"
}

resource "libvirt_volume" "web01" {
  name           = "web01.qcow2"
  pool           = libvirt_pool.vm_pool.name
  base_volume_id = "/var/lib/libvirt/images/ubuntu-2404-template.qcow2"
  size           = 40 * 1024 * 1024 * 1024
}

resource "libvirt_domain" "web01" {
  name   = "web01"
  memory = 4096
  vcpu   = 2

  disk {
    volume_id = libvirt_volume.web01.id
  }

  network_interface {
    network_name = "frontend-net"
  }

  network_interface {
    network_name = "backend-net"
  }

  console {
    type        = "pty"
    target_type = "serial"
    target_port = "0"
  }
}
~~~

### Terraform commands

~~~bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
~~~

## Ansible for Configuration Management

### Inventory `inventory.ini`

~~~ini
[web]
web01 ansible_host=192.168.10.11
web02 ansible_host=192.168.10.12

[app]
app01 ansible_host=192.168.20.21
app02 ansible_host=192.168.20.22

[db]
db01 ansible_host=192.168.30.31
db02 ansible_host=192.168.30.32
~~~

### Playbook `site.yml`

~~~yaml
- name: Configure web tier
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: true
    - name: Deploy nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/ecommerce.conf
        mode: '0644'
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/ecommerce.conf
        dest: /etc/nginx/sites-enabled/ecommerce.conf
        state: link
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded

- name: Configure app tier
  hosts: app
  become: true
  tasks:
    - name: Install PHP and dependencies
      apt:
        name:
          - php-fpm
          - php-mysql
          - git
          - unzip
        state: present
        update_cache: true

- name: Configure database tier
  hosts: db
  become: true
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present
        update_cache: true
~~~

Run it:

~~~bash
ansible-playbook -i inventory.ini site.yml
~~~

## Combined Terraform + Ansible Workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    TF[Terraform Apply] --> Infra[VMs and Networks]
    Infra --> Inventory[Dynamic or Generated Inventory]
    Inventory --> Ansible[Ansible Playbooks]
    Ansible --> App[Configured E-Commerce Stack]
~~~

Suggested sequence:

1. Terraform creates networks, disks, and VMs
2. cloud-init or Terraform outputs IP addresses
3. Ansible consumes those addresses
4. playbooks configure web, app, and DB tiers
5. CI validates connectivity and health

## Helm Charts

### Chart structure

~~~text
deploy/helm/ecommerce/
├── Chart.yaml
├── values.yaml
├── values-staging.yaml
├── values-prod.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   └── migration-job.yaml
~~~

### Example `values.yaml`

~~~yaml
image:
  repository: registry.example.internal/ecommerce/backend
  tag: latest
  pullPolicy: IfNotPresent

replicaCount: 2

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: shop.example.com
      paths:
        - path: /
          pathType: Prefix
~~~

### Environment overrides

`values-prod.yaml` example:

~~~yaml
replicaCount: 4
image:
  tag: 1.4.0
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 2000m
    memory: 2Gi
~~~

### Helm hooks for migrations

~~~yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ecommerce-migrate
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["php", "artisan", "migrate", "--force"]
~~~

### Helm testing

~~~bash
helm lint deploy/helm/ecommerce
helm template ecommerce deploy/helm/ecommerce -f deploy/helm/ecommerce/values-prod.yaml
helm test ecommerce -n ecommerce-prod
~~~

## Deployment Strategies

### Rolling update

Default Kubernetes behavior:

~~~yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
~~~

### Blue-green with service switching

Use separate Deployments and flip the Service selector.

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
    color: blue
~~~

Switch to green by changing `color: green` and applying.

### Canary with Istio or Flagger

~~~yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: backend
  namespace: ecommerce-prod
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  service:
    port: 9000
  analysis:
    interval: 1m
    threshold: 5
    stepWeight: 10
    maxWeight: 50
~~~

### A/B testing with header-based routing

~~~yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: frontend-ab
  namespace: ecommerce-prod
spec:
  hosts:
    - shop.example.com
  http:
    - match:
        - headers:
            x-experiment:
              exact: green
      route:
        - destination:
            host: frontend-green
    - route:
        - destination:
            host: frontend-blue
~~~

## Deployment Strategy Comparison

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Rolling Update] --> A1[Simple and default]
    B[Blue Green] --> B1[Fast rollback]
    C[Canary] --> C1[Low-risk gradual traffic]
    D[AB Testing] --> D1[Controlled user cohorts]
~~~

## Database Migrations

### Flyway example in Kubernetes

~~~yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: flyway-migrate
  namespace: ecommerce-prod
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: flyway
          image: flyway/flyway:10
          args:
            - -url=jdbc:mysql://mysql-0.mysql:3306/ecommerce
            - -user=ecommerce
            - -password=$(DB_PASSWORD)
            - migrate
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ecommerce-secret
                  key: DB_PASSWORD
~~~

### Migration as init container

~~~yaml
initContainers:
  - name: migrate
    image: registry.example.internal/ecommerce/backend:1.4.0
    command: ["php", "artisan", "migrate", "--force"]
~~~

### Rollback strategy

- ensure backward-compatible schema changes first
- use expand-and-contract database migrations
- keep rollback scripts for destructive changes
- tie application rollback to schema compatibility checks

## Operational Tips

- promote one immutable version across environments
- separate CI from CD if you need stronger release governance
- keep Terraform state secure and backed up
- use Ansible vault or an external secret store for infrastructure secrets
- run `helm diff` before production upgrades
- require smoke tests after deploy and before promotion

## Common Pitfalls

- rebuilding images during rollback instead of reusing immutable tags
- letting Terraform and manual changes fight each other
- coupling DB migrations too tightly to application startup
- deploying directly from developer laptops to production clusters
- ignoring security scan failures under release pressure

## Next Steps

- apply production policy controls from [07-production-kubernetes-setup.md](./07-production-kubernetes-setup.md)
- extend multi-site scaling in [09-hybrid-and-scaling.md](./09-hybrid-and-scaling.md)
- reuse VM automation patterns from [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md) when hybrid environments are required

## Summary

Automation is the difference between a platform that only works once and a platform that can be rebuilt safely on demand. Combining CI/CD, Terraform, Ansible, and Helm gives you a repeatable delivery chain from infrastructure creation to application rollout.

[← Back to Virtual Setup](./README.md)
