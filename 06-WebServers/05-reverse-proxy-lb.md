# Reverse Proxy & Load Balancing

## 5.1 Overview

A reverse proxy sits in front of one or more backend services.

### 📸 Reverse Proxy Architecture
![Reverse Proxy](https://upload.wikimedia.org/wikipedia/commons/6/67/Reverse_proxy_h2g2bob.svg)
> *Source: Wikimedia Commons — Reverse proxy architecture*

Functions:

- Hide backend topology
- Terminate TLS
- Apply rate limits
- Route traffic by host/path
- Load balance requests
- Cache responses
- Add security controls
- Centralize logging

Popular tools:

- Nginx
- HAProxy
- Traefik
- Apache HTTP Server
- Envoy

## 5.2 Reverse Proxy vs Forward Proxy

| Type | Purpose |
|---|---|
| Reverse proxy | Represents servers to clients |
| Forward proxy | Represents clients to servers |

## 5.3 Common Routing Patterns

- Host-based routing
- Path-based routing
- Header-based routing
- Cookie-based routing
- Weighted canary routing

## 5.4 Mermaid Diagram: Load Balancer Architecture

### 📸 Load Balancer
![Load Balancer](https://upload.wikimedia.org/wikipedia/commons/a/a8/Elasticsearch_Cluster_August_2014.png)
> *Source: Wikimedia Commons — Load balanced cluster architecture*

```mermaid
graph TD
    A["Clients"] --> B["Load balancer VIP"]
    B --> C["Web node 1"]
    B --> D["Web node 2"]
    B --> E["Web node 3"]
    C --> F["Shared app or data tier"]
    D --> F
    E --> F
```

## 5.5 Nginx Load Balancing Example

```nginx
upstream web_pool {
    least_conn;
    server 10.0.10.11:8080 max_fails=3 fail_timeout=30s;
    server 10.0.10.12:8080 max_fails=3 fail_timeout=30s;
    server 10.0.10.13:8080 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://web_pool;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 5.6 HAProxy Overview

HAProxy is widely used for high-performance TCP/HTTP load balancing.

Strengths:

- Excellent performance
- Rich health checks
- Advanced routing and stickiness
- Useful metrics and admin socket

## 5.7 HAProxy Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y haproxy
sudo systemctl enable --now haproxy
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y haproxy
sudo systemctl enable --now haproxy
```

## 5.8 HAProxy Basic Configuration

```haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon
    maxconn 50000

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend http_front
    bind *:80
    default_backend app_back

backend app_back
    balance roundrobin
    option httpchk GET /health
    server app1 10.0.20.11:8080 check
    server app2 10.0.20.12:8080 check
```

## 5.9 HAProxy Health Checks

Common types:

- TCP check
- HTTP check
- SSL/TLS check
- Agent check

HTTP health check example:

```haproxy
backend api_back
    option httpchk GET /health
    http-check expect status 200
    server api1 10.0.20.21:8080 check
    server api2 10.0.20.22:8080 check
```

## 5.10 HAProxy Session Persistence

Cookie-based example:

```haproxy
backend app_back
    balance roundrobin
    cookie SERVERID insert indirect nocache
    server app1 10.0.20.11:8080 check cookie app1
    server app2 10.0.20.12:8080 check cookie app2
```

Source-IP stickiness example:

```haproxy
backend app_back
    balance source
    server app1 10.0.20.11:8080 check
    server app2 10.0.20.12:8080 check
```

## 5.11 Traefik Overview

Traefik is a modern reverse proxy popular in container and dynamic environments.

Strengths:

- Dynamic service discovery
- Tight Docker/Kubernetes integration
- Automatic Let's Encrypt support
- Easy routing labels/CRDs

## 5.12 Traefik Static Configuration Example

```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
providers:
  file:
    filename: /etc/traefik/dynamic.yml
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@example.com
      storage: /etc/traefik/acme.json
      httpChallenge:
        entryPoint: web
```

## 5.13 Traefik Dynamic Configuration Example

```yaml
http:
  routers:
    app-router:
      rule: "Host(`app.example.com`)"
      service: app-service
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
  services:
    app-service:
      loadBalancer:
        servers:
          - url: "http://10.0.30.11:8080"
          - url: "http://10.0.30.12:8080"
```

## 5.14 Health Check Design

A good health check should be:

- Fast
- Deterministic
- Cheap
- Representative enough
- Separate from heavy dependencies when appropriate

Levels:

- Liveness
- Readiness
- Deep dependency health

## 5.15 Session Persistence Strategies

Options:

- Cookie-based stickiness
- Source IP hashing
- Consistent hashing
- Shared session store instead of stickiness

Preferred approach for scalable apps:

- Store sessions in Redis or database
- Keep app nodes stateless when possible

## 5.16 X-Forwarded Headers

Critical headers forwarded by proxies:

- `X-Forwarded-For`
- `X-Forwarded-Proto`
- `X-Forwarded-Host`
- `Forwarded`

Applications must trust only known proxies.

## 5.17 WebSocket and Long-Lived Connections

Load balancers must handle:

- Upgrade headers
- Idle timeouts
- Sticky routing when needed
- Connection draining during deploys

## 5.18 TLS Termination Models

| Model | Description | Pros | Cons |
|---|---|---|---|
| Edge termination | TLS ends at load balancer | Simpler | Internal traffic unencrypted unless re-encrypted |
| Re-encryption | TLS at edge and backend | Better security | More overhead |
| Passthrough | LB forwards TLS unchanged | Backend controls TLS | Less layer-7 visibility |

## 5.19 Connection Draining

Connection draining allows in-flight requests to finish before a backend is removed.

Use during:

- Deployments
- Scaling down
- Maintenance windows

## 5.20 Blue/Green and Canary Routing

Patterns:

- Blue/green: full switch between two environments
- Canary: small percentage to new version

HAProxy weighted example:

```haproxy
backend app_back
    balance roundrobin
    server blue1 10.0.40.11:8080 weight 90 check
    server green1 10.0.40.21:8080 weight 10 check
```

## 5.21 Nginx Host-Based Routing Example

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://api_pool;
    }
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://app_pool;
    }
}
```

## 5.22 Nginx Path-Based Routing Example

```nginx
server {
    listen 80;
    server_name example.com;

    location /api/ {
        proxy_pass http://api_pool;
    }

    location / {
        proxy_pass http://web_pool;
    }
}
```

## 5.23 HAProxy ACL Example

```haproxy
frontend http_front
    bind *:80
    acl is_api path_beg /api/
    use_backend api_back if is_api
    default_backend web_back
