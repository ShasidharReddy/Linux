# 04 - Docker Compose E-Commerce
<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                 Docker Compose E-Commerce                   │
└──────────────────────────────────────────────────────────────┘
</pre></div>
This guide applies the Docker concepts from [03-docker-fundamentals.md](./03-docker-fundamentals.md) to a practical e-commerce deployment. It covers a basic all-in-one setup, an intermediate production-style design, hardening, scaling, maintenance, and Traefik-based dynamic routing.
## When Compose Fits Best
Use Docker Compose when you need:
- development or staging environments with realistic topology
- small production systems on one host or a tightly managed pair of hosts
- repeatable application packaging without full Kubernetes overhead
- easy service overrides per environment
If you need large-scale self-healing, autoscaling, or GitOps workflows, continue later with [05-kubernetes-fundamentals.md](./05-kubernetes-fundamentals.md) and [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md).
## Architecture Comparison
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    subgraph Basic[Basic Compose]
        B1[Nginx]
        B2[App]
        B3[MySQL]
        B4[Redis]
    end
    subgraph Prod[Production Compose]
        P1[Traefik or Nginx]
        P2[Web xN]
        P3[App xN]
        P4[MySQL]
        P5[Redis]
        P6[Elasticsearch]
        P7[RabbitMQ]
        P8[Logging Stack]
    end
~~~
## Basic Setup - Development or Small Staging
The following design fits roughly 50 visitors/day or internal QA traffic.
### Directory Layout
~~~text
compose-ecommerce/
├── .env
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── mysql/
│   └── init.sql
├── redis/
│   └── redis.conf
└── app/
    ├── Dockerfile
    └── .dockerignore
~~~
### `.env`
~~~dotenv
COMPOSE_PROJECT_NAME=ecommerce
APP_ENV=staging
APP_DEBUG=false
APP_URL=http://localhost
DB_DATABASE=ecommerce
DB_USERNAME=ecommerce
DB_PASSWORD=ecommercepass
MYSQL_ROOT_PASSWORD=rootpass
REDIS_PORT=6379
ELASTICSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
~~~
### Basic `docker-compose.yml`
~~~yaml
services:
  nginx:
    image: nginx:1.27-alpine
    depends_on:
      app:
        condition: service_healthy
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
      - app_public:/var/www/html/public:ro
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://127.0.0.1/healthz || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    environment:
      APP_ENV: ${APP_ENV}
      APP_DEBUG: ${APP_DEBUG}
      APP_URL: ${APP_URL}
      DB_HOST: mysql
      DB_DATABASE: ${DB_DATABASE}
      DB_USERNAME: ${DB_USERNAME}
      DB_PASSWORD: ${DB_PASSWORD}
      REDIS_HOST: redis
      REDIS_PORT: ${REDIS_PORT}
    volumes:
      - app_public:/var/www/html/public
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "php-fpm -t || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    command: ["--default-authentication-plugin=mysql_native_password"]
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -uroot -p$$MYSQL_ROOT_PASSWORD || exit 1"]
      interval: 20s
      timeout: 5s
      retries: 10
  redis:
    image: redis:7.4-alpine
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 20s
      timeout: 5s
      retries: 5
volumes:
  app_public:
  mysql_data:
  redis_data:
networks:
  frontend:
  backend:
