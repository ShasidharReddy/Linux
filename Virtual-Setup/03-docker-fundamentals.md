# 03 - Docker Fundamentals

<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                    Docker Fundamentals                      │
└──────────────────────────────────────────────────────────────┘
</pre></div>

This document explains the Docker concepts you need before using [04-docker-compose-ecommerce.md](./04-docker-compose-ecommerce.md). The goal is to package web, app, cache, database, and search services into repeatable container images while keeping security, networking, and operations in mind.

## Goals

- install Docker on major Linux distributions
- understand images, containers, volumes, and networks
- write production-friendly Dockerfiles
- connect services with Docker networks and DNS
- use Docker Compose for multi-service e-commerce stacks
- apply security controls such as image scanning and read-only filesystems

## Docker Architecture

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    CLI[Docker CLI] --> Engine[Docker Engine]
    Engine --> Images[Images]
    Engine --> Containers[Containers]
    Engine --> Networks[Networks]
    Engine --> Volumes[Volumes]
    Registry[Registry] --> Images
~~~

## Installation

### Ubuntu

~~~bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
docker --version
~~~

### RHEL / Rocky / AlmaLinux

~~~bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
docker --version
~~~

### CentOS Stream

~~~bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
~~~

### Post-install configuration

~~~bash
sudo usermod -aG docker $USER
newgrp docker
docker run --rm hello-world
~~~

## Core Concepts

### Images

Images are immutable filesystem layers plus metadata. They are built from Dockerfiles and tagged for reuse.

Examples:

~~~bash
docker image pull nginx:1.27-alpine
docker image ls
docker image inspect nginx:1.27-alpine
~~~

### Containers

Containers are running instances of images.

~~~bash
docker run -d --name web -p 8080:80 nginx:1.27-alpine
docker ps
docker logs web
docker exec -it web sh
~~~

### Volumes

Volumes persist data independently of container lifecycles.

~~~bash
docker volume create mysql_data
docker run --rm -v mysql_data:/var/lib/mysql alpine:3.20 ls -la /var/lib/mysql
~~~

### Networks

Networks allow isolated service communication and built-in DNS.

~~~bash
docker network create app_net
docker run -d --name redis --network app_net redis:7-alpine
docker run --rm --network app_net alpine:3.20 ping -c 2 redis
~~~

## Docker Resource Model

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Image[Image] --> Container[Container]
    Volume[Named Volume] --> Container
    Network[Docker Network] --> Container
    Secret[Runtime Secret] --> Container
    Config[Environment Variables] --> Container
~~~

## Dockerfile Best Practices

### General rules

- start from minimal trusted base images
- pin versions where possible
- keep build context small with `.dockerignore`
- use multi-stage builds for compiled or dependency-heavy apps
- run as a non-root user unless the image truly requires root
- avoid baking secrets into images
- use `HEALTHCHECK` for operational visibility

### Layer caching tips

- copy lockfiles before application source
- install dependencies before copying fast-changing code
- consolidate related package-manager operations into one layer
- clean package caches to reduce final image size

## Example Dockerfile - Nginx

~~~dockerfile
FROM nginx:1.27-alpine
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY public/ /usr/share/nginx/html/
RUN addgroup -S app && adduser -S app -G app \
    && chown -R app:app /usr/share/nginx/html
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1/healthz || exit 1
~~~

## Example Dockerfile - PHP-FPM

~~~dockerfile
FROM composer:2.8 AS vendor
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --prefer-dist --no-scripts --no-progress --no-interaction
COPY . .
RUN composer dump-autoload --optimize

FROM php:8.3-fpm-alpine
RUN apk add --no-cache icu-dev oniguruma-dev libzip-dev unzip bash \
    && docker-php-ext-install pdo_mysql mbstring intl zip opcache
WORKDIR /var/www/html
COPY --from=vendor /app /var/www/html
RUN addgroup -g 1000 app && adduser -D -u 1000 -G app app \
    && chown -R app:app /var/www/html
USER app
EXPOSE 9000
HEALTHCHECK --interval=30s --timeout=3s CMD php-fpm -t || exit 1
CMD ["php-fpm", "-F"]
~~~

## Example Dockerfile - Node.js

~~~dockerfile
FROM node:20-alpine AS deps
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --omit=dev