```

## 5.24 Traefik Middleware Example

```yaml
http:
  middlewares:
    security-headers:
      headers:
        stsSeconds: 31536000
        stsIncludeSubdomains: true
        browserXssFilter: true
        contentTypeNosniff: true
  routers:
    app-router:
      rule: "Host(`app.example.com`)"
      middlewares:
        - security-headers
      service: app-service
```

## 5.25 Operational Best Practices

- Always define health checks
- Separate edge and app logs
- Pass real client IP correctly
- Size timeouts to real workloads
- Avoid relying on sticky sessions unless required
- Use deployment draining
- Monitor backend saturation and error rate

## 5.26 Setting Up a Complete Web Stack

This walkthrough shows a practical production baseline on Ubuntu using:

- Nginx as the edge web server and reverse proxy
- Let's Encrypt for public TLS certificates
- Django served by Gunicorn
- PostgreSQL for the primary relational database
- Redis for cache, sessions, or background task coordination

If you deploy Node.js instead of Django, the same Nginx, PostgreSQL, Redis, systemd, and TLS patterns still apply. The main change is replacing Gunicorn with a Node.js process manager or systemd service.

### 5.26.1 Target Architecture

```mermaid
graph TD
    A["Internet clients"] --> B["Nginx reverse proxy"]
    B --> C["Gunicorn Django app"]
    C --> D["PostgreSQL"]
    C --> E["Redis"]
    B --> F["Let's Encrypt certificate files"]
