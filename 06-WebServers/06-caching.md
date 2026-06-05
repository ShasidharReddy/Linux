# Caching

## 6.1 Overview

Caching reduces latency and backend load by reusing stored responses or objects.

Cache layers:

- Browser cache
- Reverse proxy cache
- Application cache
- Object cache
- Database query cache equivalents
- CDN edge cache

## 6.2 Cache Concepts

| Term | Meaning |
|---|---|
| HIT | Response served from cache |
| MISS | Cache had no usable entry |
| BYPASS | Cache intentionally skipped |
| EXPIRED | Entry exists but is stale |
| REVALIDATED | Freshness confirmed with origin |
| PURGE | Manual removal from cache |

## 6.3 HTTP Cache Headers

Important headers:

- `Cache-Control`
- `Expires`
- `ETag`
- `Last-Modified`
- `Age`
- `Vary`

### Common Cache-Control Values

| Directive | Meaning |
|---|---|
| `public` | Can be cached by shared caches |
| `private` | Only browser/private caches should store |
| `no-store` | Do not store at all |
| `no-cache` | Must revalidate before use |
| `max-age=3600` | Fresh for 3600 seconds |
| `s-maxage=3600` | Shared cache freshness |
| `immutable` | Asset will not change during freshness window |

## 6.4 Caching Strategy by Content Type

| Content Type | Typical Policy |
|---|---|
| Versioned JS/CSS | Long TTL + immutable |
| Images | Long TTL |
| HTML pages | Short TTL or revalidate |
| Authenticated API data | Usually private or no-store |
| Public API responses | Short shared cache if safe |

## 6.5 Nginx Proxy Cache Recap

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=site_cache:100m max_size=10g inactive=60m use_temp_path=off;

server {
    listen 80;
    server_name cache.example.com;

    location / {
        proxy_pass http://origin_pool;
        proxy_cache site_cache;
        proxy_cache_valid 200 5m;
        proxy_cache_valid 301 302 10m;
        proxy_cache_valid 404 1m;
        add_header X-Cache-Status $upstream_cache_status always;
    }
}
```

## 6.6 Varnish Overview

Varnish is a specialized HTTP accelerator.

Strengths:

- Excellent performance
- Flexible VCL language
- Common for caching dynamic sites

## 6.7 Varnish Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y varnish
sudo systemctl enable --now varnish
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y varnish
sudo systemctl enable --now varnish
```

## 6.8 Basic Varnish Configuration

Backend definition in `/etc/varnish/default.vcl`:

```vcl
vcl 4.1;

backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {
    if (req.method != "GET" && req.method != "HEAD") {
        return (pass);
    }
}
```

## 6.9 Varnish Pass on Authenticated Requests

```vcl
sub vcl_recv {
    if (req.http.Authorization || req.http.Cookie) {
        return (pass);
    }
}
```

## 6.10 Varnish TTL Example

```vcl
sub vcl_backend_response {
    if (bereq.url ~ "\.(css|js|jpg|png|svg)$") {
        set beresp.ttl = 24h;
    } else {
        set beresp.ttl = 5m;
    }
}
```

## 6.11 Purging in Varnish

Simple purge example:

```vcl
acl purge {
    "127.0.0.1";
    "10.0.0.0"/24;
}

sub vcl_recv {
    if (req.method == "PURGE") {
        if (!client.ip ~ purge) {
            return (synth(405, "Not allowed"));
        }
        return (purge);
    }
}
```

## 6.12 Redis as Cache

Redis is an in-memory key-value store commonly used for:

- Sessions
- Page fragments
- Object cache
- Rate limiting state
- Queues

### Installation

```bash
sudo apt update
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
```

### Basic Redis Hardening

- Bind to localhost or private networks only
- Require authentication if network exposure exists
- Disable dangerous commands if needed
- Use firewall restrictions
- Enable TLS if required in your deployment

## 6.13 Memcached as Cache

Memcached is a lightweight distributed memory object cache.

Good for:

- Simple ephemeral caching
- Read-heavy workloads

Basic install:

```bash
sudo apt update
sudo apt install -y memcached
sudo systemctl enable --now memcached
```

## 6.14 CDN Concepts

A CDN provides geographically distributed edge caches.

Benefits:

- Reduced latency for global users
- Offloaded origin traffic
- DDoS absorption capabilities
- Edge TLS termination

Common CDN features:

- Edge caching
- WAF
- Bot mitigation
- Image optimization
- Geo routing

## 6.15 Cache Invalidation Strategies

Hard problem areas:

- Purge exact URL
- Purge by tag/key pattern
- Versioned asset filenames
- Short TTLs with background refresh
- Event-driven invalidation

Best practice for static assets:

- Use fingerprinted filenames such as `app.abcdef.js`

## 6.16 Stale Content Strategies

Useful patterns:

- `stale-if-error`
- `stale-while-revalidate`
- Proxy serving stale on origin failure

## 6.17 Common Caching Mistakes

- Caching personalized content publicly
- Ignoring `Vary` requirements
- Not segmenting by auth state
- Excessively long TTLs on HTML
- No purge strategy
- No observability of HIT ratio

## 6.18 Practical Nginx Cache Example with Stale Support

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api_cache:50m max_size=2g inactive=30m use_temp_path=off;

server {
    listen 80;
    server_name api-cache.example.com;

    location /public-api/ {
        proxy_pass http://api_pool;
        proxy_cache api_cache;
        proxy_cache_valid 200 60s;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        add_header X-Cache-Status $upstream_cache_status always;
    }
}
```

## 6.19 Monitor Cache Effectiveness

Track:

- HIT ratio
- MISS ratio
- Backend origin load
- Cache storage usage
- Evictions
- Response time distribution

## 6.20 Cache Best Practices Summary

- Cache safe public content aggressively
- Never cache sensitive personalized responses unless carefully segmented
- Use versioned asset names
- Make cache status visible in headers or metrics
- Define purge and stale behavior intentionally

---

### 6.3 Cache Tool Comparison
| Tool | Best Use |
|---|---|
| Nginx cache | Integrated simple reverse proxy caching |
| Varnish | Dedicated HTTP acceleration |
| Redis | Object/session cache |
| Memcached | Simple in-memory object cache |
| CDN | Global edge caching |

### 6.6 Caching Reinforcement
- Long TTLs suit versioned assets.
- Dynamic personalized pages require caution.
- `Cache-Control` matters more than guesswork.
- Stale serving can improve resilience during backend failure.
- Observe hit ratio, not just configuration intent.
- Invalidating cache precisely is often preferable to global purges.