FROM node:20-alpine
WORKDIR /usr/src/app
COPY --from=deps /usr/src/app/node_modules ./node_modules
COPY . .
RUN addgroup -S app && adduser -S app -G app \
    && chown -R app:app /usr/src/app
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1:3000/healthz || exit 1
CMD ["node", "server.js"]
~~~

## Example Dockerfile - MySQL Customization

~~~dockerfile
FROM mysql:8.4
COPY mysql/my.cnf /etc/mysql/conf.d/my.cnf
EXPOSE 3306
HEALTHCHECK --interval=30s --timeout=5s CMD mysqladmin ping -h 127.0.0.1 -uroot -p$MYSQL_ROOT_PASSWORD || exit 1
~~~

## Example Dockerfile - Redis

~~~dockerfile
FROM redis:7.4-alpine
COPY redis/redis.conf /usr/local/etc/redis/redis.conf
EXPOSE 6379
HEALTHCHECK --interval=30s --timeout=3s CMD redis-cli ping || exit 1
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
~~~

## Multi-Stage Builds

Multi-stage builds reduce image size and attack surface by keeping build tools out of the final runtime image.

Build example:

~~~bash
docker build -t registry.example.com/ecommerce/app:1.0.0 -f Dockerfile .
~~~

Inspect size:

~~~bash
docker image ls | grep ecommerce
~~~

## Docker Networking

Docker supports several networking modes.

### Bridge

Default option for most standalone workloads.

~~~bash
docker network create --driver bridge frontend_net
docker network inspect frontend_net
~~~

Use it when:

- most services run on one host
- you need name resolution and service isolation
- you want easy port publishing at the edge

### Host

Shares the host network stack.

~~~bash
docker run --rm --network host nginx:1.27-alpine
~~~

Use carefully:

- excellent performance
- no port namespace isolation
- generally unsuitable for dense multi-service deployments

### Overlay

Used in Docker Swarm or similar multi-host setups.

~~~bash
docker network create --driver overlay --attachable ecommerce_overlay
~~~

### macvlan

Gives containers direct presence on the physical network.

~~~bash
docker network create -d macvlan \
  --subnet=192.168.50.0/24 \
  --gateway=192.168.50.1 \
  -o parent=eth0 pub_net
~~~

Use macvlan when:

- legacy systems require direct L2 visibility
- appliances or monitoring systems expect unique IPs

## Container Networking for E-Commerce

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    User[Client] --> Nginx[Nginx container]
    Nginx --> PHP[PHP-FPM container]
    PHP --> MySQL[MySQL container]
    PHP --> Redis[Redis container]
    PHP --> ES[Elasticsearch container]
~~~

### Service DNS resolution

Containers on the same user-defined network can reach each other by service or container name.

~~~bash
docker network create ecommerce_backend
docker run -d --name mysql --network ecommerce_backend mysql:8.4
docker run --rm --network ecommerce_backend alpine:3.20 getent hosts mysql
~~~

### Custom network design

Suggested separation:

- `frontend_net` for reverse proxy exposure
- `backend_net` for app-to-data traffic
- `logging_net` for log shippers and collectors

## Docker Volumes

### Named volumes

Managed by Docker and portable across Compose projects.

~~~bash
docker volume create redis_data
docker volume inspect redis_data
~~~

### Bind mounts

Map host paths directly into containers.

~~~bash
docker run --rm -v $(pwd)/src:/app alpine:3.20 ls /app
~~~

Trade-offs:

| Type | Best For | Trade-Off |
| --- | --- | --- |
| Named volume | Production persistence | Less transparent to host file browsing |
| Bind mount | Local development | Host path coupling and permission drift |

### Production volume drivers

Examples:

- local driver for small deployments
- NFS-backed plugins for shared data
- cloud block or file drivers on managed platforms
- CSI-backed storage once moving to Kubernetes

## Docker Compose

Compose defines multi-container applications in YAML.

### Full `docker-compose.yml`

