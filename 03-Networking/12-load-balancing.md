# Load Balancing

Load balancers distribute traffic across multiple backend servers and improve scalability and availability.

## 11.1 Load balancing concepts

Core capabilities:

- Traffic distribution
- Health checks
- TLS termination
- Session persistence
- Failover
- Reverse proxying

## 11.2 Common Linux load balancers

- HAProxy
- Nginx
- Keepalived for VIP failover

## 11.3 HAProxy overview

HAProxy is widely used for high-performance TCP and HTTP load balancing.

## 11.4 Basic HAProxy config structure

Files commonly live at:

```text
/etc/haproxy/haproxy.cfg
```

Sections include:

- `global`
- `defaults`
- `frontend`
- `backend`
- `listen`

## 11.5 Example HAProxy HTTP load balancer

```haproxy
global
    log /dev/log local0
    daemon
    maxconn 5000

defaults
    log global
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend http_in
    bind *:80
    default_backend web_pool

backend web_pool
    balance roundrobin
    option httpchk GET /health
    server web1 10.10.20.11:80 check
    server web2 10.10.20.12:80 check
```

## 11.6 Example HAProxy TCP load balancer

```haproxy
frontend db_in
    bind *:5432
    mode tcp
    default_backend pg_pool

backend pg_pool
    mode tcp
    balance leastconn
    server db1 10.10.30.11:5432 check
    server db2 10.10.30.12:5432 check backup
```

## 11.7 Validate HAProxy config

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

## 11.8 Nginx as a load balancer

Nginx can proxy traffic to upstreams.

Example:

```nginx
upstream app_backend {
    server 10.10.20.11:8080;
    server 10.10.20.12:8080;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 11.9 Common balancing algorithms

| Algorithm | Description |
|---|---|
| Round robin | Rotate across backends |
| Least connections | Prefer least busy backend |
| Source hash | Sticky by client source |
| URI hash | Sticky by request URI |
| Random | Random distribution |

## 11.10 Health checks

Health checks determine if a backend should receive traffic.

Types:

- TCP connect
- HTTP status check
- SSL handshake
- Custom endpoint checks

## 11.11 Session persistence

Some applications need stickiness.

Methods:

- Cookie-based persistence
- Source-IP hashing
- App-layer session replication

Prefer stateless apps when possible.

## 11.12 TLS termination at the load balancer

Benefits:

- Centralized certificate management
- Offload crypto from app servers
- Add HTTP headers like `X-Forwarded-Proto`

Remember to secure LB-to-backend traffic if needed.

## 11.13 Keepalived overview

Keepalived provides VRRP-based failover for a virtual IP.

Common design:

- Two load balancers
- One active, one standby
- Shared VIP moves on failure

## 11.14 Example Keepalived config

```conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass StrongSecret
    }
    virtual_ipaddress {
        10.10.20.100/24
    }
}
```

## 11.15 Keepalived with health checks

Keepalived can track services and lower priority if HAProxy or Nginx fails.

This improves failover behavior.

## 11.16 Load balancer troubleshooting

Check:

- Frontend listener active?
- Backend health up?
- Firewall allows client and backend traffic?
- TLS cert valid?
- App trusts forwarded headers?

Useful commands:

```bash
ss -tulpn | grep -E ':80|:443|:5432'
curl -I http://127.0.0.1
curl -I http://10.10.20.11/health
journalctl -u haproxy
journalctl -u keepalived
```

## 11.17 High availability design notes

Combine:

- HAProxy or Nginx for balancing
- Keepalived for VIP failover
- Monitoring for proactive alerting
- Redundant upstream paths

## 11.18 Example architecture pattern

- VIP: `10.10.20.100`
- LB1: `10.10.20.21`
- LB2: `10.10.20.22`
- Backends: `10.10.20.11`, `10.10.20.12`

Flow:

1. Client sends request to VIP.
2. Keepalived ensures active node owns VIP.
3. HAProxy or Nginx selects healthy backend.
4. Backend responds through load balancer.

## 11.19 Security best practices for load balancers

- Restrict admin stats pages
- Enforce TLS best practices
- Limit source networks for management
- Validate health endpoints carefully
- Monitor backend failures and response times

## 11.20 Summary

Load balancing is not only about spreading traffic. It is also about health, failover, observability, and safe traffic mediation.

---

<a id="practical-network-scenarios"></a>

---

## 12.4 Configure Nginx as TCP load balancer

Objective:

- Balance PostgreSQL or another TCP service across backends
- Keep the frontend endpoint stable during backend maintenance

Example `stream` config:

```nginx
stream {
    upstream postgres_backend {
        least_conn;
        server 10.20.30.31:5432 max_fails=3 fail_timeout=10s;
        server 10.20.30.32:5432 max_fails=3 fail_timeout=10s;
    }

    server {
        listen 5432;
        proxy_connect_timeout 3s;
        proxy_timeout 1h;
        proxy_pass postgres_backend;
    }
}
```

Checks:

```bash
sudo nginx -t
ss -tlnp | grep 5432
nc -zv lb1.example.com 5432
```

---

# Load Balancer Deployment Checklist

## X.4 Minimal load balancer checklist

1. Install HAProxy or Nginx.
2. Define backend pool.
3. Add health checks.
4. Bind frontend listener.
5. Open firewall port.
6. Test direct backend responses.
7. Test through load balancer.

---

# Final Summary

This guide covered Linux networking from the ground up:

- Network models and addressing
- Linux interface and route configuration
- DNS resolver behavior and server setup
- Firewall technologies and packet flow
- SSH usage and secure remote access
- Troubleshooting tools and structured workflows
- Common network services
- Bonding, VLANs, and bridging
- Namespaces, NAT, forwarding, and traffic shaping
- VPNs with OpenVPN and WireGuard
- Load balancing with HAProxy, Nginx, and Keepalived

Use it as both a learning guide and an operational reference.