```

### 5.26.2 Assumptions

This example assumes:

- Ubuntu 22.04 or 24.04
- Public DNS already points `app.example.com` to the server
- You have SSH access with sudo
- Ports 80 and 443 can be reached from the internet
- The application will run as a dedicated non-login system user

### 5.26.3 Install Base Packages

```bash
sudo apt update
sudo apt install -y nginx postgresql postgresql-contrib redis-server python3-venv python3-pip git curl ufw certbot python3-certbot-nginx
```

Enable services:

```bash
sudo systemctl enable --now nginx
sudo systemctl enable --now postgresql
sudo systemctl enable --now redis-server
```

Open firewall ports:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### 5.26.4 Create an Application User and Directory Layout

```bash
sudo adduser --system --group --home /srv/app appuser
sudo mkdir -p /srv/app/releases
sudo mkdir -p /srv/app/shared
sudo mkdir -p /srv/app/shared/media
sudo mkdir -p /srv/app/shared/log
sudo chown -R appuser:appuser /srv/app
```

Recommended layout:

```text
/srv/app/
├── current -> /srv/app/releases/2025-01-14-01
├── releases/
│   └── 2025-01-14-01/
├── shared/
│   ├── media/
│   └── log/
└── venv/
```

Why this layout helps:

- Releases are easy to roll back
- Shared media survives deploys
- The current symlink makes systemd and Nginx configs stable

### 5.26.5 Set Up PostgreSQL

Switch to the PostgreSQL admin user:

```bash
sudo -u postgres psql
```

Create a role and database:

```sql
CREATE ROLE appdbuser WITH LOGIN PASSWORD 'change-this-now';
CREATE DATABASE appdb OWNER appdbuser;
\q
```

Basic checks:

```bash
sudo -u postgres psql -c '\l'
sudo -u postgres psql -c '\du'
```

Production advice:

- Use a strong unique password or local socket auth strategy
- Limit remote PostgreSQL exposure unless truly needed
- Back up database and WAL strategy separately from app code

### 5.26.6 Configure Redis

Redis often handles:

- Cache backend
- Session backend
- Celery broker or queue
- Rate-limit counters

Baseline service verification:

```bash
redis-cli ping
sudo systemctl status redis-server --no-pager
```

Optional hardening ideas:

- Bind only to localhost if no remote clients need it
- Require authentication if remote access exists
- Use persistent storage intentionally, not by accident

### 5.26.7 Deploy the Django Application Code

Clone the application into a release directory:

```bash
sudo -u appuser git clone https://github.com/example/myapp.git /srv/app/releases/2025-01-14-01
sudo ln -sfn /srv/app/releases/2025-01-14-01 /srv/app/current
```

Create a virtual environment:

```bash
sudo -u appuser python3 -m venv /srv/app/venv
sudo -u appuser /srv/app/venv/bin/pip install --upgrade pip wheel setuptools
```

Install dependencies:

```bash
sudo -u appuser /srv/app/venv/bin/pip install -r /srv/app/current/requirements.txt
sudo -u appuser /srv/app/venv/bin/pip install gunicorn psycopg2-binary redis
```

### 5.26.8 Create the Django Environment File

Create `/etc/myapp.env`:

```bash
sudo tee /etc/myapp.env >/dev/null <<'EOF'
DJANGO_SETTINGS_MODULE=config.settings.production
SECRET_KEY=replace-this-with-a-real-secret
DEBUG=False
ALLOWED_HOSTS=app.example.com
DATABASE_URL=postgresql://appdbuser:change-this-now@127.0.0.1:5432/appdb
REDIS_URL=redis://127.0.0.1:6379/0
CSRF_TRUSTED_ORIGINS=https://app.example.com
EOF
```

Protect it:

```bash
sudo chown root:appuser /etc/myapp.env
sudo chmod 640 /etc/myapp.env
```

### 5.26.9 Django Production Settings Checklist

Your production settings should include at least:

- `DEBUG = False`
- `ALLOWED_HOSTS = ["app.example.com"]`
- Secure session cookies
- Secure CSRF cookies
- Database from environment variables
- Static root configured
- Logging to stdout or files intentionally
- Trusted proxy handling if behind an additional load balancer

Example settings highlights:

```python
DEBUG = False
ALLOWED_HOSTS = ["app.example.com"]
SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_REFERRER_POLICY = "strict-origin-when-cross-origin"
STATIC_ROOT = "/srv/app/current/staticfiles"
MEDIA_ROOT = "/srv/app/shared/media"
```

### 5.26.10 Run Database Migrations and Collect Static Files

```bash
sudo -u appuser bash -lc 'set -a && source /etc/myapp.env && set +a && /srv/app/venv/bin/python /srv/app/current/manage.py migrate --noinput'
sudo -u appuser bash -lc 'set -a && source /etc/myapp.env && set +a && /srv/app/venv/bin/python /srv/app/current/manage.py collectstatic --noinput'
```

Optional smoke test:

```bash
sudo -u appuser bash -lc 'set -a && source /etc/myapp.env && set +a && /srv/app/venv/bin/python /srv/app/current/manage.py check --deploy'
```

### 5.26.11 Create the Gunicorn Systemd Service

Create `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=Gunicorn service for myapp
After=network.target postgresql.service redis-server.service
Requires=postgresql.service redis-server.service

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/srv/app/current
EnvironmentFile=/etc/myapp.env
ExecStart=/srv/app/venv/bin/gunicorn \
    --workers 4 \
    --bind 127.0.0.1:8000 \
    --access-logfile - \
    --error-logfile - \
    --capture-output \
    --timeout 60 \
    config.wsgi:application