~~~
### Basic Nginx `nginx/default.conf`
~~~nginx
upstream app_backend {
    server app:9000;
}
server {
    listen 80;
    server_name _;
    root /var/www/html/public;
    index index.php index.html;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location /healthz {
        access_log off;
        return 200 'ok';
        add_header Content-Type text/plain;
    }
    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass app_backend;
    }
}
~~~
### MySQL seed `mysql/init.sql`
~~~sql
CREATE TABLE IF NOT EXISTS healthcheck (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(64) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO healthcheck (name) VALUES ('compose-bootstrap');
~~~
### Redis config `redis/redis.conf`
~~~ini
bind 0.0.0.0
appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000
maxmemory 256mb
maxmemory-policy allkeys-lru
~~~
### Start the stack
~~~bash
docker compose up -d --build
docker compose ps
curl -I http://localhost/healthz
~~~
## Volume Management for Persistence
Use named volumes for stateful services.
~~~bash
docker volume ls
docker volume inspect compose-ecommerce_mysql_data
docker volume inspect compose-ecommerce_redis_data
~~~
Backup volumes with temporary utility containers:
~~~bash
mkdir -p backups
docker run --rm \
  -v compose-ecommerce_mysql_data:/source:ro \
  -v $(pwd)/backups:/backup \
  alpine:3.20 sh -c "cd /source && tar czf /backup/mysql-data-$(date +%F).tar.gz ."
~~~
## Intermediate Setup - 1K to 10K Visitors/Day
At this stage you should separate concerns and add operational services.
### Components
- Nginx reverse proxy with TLS
- multiple app containers
- MySQL with tuned configuration
- Redis with AOF and RDB persistence
- Elasticsearch for catalog and search
- RabbitMQ for asynchronous jobs
- worker containers for queues and emails
- Fluentd shipping logs to EFK or another log backend
## Multi-Compose Structure
~~~text
compose-prod/
├── .env
├── compose.yaml
├── compose.prod.yaml
├── nginx/
├── traefik/
├── mysql/
├── redis/
├── fluentd/
└── monitoring/
~~~
### Base `compose.yaml`
~~~yaml
services:
  app:
    image: registry.example.internal/ecommerce/app:1.4.0
    env_file: .env
    networks: [backend, data]
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://127.0.0.1:9000 || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3
  worker:
    image: registry.example.internal/ecommerce/app:1.4.0
    command: ["php", "artisan", "queue:work", "--sleep=1", "--tries=3"]
    env_file: .env
    networks: [backend, data]
    restart: unless-stopped
  mysql:
    image: mysql:8.4
    env_file: .env
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf:ro
    networks: [data]
    restart: unless-stopped
  redis:
    image: redis:7.4-alpine
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
    networks: [data]
    restart: unless-stopped
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      ES_JAVA_OPTS: "-Xms2g -Xmx2g"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    networks: [data]
    restart: unless-stopped
  rabbitmq:
    image: rabbitmq:3.13-management
    environment:
      RABBITMQ_DEFAULT_USER: ecommerce
      RABBITMQ_DEFAULT_PASS: rabbitpass
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks: [backend, data]
    restart: unless-stopped
volumes:
  mysql_data:
  redis_data:
  es_data:
  rabbitmq_data:
networks:
  frontend:
  backend:
  data:
~~~
### Production overlay `compose.prod.yaml`
~~~yaml
services:
  nginx:
    image: nginx:1.27-alpine
    depends_on:
      app:
        condition: service_started
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
      - app_public:/var/www/html/public:ro
    networks: [frontend, backend]
    restart: unless-stopped
  app:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2048M
        reservations:
          cpus: '1.0'
          memory: 1024M
  worker:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1024M
  mysql:
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h 127.0.0.1 -uroot -p$$MYSQL_ROOT_PASSWORD || exit 1"]
      interval: 20s
      timeout: 5s
      retries: 10
  fluentd:
    image: fluent/fluentd:v1.17-debian-1
    volumes:
      - ./fluentd/fluent.conf:/fluentd/etc/fluent.conf:ro
    networks: [backend]
    restart: unless-stopped
volumes:
  app_public:
~~~
Start it:
~~~bash
docker compose -f compose.yaml -f compose.prod.yaml up -d
~~~
## Production Nginx Reverse Proxy with SSL
Create `nginx/nginx.conf`:
~~~nginx
worker_processes auto;
events { worker_connections 4096; }
http {
    include /etc/nginx/mime.types;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    upstream app_backend {
        least_conn;
        server app:9000;
    }
    server {
        listen 80;
        server_name shop.example.com;
        return 301 https://$host$request_uri;
    }
    server {
        listen 443 ssl http2;
        server_name shop.example.com;
        ssl_certificate /etc/nginx/certs/fullchain.pem;
        ssl_certificate_key /etc/nginx/certs/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        root /var/www/html/public;
        index index.php index.html;
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
        location /healthz {
            return 200 'ok';
            add_header Content-Type text/plain;
        }
        location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_pass app_backend;
        }
    }
}
~~~
## MySQL Tuning with Custom `my.cnf`
Create `mysql/my.cnf`:
~~~ini
[mysqld]
skip-name-resolve
max_connections = 500
innodb_buffer_pool_size = 2G
innodb_buffer_pool_instances = 2
innodb_log_file_size = 512M
innodb_flush_method = O_DIRECT
innodb_flush_log_at_trx_commit = 1
max_allowed_packet = 64M
slow_query_log = 1
long_query_time = 1
~~~
## Redis Persistence
Create `redis/redis.conf`:
~~~ini
bind 0.0.0.0
appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000
tcp-keepalive 300
maxmemory 1024mb
maxmemory-policy allkeys-lru
~~~
## Elasticsearch Service
Useful checks:
~~~bash
curl http://localhost:9200/_cluster/health?pretty
curl http://localhost:9200/_cat/indices?v
~~~
## RabbitMQ for Async Processing
Open the management UI:
~~~bash
curl -u ecommerce:rabbitpass http://localhost:15672/api/overview
~~~
Worker example command:
~~~yaml
worker:
  image: registry.example.internal/ecommerce/app:1.4.0
  command: ["php", "artisan", "queue:work", "rabbitmq", "--sleep=1", "--tries=3"]
~~~
## Logging with Fluentd and EFK
Create `fluentd/fluent.conf`:
~~~conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match **>
  @type stdout
</match>
~~~
Attach app logging driver example:
~~~yaml
app:
  logging:
    driver: fluentd
    options:
      fluentd-address: localhost:24224
      tag: ecommerce.app
~~~
## Production Hardening
### Resource limits
Use Compose resource constraints where supported by the runtime.
~~~yaml
services:
  nginx:
    mem_limit: 512m
    cpus: 1.0
  app:
    mem_limit: 2g
    cpus: 2.0