~~~yaml
services:
  nginx:
    image: nginx:1.27-alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
      - app_public:/var/www/html/public:ro
    depends_on:
      php:
        condition: service_healthy
    networks:
      - frontend
      - backend
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://127.0.0.1/healthz || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3

  php:
    build:
      context: ./app
      dockerfile: Dockerfile
    environment:
      APP_ENV: production
      DB_HOST: mysql
      DB_DATABASE: ecommerce
      DB_USERNAME: ecommerce
      DB_PASSWORD: ecommercepass
      REDIS_HOST: redis
      ELASTICSEARCH_HOST: http://elasticsearch:9200
    volumes:
      - app_public:/var/www/html/public
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "php-fpm -t || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3

  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: ecommerce
      MYSQL_USER: ecommerce
      MYSQL_PASSWORD: ecommercepass
    command: ["--default-authentication-plugin=mysql_native_password"]
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -uroot -p$$MYSQL_ROOT_PASSWORD || exit 1"]
      interval: 20s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7.4-alpine
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 20s
      timeout: 5s
      retries: 5

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      ES_JAVA_OPTS: "-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es_data:/usr/share/elasticsearch/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://127.0.0.1:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 10

volumes:
  app_public:
  mysql_data:
  redis_data:
  es_data:

networks:
  frontend:
  backend:
~~~

### `docker-compose.override.yml`

~~~yaml
services:
  nginx:
    ports:
      - "8080:80"
  php:
    environment:
      APP_ENV: local
      APP_DEBUG: "true"
    volumes:
      - ./app:/var/www/html
  mysql:
    ports:
      - "3306:3306"
~~~

### Resource limits

~~~yaml
services:
  php:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2048M
        reservations:
          cpus: '1.0'
          memory: 1024M
~~~

Bring the stack up:

~~~bash
docker compose up -d --build
docker compose ps
docker compose logs -f nginx
~~~

## Private Registry Setup

Run a simple internal registry:

~~~bash
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v registry_data:/var/lib/registry \
  registry:2
~~~

Tag and push images:

~~~bash
docker tag ecommerce/app:1.0.0 registry.example.internal:5000/ecommerce/app:1.0.0
docker push registry.example.internal:5000/ecommerce/app:1.0.0
~~~

### Tagging strategy

Recommended tags:

- `1.4.2` for immutable semver release
- `1.4` for minor stream
- `stable` for approved production baseline
- `git-<sha>` for traceability in CI

Avoid using only `latest` in production automation.

## Docker Security

### Scan images with Trivy

~~~bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
sudo mv ./bin/trivy /usr/local/bin/
trivy image nginx:1.27-alpine
trivy image registry.example.internal:5000/ecommerce/app:1.0.0
~~~

### seccomp profiles

Run a container with the default profile explicitly:

~~~bash
docker run --rm --security-opt seccomp=default nginx:1.27-alpine
~~~

### Read-only root filesystem

~~~bash
docker run -d \
  --name secure-nginx \
  --read-only \
  --tmpfs /var/cache/nginx \
  --tmpfs /var/run \
  -p 8081:80 nginx:1.27-alpine
~~~

### Drop capabilities

~~~bash
docker run --rm \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  -p 8082:80 nginx:1.27-alpine
~~~

### Docker Bench Security

~~~bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh
~~~

## CI/CD with Docker

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Code[Source Code] --> Build[Docker Build]
    Build --> Test[Container Tests]
    Test --> Scan[Trivy Scan]
    Scan --> Push[Push to Registry]
    Push --> Deploy[Compose or Kubernetes Deploy]
~~~

## Common Commands

~~~bash
docker build -t ecommerce/web:1.0.0 ./web
docker run --rm -it ecommerce/web:1.0.0 sh
docker compose config
docker compose pull
docker compose up -d
docker system df
docker image prune -f
~~~

## Practical Tips

- keep one Dockerfile per deployable service
- use `.dockerignore` aggressively
- publish only edge ports such as 80 or 443
- keep database and cache services on private networks
- pin base image versions and update them intentionally
- test health checks locally before adding them to automation

## Common Pitfalls

- running everything as root inside containers
- using bind mounts in production without thinking through permissions and backups
- hardcoding secrets in Compose files
- assuming container restart equals application recovery
- skipping image scanning in the build pipeline
- exposing MySQL, Redis, and Elasticsearch directly to the internet

## Where to Go Next

- Build a full Compose stack in [04-docker-compose-ecommerce.md](./04-docker-compose-ecommerce.md).
- Learn Kubernetes primitives in [05-kubernetes-fundamentals.md](./05-kubernetes-fundamentals.md).
- Compare container operations with the VM model in [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md).

## Summary

Docker gives you a portable packaging format and predictable runtime environment for modern e-commerce services. Once you understand images, layers, networks, and persistence, you can move confidently from local development to staging, then into Compose or Kubernetes-based production platforms.

[← Back to Virtual Setup](./README.md)