Restart=always
RestartSec=5
RuntimeDirectory=myapp
StateDirectory=myapp
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
ReadWritePaths=/srv/app/shared /srv/app/current/staticfiles
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Reload and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
sudo systemctl status myapp --no-pager
```

Quick local test:

```bash
curl -I http://127.0.0.1:8000
```

### 5.26.12 Production Nginx Site for Django

Create `/etc/nginx/sites-available/myapp.conf`:

```nginx
upstream myapp_backend {
    server 127.0.0.1:8000;
    keepalive 32;
}

server {
    listen 80;
    listen [::]:80;
    server_name app.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/letsencrypt;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name app.example.com;

    ssl_certificate /etc/letsencrypt/live/app.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:20m;
    ssl_session_tickets off;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location /static/ {
        alias /srv/app/current/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, max-age=2592000, immutable" always;
        access_log off;
    }

    location /media/ {
        alias /srv/app/shared/media/;
        expires 1h;
        add_header Cache-Control "public, max-age=3600" always;
    }

    location = /health {
        access_log off;
        default_type text/plain;
        return 200 'ok';
    }

    location / {
        proxy_pass http://myapp_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Request-ID $request_id;
        proxy_connect_timeout 5s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}
```

Enable the site:

```bash
sudo mkdir -p /var/www/letsencrypt
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/myapp.conf
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

### 5.26.13 Obtain the Let's Encrypt Certificate

```bash
sudo certbot --nginx -d app.example.com
```

Validate renewal:

```bash
sudo certbot renew --dry-run
```

If you want an explicit deploy hook:

```bash
sudo certbot renew --deploy-hook "systemctl reload nginx"
```

### 5.26.14 End-to-End Validation

Check local services:

```bash
sudo systemctl status nginx myapp postgresql redis-server --no-pager
ss -tulpn | grep ':80\|:443\|:8000\|:5432\|:6379'
```

Check the app locally through Nginx:

```bash
curl -I http://127.0.0.1
curl -Ik https://127.0.0.1
curl -k https://127.0.0.1/health
```

Check externally by hostname:

```bash
curl -I https://app.example.com
curl -vk https://app.example.com/
openssl s_client -connect app.example.com:443 -servername app.example.com </dev/null
```

### 5.26.15 Log Locations

| Component | Typical logs |
|---|---|
| Nginx | `/var/log/nginx/access.log`, `/var/log/nginx/error.log` |
| Gunicorn via systemd | `journalctl -u myapp -f` |
| PostgreSQL | `journalctl -u postgresql -f` or distro-specific log path |
| Redis | `journalctl -u redis-server -f` |
| Certbot | `/var/log/letsencrypt/letsencrypt.log` |

### 5.26.16 Deployment Workflow from Here

A clean deploy flow often looks like this:

1. Pull new code into a new release directory.
2. Install dependencies in the virtual environment.
3. Run migrations.
4. Run collectstatic.
5. Update `/srv/app/current` symlink.
6. Restart or reload Gunicorn.
7. Validate health endpoint and homepage.
8. Roll back symlink if validation fails.

### 5.26.17 Backup and Recovery Considerations

At minimum, define backups for:

- PostgreSQL database dumps or physical backups
- Uploaded media under `/srv/app/shared/media`
- Environment file secrets from a secure secret store or backup plan
- Nginx site config and systemd unit files through source control or config management

A stack is not production-ready if restore is untested.

### 5.26.18 Common Failure Modes in This Stack

| Symptom | Likely cause | First command |
|---|---|---|
| Nginx returns `502` | Gunicorn not running or wrong upstream | `systemctl status myapp --no-pager` |
| Django works locally but static files 404 | `collectstatic` missing or bad alias path | `ls -la /srv/app/current/staticfiles` |
| App throws database errors | Bad `DATABASE_URL` or role permissions | `sudo -u postgres psql -c '\l'` |
| Redis-dependent features fail | Redis not running or wrong URL | `redis-cli ping` |
| HTTPS fails | Cert not issued or Nginx config invalid | `sudo nginx -t` and `certbot certificates` |
| Wrong redirect URLs | Missing `SECURE_PROXY_SSL_HEADER` or forwarding headers | Inspect Django settings and Nginx headers |

### 5.26.19 Node.js Variant Notes

If you deploy Node.js instead of Django:

- Replace the virtualenv with a Node.js runtime install
- Replace Gunicorn with a systemd-managed `node` process, PM2, or another supervisor
- Keep PostgreSQL, Redis, Nginx, TLS, firewall, and validation steps mostly unchanged

Example service shape:

```ini
[Service]
User=appuser
Group=appuser
WorkingDirectory=/srv/app/current
EnvironmentFile=/etc/myapp.env
ExecStart=/usr/bin/node /srv/app/current/server.js
Restart=always
RestartSec=5
```

### 5.26.20 Production Readiness Checklist

- Dedicated app user exists.
- PostgreSQL and Redis are enabled and healthy.
- App secrets live outside source control.
- Gunicorn or Node.js service starts on boot.
- Nginx config passes `nginx -t`.
- HTTPS works with a valid public certificate.
- Static and media paths are correct.
- Health endpoint is reachable.
- Backups exist and restores are tested.
- Monitoring and alerting are configured for latency, `5xx`, disk, memory, and certificate expiry.

---

### 12.5 Reverse Proxy Checklist

- Upstream health endpoint exists
- `Host` and `X-Forwarded-*` headers forwarded
- Timeouts match workload
- Rate limiting set if public
- Access logs enabled
- Sticky sessions used only if needed
- Draining plan exists

### 15.3 Scenario: API Reverse Proxy with Nginx

Requirements:

- Proxy to app pool
- Rate limiting
- Health endpoint
- Basic security headers

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=20r/s;

upstream api_pool {
    least_conn;
    server 10.10.1.11:8080 max_fails=3 fail_timeout=10s;
    server 10.10.1.12:8080 max_fails=3 fail_timeout=10s;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options SAMEORIGIN always;

    location = /health {
        access_log off;
        return 200 'ok';
    }

    location / {
        limit_req zone=api_limit burst=50 nodelay;
        proxy_pass http://api_pool;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 3s;
        proxy_read_timeout 30s;
    }
}
```

### 15.4 Scenario: HAProxy in Front of Two App Servers

```haproxy
global
    log /dev/log local0
    daemon

defaults
    mode http
    log global
    option httplog
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend fe_http
    bind *:80
    default_backend be_apps

backend be_apps
    option httpchk GET /health
    http-check expect status 200
    balance leastconn
    server app1 10.20.1.11:8080 check
    server app2 10.20.1.12:8080 check
```

### 17.2 Load Balancer Comparison

| Tool | Strength |
|---|---|
| Nginx | Simple HTTP reverse proxy and edge server |
| HAProxy | Advanced balancing, health checks, performance |
| Traefik | Dynamic modern environments |

### 19.5 Reverse Proxy Reinforcement

- Health checks are essential.
- Stateless backends simplify scaling.
- Sticky sessions are sometimes necessary but often avoidable.
- Connection draining supports safer deployments.
- Host-based and path-based routing are common first steps.
- Trust proxy headers only from known upstreams.
- WebSockets require special header handling.
- Load balancing can improve both scale and resilience.

### 24.12 HAProxy HTTPS Redirect Example

```haproxy
frontend fe_http
    bind *:80
    http-request redirect scheme https code 301 unless { ssl_fc }
    default_backend be_apps
```