~~~
### Restart policies
~~~yaml
restart: unless-stopped
~~~
### Health checks for all services
Add health checks to:
- Nginx
- app
- MySQL
- Redis
- Elasticsearch
- RabbitMQ
- workers where possible
### Log rotation
~~~yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "5"
~~~
### Secrets management
Compose standalone support for secrets is limited compared with Swarm, but you can still use files:
~~~yaml
services:
  app:
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt
~~~
### Network segmentation
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    Internet --> Frontend[frontend network]
    Frontend --> Nginx[Nginx or Traefik]
    Nginx --> Backend[backend network]
    Backend --> App[App and Workers]
    App --> Data[data network]
    Data --> MySQL[MySQL]
    Data --> Redis[Redis]
    Data --> ES[Elasticsearch]
    Data --> MQ[RabbitMQ]
~~~
Recommended layout:
- `frontend`: reverse proxy only
- `backend`: web/app/workers
- `data`: databases, cache, queue, search
## Scaling with Compose
Scale web services:
~~~bash
docker compose up -d --scale app=3
~~~
Note:
- scaling `nginx` or `app` works well for stateless services
- MySQL requires replication or external HA design rather than simple scale-up copies
## Traefik as Dynamic Reverse Proxy
### Compose definition
~~~yaml
services:
  traefik:
    image: traefik:v3.1
    command:
      - --api.dashboard=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.le.acme.email=admin@example.com
      - --certificatesresolvers.le.acme.storage=/letsencrypt/acme.json
      - --certificatesresolvers.le.acme.httpchallenge=true
      - --certificatesresolvers.le.acme.httpchallenge.entrypoint=web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/letsencrypt:/letsencrypt
    networks: [frontend]
  web:
    image: registry.example.internal/ecommerce/web:1.4.0
    labels:
      - traefik.enable=true
      - traefik.http.routers.web.rule=Host(`shop.example.com`)
      - traefik.http.routers.web.entrypoints=websecure
      - traefik.http.routers.web.tls.certresolver=le
      - traefik.http.services.web.loadbalancer.server.port=80
    networks: [frontend, backend]
~~~
### Scaling architecture
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    User[Users] --> T[Traefik]
    T --> W1[web/app replica 1]
    T --> W2[web/app replica 2]
    T --> W3[web/app replica 3]
    W1 --> Data[Shared data services]
    W2 --> Data
    W3 --> Data
~~~
## Maintenance Operations
### Zero-downtime deployment pattern
1. build and push a new app image
2. pull the image onto the host
3. start one additional replica with the new image if capacity allows
4. update the remaining replicas gradually
5. run post-deploy checks
6. remove old replicas
Example:
~~~bash
docker compose pull app worker
docker compose up -d --no-deps --scale app=4 app
sleep 10
docker compose up -d --no-deps app worker
~~~
### Database migration strategy
Run migrations from a one-off container before app traffic shifts:
~~~bash
docker compose run --rm app php artisan migrate --force
~~~
### Backup containers
MySQL backup example:
~~~bash
mkdir -p backups
docker compose exec -T mysql sh -c 'exec mysqldump -uroot -p$MYSQL_ROOT_PASSWORD --all-databases' > backups/mysql-$(date +%F).sql
~~~
Redis backup example:
~~~bash
docker compose exec redis redis-cli BGSAVE
~~~
### Monitoring with cAdvisor and Prometheus
~~~yaml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.49.1
    ports:
      - "8081:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
  prometheus:
    image: prom/prometheus:v2.54.1
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
~~~
Prometheus config:
~~~yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: cadvisor
    static_configs:
      - targets: ["cadvisor:8080"]
~~~
## Validation Checklist
~~~bash
docker compose config
docker compose ps
docker compose logs --tail=100 nginx
docker compose exec mysql mysql -uecommerce -p${DB_PASSWORD} -e 'SHOW DATABASES;'
docker compose exec redis redis-cli ping
curl -I http://localhost/healthz
~~~
## Common Pitfalls
- scaling stateful services like MySQL as if they were stateless app containers
- mounting writable source code into production containers
- relying on one huge Compose file without environment overlays
- skipping health checks and restart policies
- failing to segment networks between edge and data services
- forgetting backup and restore testing
## Where to Go Next
- Learn the scheduler model in [05-kubernetes-fundamentals.md](./05-kubernetes-fundamentals.md).
- Build the equivalent orchestrated stack in [06-kubernetes-ecommerce-deployment.md](./06-kubernetes-ecommerce-deployment.md).
- Revisit image design in [03-docker-fundamentals.md](./03-docker-fundamentals.md) if your Compose setup becomes hard to maintain.
## Summary
Docker Compose is a strong platform for e-commerce development, staging, and moderate production workloads when you design around service boundaries, persistence, backups, and edge routing. It gives you a practical bridge between simple single-host environments and full Kubernetes orchestration.
[← Back to Virtual Setup](./README.md)
