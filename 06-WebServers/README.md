# Linux Web Servers & Services Guide


---

## 🎬 Web Request Lifecycle — Animated Workflow

```mermaid
graph LR
    subgraph REQUEST["🌐 HTTP Request Flow"]
        direction LR
        C["🖥️ Client"]:::client --> DNS["🌐 DNS"]:::dns
        DNS --> LB["⚖️ Load Balancer"]:::lb
        LB --> N1["🟢 Nginx/Apache"]:::web
        LB --> N2["🟢 Nginx/Apache"]:::web
        N1 --> APP["⚙️ App Server"]:::app
        N2 --> APP
        APP --> DB["🗄️ Database"]:::db
        APP --> CACHE["⚡ Cache (Redis)"]:::cache
        DB --> APP
        CACHE --> APP
        APP --> N1
        N1 --> C
    end

    subgraph SSL["🔐 TLS Handshake"]
        direction TB
        S1["👋 ClientHello"]:::hello --> S2["👋 ServerHello"]:::hello
        S2 --> S3["📜 Certificate"]:::cert
        S3 --> S4["🔑 Key Exchange"]:::key
        S4 --> S5["✅ Encrypted Channel"]:::done
    end

    classDef client fill:#2196F3,stroke:#1565C0,color:#fff
    classDef dns fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef lb fill:#FF9800,stroke:#E65100,color:#fff,stroke-width:3px
    classDef web fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef app fill:#e94560,stroke:#b71c1c,color:#fff
    classDef db fill:#0f3460,stroke:#16213e,color:#fff
    classDef cache fill:#00BCD4,stroke:#00838F,color:#fff
    classDef hello fill:#2196F3,stroke:#1565C0,color:#fff
    classDef cert fill:#FF9800,stroke:#E65100,color:#fff
    classDef key fill:#9C27B0,stroke:#6A1B9A,color:#fff
    classDef done fill:#4CAF50,stroke:#2E7D32,color:#fff,stroke-width:3px
```

---

## Table of Contents

1. [Web Server Fundamentals](#1-web-server-fundamentals)
2. [Apache HTTP Server](#2-apache-http-server)
3. [Nginx](#3-nginx)
4. [SSL/TLS](#4-ssltls)
5. [Reverse Proxy & Load Balancing](#5-reverse-proxy--load-balancing)
6. [Caching](#6-caching)
7. [Database Servers](#7-database-servers)
8. [Mail Servers](#8-mail-servers)
9. [DNS Servers](#9-dns-servers)
10. [Monitoring Services](#10-monitoring-services)
11. [High Availability](#11-high-availability)
12. [Operational Checklists](#12-operational-checklists)
13. [Troubleshooting Quick Reference](#13-troubleshooting-quick-reference)
14. [Appendix: Ports, Files, and Commands](#14-appendix-ports-files-and-commands)

---

# 1. Web Server Fundamentals

## 1.1 What a Web Server Does

A web server is software that listens for HTTP or HTTPS requests and returns responses.

It can:

- Serve static files
- Pass dynamic requests to application runtimes
- Terminate TLS/SSL
- Reverse proxy to upstream services
- Load balance traffic
- Cache content
- Enforce authentication and access controls
- Log requests and errors

Common Linux web service roles include:

- Front-end HTTP server
- Reverse proxy
- Application gateway
- API gateway
- TLS offloader
- Static content server
- Edge cache

## 1.2 Core Concepts

### 1.2.1 Client

A client is usually:

- A web browser
- A mobile app
- A CLI tool such as `curl`
- Another server
- A monitoring system

### 1.2.2 Server

A server is the endpoint receiving requests and generating responses.

### 1.2.3 Resource

A resource may be:

- HTML page
- Image
- CSS file
- JavaScript file
- JSON document
- Video
- API endpoint

### 1.2.4 URI and URL

- URI: Uniform Resource Identifier
- URL: Uniform Resource Locator

Example:

```text
https://example.com:443/docs/index.html?lang=en#intro
```

Components:

- Scheme: `https`
- Host: `example.com`
- Port: `443`
- Path: `/docs/index.html`
- Query string: `lang=en`
- Fragment: `intro`

## 1.3 HTTP Overview

HTTP stands for Hypertext Transfer Protocol.

HTTP is:

- Stateless
- Request/response based
- Text-based in HTTP/1.1
- Binary-framed in HTTP/2
- Usually run over TCP
- Typically secured with TLS for HTTPS

### 1.3.1 HTTP Versions

| Version | Transport | Key Features | Notes |
|---|---|---|---|
| HTTP/1.0 | TCP | One request per connection by default | Legacy |
| HTTP/1.1 | TCP | Persistent connections, chunked transfer | Still common |
| HTTP/2 | TCP + TLS usually | Multiplexing, header compression | Very common |
| HTTP/3 | QUIC/UDP | Faster connection establishment, stream independence | Modern edge deployments |

## 1.4 HTTPS Overview

HTTPS is HTTP over TLS.

Benefits:

- Encryption in transit
- Integrity protection
- Server authentication
- Optional client certificate authentication

HTTPS does **not** automatically mean:

- Secure application logic
- Secure cookies unless configured properly
- Safe from XSS, CSRF, or SQL injection

## 1.5 Request/Response Cycle

### 1.5.1 High-Level Steps

1. User enters a URL.
2. DNS resolves the hostname.
3. Client connects to the server IP and port.
4. TLS handshake occurs for HTTPS.
5. Client sends an HTTP request.
6. Web server receives and processes the request.
7. Web server serves content or proxies upstream.
8. Server sends an HTTP response.
9. Client renders or consumes the response.

### 1.5.2 Mermaid Diagram: HTTP Request Flow

```mermaid
graph TD
    A["User enters URL"] --> B["DNS resolves hostname"]
    B --> C["Client connects to server"]
    C --> D["TLS handshake for HTTPS"]
    D --> E["HTTP request sent"]
    E --> F["Web server receives request"]
    F --> G["Static response or upstream processing"]
    G --> H["HTTP response returned"]
    H --> I["Client renders content"]
```

## 1.6 Anatomy of an HTTP Request

Example:

```http
GET /products?id=10 HTTP/1.1
Host: example.com
User-Agent: curl/8.7.1
Accept: */*
Authorization: Bearer token123
Connection: keep-alive

```

Parts:

- Method
- Request target
- HTTP version
- Headers
- Optional body

## 1.7 Anatomy of an HTTP Response

Example:

```http
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 01 Jan 2025 10:00:00 GMT
Content-Type: application/json
Content-Length: 27
Cache-Control: no-store

{"status":"ok","id":10}
```

Parts:

- Status line
- Headers
- Optional body

## 1.8 HTTP Methods

| Method | Purpose | Safe | Idempotent | Common Use |
|---|---|---:|---:|---|
| GET | Retrieve resource | Yes | Yes | Read page/data |
| HEAD | Retrieve headers only | Yes | Yes | Health checks |
| POST | Create/submit data | No | No | Form submit/API create |
| PUT | Replace resource | No | Yes | Full update |
| PATCH | Partial update | No | No | Partial modification |
| DELETE | Remove resource | No | Yes | Delete object |
| OPTIONS | Query capabilities | Yes | Yes | CORS preflight |
| TRACE | Diagnostic loopback | Yes | Yes | Rarely enabled |
| CONNECT | Tunnel request | No | No | Proxies |

### 1.8.1 Safe vs Idempotent

Safe means the method should not modify the resource.

Idempotent means repeating the same request should result in the same state.

## 1.9 Common HTTP Status Codes

### 1.9.1 Informational

| Code | Meaning |
|---|---|
| 100 | Continue |
| 101 | Switching Protocols |
| 103 | Early Hints |

### 1.9.2 Success

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 202 | Accepted |
| 204 | No Content |
| 206 | Partial Content |

### 1.9.3 Redirection

| Code | Meaning |
|---|---|
| 301 | Moved Permanently |
| 302 | Found |
| 304 | Not Modified |
| 307 | Temporary Redirect |
| 308 | Permanent Redirect |

### 1.9.4 Client Errors

| Code | Meaning |
|---|---|
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 408 | Request Timeout |
| 409 | Conflict |
| 413 | Payload Too Large |
| 414 | URI Too Long |
| 415 | Unsupported Media Type |
| 429 | Too Many Requests |

### 1.9.5 Server Errors

| Code | Meaning |
|---|---|
| 500 | Internal Server Error |
| 501 | Not Implemented |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

## 1.10 Important Request Headers

| Header | Purpose |
|---|---|
| Host | Target hostname |
| User-Agent | Client identity |
| Accept | Accepted content types |
| Accept-Encoding | Compression preferences |
| Authorization | Credentials/token |
| Cookie | Session or state info |
| Content-Type | Body media type |
| Content-Length | Body size |
| Origin | Browser origin |
| Referer | Previous page |
| X-Forwarded-For | Client IP forwarding |
| X-Forwarded-Proto | Original protocol |
| If-None-Match | Cache validation via ETag |
| If-Modified-Since | Cache validation via timestamp |
| Range | Partial content requests |

## 1.11 Important Response Headers

| Header | Purpose |
|---|---|
| Content-Type | MIME type of response |
| Content-Length | Response size |
| Cache-Control | Cache policy |
| Expires | Expiration timestamp |
| ETag | Resource fingerprint |
| Last-Modified | Last change timestamp |
| Set-Cookie | Cookie creation |
| Location | Redirect destination |
| Strict-Transport-Security | HSTS policy |
| Content-Security-Policy | Browser security policy |
| X-Frame-Options | Clickjacking defense |
| X-Content-Type-Options | MIME sniffing defense |
| Referrer-Policy | Referer behavior |
| Access-Control-Allow-Origin | CORS policy |
| Server | Server software identifier |

## 1.12 MIME Types

| Extension | MIME Type |
|---|---|
| .html | text/html |
| .css | text/css |
| .js | application/javascript |
| .json | application/json |
| .png | image/png |
| .jpg | image/jpeg |
| .svg | image/svg+xml |
| .pdf | application/pdf |
| .txt | text/plain |
| .xml | application/xml |

## 1.13 Connection Handling

Important concepts:

- Keep-alive
- Connection pooling
- Timeouts
- Request buffering
- Slow client protection
- Slowloris mitigation

### 1.13.1 Keep-Alive

Keep-alive allows multiple requests over one TCP connection.

Pros:

- Reduced latency
- Fewer TCP handshakes
- Lower CPU overhead

Cons if misconfigured:

- Too many idle connections
- Excessive memory usage

## 1.14 Content Encoding and Compression

Common compression types:

- gzip
- brotli

Typical response headers:

```http
Content-Encoding: gzip
Vary: Accept-Encoding
```

## 1.15 Caching Basics

### 1.15.1 Browser Caching

Used for:

- Static assets
- Images
- JS bundles
- CSS

### 1.15.2 Proxy Caching

Used by:

- Nginx
- Varnish
- CDN edges

### 1.15.3 Cache Validation

Mechanisms:

- ETag / If-None-Match
- Last-Modified / If-Modified-Since

## 1.16 Sessions and Cookies

Cookies commonly store:

- Session identifiers
- Preferences
- Tracking identifiers

Secure cookie attributes:

- `Secure`
- `HttpOnly`
- `SameSite=Lax` or `SameSite=Strict`

Example:

```http
Set-Cookie: sessionid=abc123; Path=/; Secure; HttpOnly; SameSite=Lax
```

## 1.17 Authentication Patterns

Common methods:

- Basic Auth
- Digest Auth
- Bearer tokens
- Session cookies
- Mutual TLS
- OAuth/OpenID Connect at app layer

## 1.18 Common Security Headers

Recommended baseline:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self';
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## 1.19 Common Linux Paths

| Purpose | Path |
|---|---|
| Global configs | `/etc` |
| Logs | `/var/log` |
| Web content | `/var/www` |
| Runtime files | `/run` |
| Certificates | `/etc/ssl` or `/etc/letsencrypt` |
| Service files | `/etc/systemd/system` |

## 1.20 Common Service Management Commands

```bash
sudo systemctl status nginx
sudo systemctl status apache2
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl reload apache2
sudo journalctl -u nginx -xe
sudo journalctl -u apache2 -xe
```

## 1.21 Common Networking Tools

```bash
curl -I https://example.com
ss -tulpn
ip addr show
dig example.com
openssl s_client -connect example.com:443 -servername example.com
traceroute example.com
```

## 1.22 Basic Performance Concepts

Key metrics:

- Throughput
- Latency
- Error rate
- Connection count
- CPU usage
- Memory usage
- Disk I/O
- Network I/O

## 1.23 Basic Hardening Principles

- Patch regularly
- Run only required services
- Restrict ports with firewall rules
- Use least privilege
- Disable directory listing unless needed
- Use TLS everywhere possible
- Remove default pages and sample configs
- Rotate logs
- Monitor errors and access anomalies

## 1.24 Practical Example: Inspecting a Site

```bash
curl -I https://example.com
curl -vk https://example.com/
ss -tulpn | grep ':80\|:443'
openssl s_client -connect example.com:443 -servername example.com
```

## 1.25 Practical Example: Minimal Static Site Layout

```text
/var/www/example.com/
├── html/
│   ├── index.html
│   ├── app.js
│   └── styles.css
└── logs/
    ├── access.log
    └── error.log
```

## 1.26 HTTP Deep Dive

This section extends the earlier HTTP overview with protocol-level details you will use while debugging browsers, APIs, reverse proxies, caches, CDNs, and load balancers.

### 1.26.1 Complete HTTP Request Anatomy

A real HTTP request has more structure than just `GET /`.

Request layers:

1. DNS lookup decides which IP to contact.
2. TCP or QUIC connection is created.
3. TLS handshake happens for HTTPS.
4. HTTP metadata is sent.
5. Optional request body is streamed.
6. The server returns headers, then a body.

#### HTTP/1.1 Request Example

```http
POST /api/v1/orders HTTP/1.1
Host: shop.example.com
User-Agent: curl/8.7.1
Accept: application/json
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json
Content-Length: 74
X-Request-ID: 8f10f8ef-1e45-4ce4-a5f2-d6bf29f74233
Connection: keep-alive

{"customer_id":42,"items":[{"sku":"KB-100","qty":1}],"payment_method":"card"}
```

Field-by-field:

| Part | Example | Why it matters |
|---|---|---|
| Method | `POST` | Tells the server the intended action |
| Request target | `/api/v1/orders` | Identifies the resource or action endpoint |
| Version | `HTTP/1.1` | Determines framing and connection behavior |
| Host | `shop.example.com` | Required in HTTP/1.1 for virtual hosting |
| Authorization | `Bearer ...` | Carries API credentials or tokens |
| Content-Type | `application/json` | Tells the server how to parse the body |
| Content-Length | `74` | Lets the server know how many bytes to read |
| Body | JSON payload | Contains submitted data |

#### HTTP/1.1 Response Example

```http
HTTP/1.1 201 Created
Server: nginx
Date: Tue, 14 Jan 2025 08:21:33 GMT
Content-Type: application/json
Content-Length: 112
Location: /api/v1/orders/9812
Cache-Control: no-store
X-Request-ID: 8f10f8ef-1e45-4ce4-a5f2-d6bf29f74233

{"id":9812,"status":"created","created_at":"2025-01-14T08:21:33Z","links":{"self":"/api/v1/orders/9812"}}
```

Response fields:

| Part | Example | Why it matters |
|---|---|---|
| Status line | `HTTP/1.1 201 Created` | Tells the client whether the request succeeded |
| Content-Type | `application/json` | Drives browser or client parsing behavior |
| Location | `/api/v1/orders/9812` | Often present after a successful create |
| Cache-Control | `no-store` | Stops intermediaries from storing sensitive data |
| Body | JSON document | Carries the actual response payload |

### 1.26.2 `curl -v` Output Explained

`curl -v` is one of the fastest ways to see what is really happening on the wire.

Command:

```bash
curl -v https://api.example.com/v1/orders/42 \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer demo-token'
```

Representative output:

```text
* Host api.example.com:443 was resolved.
* IPv6: (none)
* IPv4: 93.184.216.34
*   Trying 93.184.216.34:443...
* Connected to api.example.com (93.184.216.34) port 443
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519
* ALPN: server accepted h2
> GET /v1/orders/42 HTTP/2
> Host: api.example.com
> User-Agent: curl/8.7.1
> Accept: application/json
> Authorization: Bearer demo-token
>
< HTTP/2 200
< date: Tue, 14 Jan 2025 08:30:01 GMT
< content-type: application/json
< content-length: 87
< cache-control: no-store
< x-request-id: req-01hkvm0t8m4r9n7q3m6b4n2m7d
< server: nginx
<
{"id":42,"status":"shipped","carrier":"dhl","tracking":"JD014600006838281936"}
* Connection #0 to host api.example.com left intact
```

What each block tells you:

| `curl -v` line | Meaning | Operational use |
|---|---|---|
| `Host ... was resolved` | DNS resolution succeeded | Confirms name resolution before blaming HTTP |
| `Trying ...` | TCP connection attempt started | Useful for firewall and routing checks |
| `Connected to ...` | TCP handshake succeeded | Confirms the port is reachable |
| `ALPN: curl offers h2,http/1.1` | Client advertises supported protocols | Helps explain protocol negotiation |
| `TLS handshake` lines | TLS negotiation is happening | Useful when debugging certificate or cipher failures |
| `SSL connection using TLSv1.3 ...` | Shows negotiated TLS version and cipher | Confirms policy and performance expectations |
| `ALPN: server accepted h2` | Server selected HTTP/2 | Explains later framing behavior |
| `>` lines | Raw request headers sent by the client | Lets you verify method, path, and headers |
| `<` lines | Raw response headers received from the server | Lets you inspect status, content type, cache policy |
| `left intact` | Keep-alive connection remains reusable | Confirms persistent connection behavior |

### 1.26.3 HTTP Request Path Through a Reverse-Proxy Stack

```mermaid
graph TD
    A["Browser or API client"] --> B["DNS resolver"]
    B --> C["TCP or QUIC connection"]
    C --> D["TLS handshake"]
    D --> E["Nginx or Apache edge server"]
    E --> F["Application service"]
    F --> G["Database or cache"]
    G --> F
    F --> E
    E --> A
```

Each layer can add latency or fail independently:

- DNS problems fail before HTTP starts.
- TCP problems look like connection refused, reset, or timeout.
- TLS problems look like certificate, protocol, or SNI errors.
- HTTP problems show valid HTTP status codes.
- Application problems usually surface as `5xx`, long response times, or malformed payloads.

### 1.26.4 HTTP Methods in REST API Context

REST does not mean every endpoint must be pure CRUD, but the standard methods still communicate intent.

| Method | Typical REST pattern | Example URI | Typical success code | Notes |
|---|---|---|---|---|
| GET | Read a collection or single resource | `/api/users/42` | `200` | Safe and cacheable when designed correctly |
| HEAD | Read metadata without body | `/healthz` | `200` | Great for health checks and quick probes |
| POST | Create or trigger an action | `/api/orders` | `201` or `202` | Not idempotent by default |
| PUT | Replace an entire resource | `/api/users/42` | `200` or `204` | Idempotent when the same body yields same state |
| PATCH | Modify part of a resource | `/api/users/42` | `200` or `204` | Good for partial field updates |
| DELETE | Remove a resource | `/api/users/42` | `204` | Should be idempotent even if repeated |
| OPTIONS | Discover allowed methods or CORS policy | `/api/users` | `204` | Browser preflight uses this a lot |
| CONNECT | Build a tunnel through a proxy | `proxy.example.com:443` | `200` | Mostly proxy infrastructure |
| TRACE | Reflect request for diagnostics | `/` | `200` | Usually disabled for security reasons |

#### REST-style Examples

Create a resource:

```bash
curl -X POST https://api.example.com/v1/users \
  -H 'Content-Type: application/json' \
  -d '{"email":"alice@example.com","role":"admin"}'
```

Typical response:

```http
HTTP/1.1 201 Created
Location: /v1/users/1001
Content-Type: application/json

{"id":1001,"email":"alice@example.com","role":"admin"}
```

Replace a resource:

```bash
curl -X PUT https://api.example.com/v1/users/1001 \
  -H 'Content-Type: application/json' \
  -d '{"email":"alice@example.com","role":"viewer"}'
```

Partially update a resource:

```bash
curl -X PATCH https://api.example.com/v1/users/1001 \
  -H 'Content-Type: application/merge-patch+json' \
  -d '{"role":"editor"}'
```

Delete a resource:

```bash
curl -X DELETE https://api.example.com/v1/users/1001
```

#### Safe and Idempotent Behavior in Practice

| Method | Safe | Idempotent | Real-world implication |
|---|---:|---:|---|
| GET | Yes | Yes | A load balancer may retry it more safely than a POST |
| HEAD | Yes | Yes | Great for uptime checks |
| POST | No | No | Repeat can create duplicates unless you use idempotency keys |
| PUT | No | Yes | Repeating should not create multiple resources |
| PATCH | No | Usually no | Some patch formats are idempotent, some are not |
| DELETE | No | Yes | Second delete may return `404`, but state is still deleted |

A production API often adds an `Idempotency-Key` header to payment or order creation endpoints so a retried `POST` does not double-charge or double-create.

### 1.26.5 Complete Status Code Reference

The code itself is only half the story. The important operational question is: when do you actually see it?

| Code | Meaning | When you see it |
|---|---|---|
| 100 | Continue | Large uploads when the client waits for `100-continue` before sending the body |
| 101 | Switching Protocols | WebSocket upgrade or protocol negotiation |
| 103 | Early Hints | CDN or origin hints preload assets before final response |
| 200 | OK | Normal successful page load or API read |
| 201 | Created | Successful `POST` created a resource |
| 202 | Accepted | Async job queued, but not finished yet |
| 203 | Non-Authoritative Information | Proxy modified or transformed origin metadata |
| 204 | No Content | Successful delete, health endpoint, or update with no body |
| 206 | Partial Content | Byte-range download for video, resumable file fetches |
| 207 | Multi-Status | WebDAV-style multi-object responses |
| 301 | Moved Permanently | Permanent URL move, HTTPS redirect, canonical host redirect |
| 302 | Found | Legacy temporary redirect used by many frameworks |
| 303 | See Other | Redirect after form submission to a result page |
| 304 | Not Modified | Conditional GET hit with matching `ETag` or `Last-Modified` |
| 307 | Temporary Redirect | Temporary redirect that preserves method and body |
| 308 | Permanent Redirect | Permanent redirect that preserves method and body |
| 400 | Bad Request | Malformed JSON, missing required parameter, invalid syntax |
| 401 | Unauthorized | Missing or invalid credentials |
| 403 | Forbidden | Authenticated but not allowed |
| 404 | Not Found | Wrong path, deleted resource, bad route mapping |
| 405 | Method Not Allowed | `POST` sent to a read-only endpoint |
| 406 | Not Acceptable | Client asked for a representation the server cannot produce |
| 408 | Request Timeout | Client took too long to send request data |
| 409 | Conflict | Version conflict, duplicate state, or business rule collision |
| 410 | Gone | Resource intentionally removed and not coming back |
| 411 | Length Required | Server requires `Content-Length` |
| 412 | Precondition Failed | `If-Match` or `If-Unmodified-Since` precondition failed |
| 413 | Payload Too Large | Upload exceeded proxy or app limit |
| 414 | URI Too Long | Query string or path exceeded server limits |
| 415 | Unsupported Media Type | Wrong `Content-Type`, such as XML sent to JSON-only API |
| 416 | Range Not Satisfiable | Invalid byte range requested |
| 417 | Expectation Failed | `Expect` header requested unsupported behavior |
| 421 | Misdirected Request | HTTP/2 request routed to the wrong origin or SNI context |
| 422 | Unprocessable Content | JSON is syntactically valid but semantically invalid |
| 425 | Too Early | Server refuses replay-risk request during early data |
| 426 | Upgrade Required | Server requires a newer protocol or TLS policy |
| 429 | Too Many Requests | Rate limit triggered at app, WAF, or reverse proxy |
| 431 | Request Header Fields Too Large | Headers or cookies are too large |
| 451 | Unavailable For Legal Reasons | Legal restriction or geo/legal block |
| 500 | Internal Server Error | Unhandled exception or generic server-side failure |
| 501 | Not Implemented | Method or feature not implemented by origin or proxy |
| 502 | Bad Gateway | Reverse proxy received an invalid upstream response |
| 503 | Service Unavailable | Maintenance mode, worker exhaustion, or backend unavailable |
| 504 | Gateway Timeout | Proxy waited too long for upstream |
| 505 | HTTP Version Not Supported | Client used unsupported HTTP version |

#### Status Code Patterns in Web-Server Operations

| Pattern | Typical component | Common root causes |
|---|---|---|
| `301` and `308` spikes | Nginx or Apache | Redirect loops, canonical host changes, forced HTTPS |
| `304` spikes | Browser and CDN | Healthy conditional caching |
| `401` spikes | App or API gateway | Expired tokens, missing Authorization header |
| `403` spikes | WAF, auth layer, filesystem ACLs | Blocked IPs, bad permissions, policy failures |
| `404` spikes | App router or web root | Broken links, missing deploy artifact, bad rewrite |
| `413` spikes | Nginx or Apache | Upload exceeded `client_max_body_size` or request limits |
| `429` spikes | Rate limiter | Abuse, traffic burst, or overly strict limits |
| `502` spikes | Reverse proxy | Upstream down, wrong port, crash loop, invalid headers |
| `503` spikes | Load balancer or app | Draining, overload, maintenance, dependency outage |
| `504` spikes | Reverse proxy | Slow database, long query, slow upstream, timeout mismatch |

### 1.26.6 HTTP Headers Deep Dive

Headers drive routing, authentication, caching, compression, content negotiation, browser security, and CORS behavior.

| Header | Direction | Example | When you see it | Notes |
|---|---|---|---|---|
| `Host` | Request | `Host: api.example.com` | Every HTTP/1.1 request | Required for virtual hosts |
| `:authority` | Request | `:authority: api.example.com` | HTTP/2 and HTTP/3 | Replaces the role of `Host` in pseudo-header form |
| `Accept` | Request | `Accept: application/json` | API clients and browsers | Drives content negotiation |
| `Accept-Encoding` | Request | `Accept-Encoding: gzip, br` | Compression-aware clients | Lets servers choose gzip or Brotli |
| `Accept-Language` | Request | `Accept-Language: en-US` | Browser requests | Used for localization |
| `Authorization` | Request | `Authorization: Bearer ...` | APIs, OAuth-protected endpoints | Never cache blindly with shared caches |
| `Cookie` | Request | `Cookie: sessionid=...` | Browser sessions | Large cookies increase every request size |
| `Content-Type` | Request and response | `Content-Type: application/json` | Any body-carrying request or response | Must match actual encoding |
| `Content-Length` | Request and response | `Content-Length: 874` | Fixed-size payloads | Important for framing in HTTP/1.1 |
| `Transfer-Encoding` | Response | `Transfer-Encoding: chunked` | Streaming HTTP/1.1 response | Allows body without pre-known length |
| `Cache-Control` | Request and response | `Cache-Control: no-store` | APIs, browsers, CDNs | One of the most important caching headers |
| `ETag` | Response | `ETag: "a9c2f-5f6c"` | Cacheable responses | Enables conditional revalidation |
| `If-None-Match` | Request | `If-None-Match: "a9c2f-5f6c"` | Browser or CDN revalidation | Leads to `304` if unchanged |
| `Last-Modified` | Response | `Last-Modified: Tue, 14 Jan 2025 08:00:00 GMT` | Static assets and docs | Simpler validator than `ETag` |
| `If-Modified-Since` | Request | `If-Modified-Since: Tue, 14 Jan 2025 08:00:00 GMT` | Browser refresh | Also leads to `304` |
| `Location` | Response | `Location: /login` | Redirects or `201 Created` | Tells client where to go next |
| `Set-Cookie` | Response | `Set-Cookie: sid=...; Secure; HttpOnly` | Session creation or state changes | Security attributes matter |
| `Origin` | Request | `Origin: https://app.example.com` | Browser CORS or CSRF-sensitive requests | Important for cross-origin policy |
| `Referer` | Request | `Referer: https://app.example.com/profile` | Browser navigations | Often useful in analytics and incident review |
| `User-Agent` | Request | `User-Agent: curl/8.7.1` | Almost every client | Helpful but spoofable |
| `X-Request-ID` | Request and response | `X-Request-ID: req-123` | Traced environments | Great for correlation |
| `X-Forwarded-For` | Request | `X-Forwarded-For: 198.51.100.20` | Reverse proxy chains | Trust only known proxies |
| `X-Forwarded-Proto` | Request | `X-Forwarded-Proto: https` | App behind TLS-terminating proxy | Needed for secure URL generation |
| `Forwarded` | Request | `Forwarded: for=198.51.100.20;proto=https;host=api.example.com` | Standards-oriented proxies | More structured than `X-Forwarded-*` |
| `Access-Control-Allow-Origin` | Response | `Access-Control-Allow-Origin: https://app.example.com` | Cross-origin browser APIs | Controls who can read the response |
| `Access-Control-Allow-Methods` | Response | `Access-Control-Allow-Methods: GET, POST, PATCH` | CORS preflight response | Must match intended client behavior |
| `Access-Control-Allow-Headers` | Response | `Access-Control-Allow-Headers: Authorization, Content-Type` | CORS preflight response | Needed for custom request headers |
| `Vary` | Response | `Vary: Accept-Encoding, Origin` | Cached responses | Prevents cache poisoning or incorrect reuse |
| `Strict-Transport-Security` | Response | `Strict-Transport-Security: max-age=31536000; includeSubDomains` | HTTPS sites | Tells browsers to keep using HTTPS |
| `Content-Security-Policy` | Response | `Content-Security-Policy: default-src 'self'` | Browser-facing apps | Major XSS mitigation tool |
| `WWW-Authenticate` | Response | `WWW-Authenticate: Bearer realm="api"` | `401` responses | Tells clients how to authenticate |

#### `Content-Type` and `Accept`

These two are often confused:

- `Content-Type` describes the format of the body you are sending or receiving.
- `Accept` describes the formats the client is willing to receive.

Example request:

```http
POST /api/import HTTP/1.1
Accept: application/json
Content-Type: text/csv
```

Meaning:

- The client is uploading CSV.
- The client wants JSON back.

#### `Authorization`

Common patterns:

```http
Authorization: Basic YWRtaW46c2VjcmV0
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Authorization: ApiKey 8f9c6f3b8e8c...
```

Operational notes:

- Do not log full tokens.
- Treat responses to authenticated requests as private unless explicitly designed for shared caching.
- Make sure reverse proxies pass the header upstream if required.

#### `Cache-Control`

Common values and when to use them:

| Header value | Use case |
|---|---|
| `Cache-Control: no-store` | Login pages, personal API responses, token responses |
| `Cache-Control: no-cache` | Content may be stored but must be revalidated |
| `Cache-Control: public, max-age=31536000, immutable` | Versioned JS, CSS, font files |
| `Cache-Control: public, s-maxage=60, stale-while-revalidate=30` | Public API or HTML behind CDN |
| `Cache-Control: private, max-age=0, must-revalidate` | User-specific browser-cacheable content |

#### CORS Headers

Browser CORS is a read-control mechanism, not a server-to-server security boundary.

Typical preflight request:

```http
OPTIONS /api/v1/profile HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Authorization, Content-Type
```

Typical preflight response:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 600
Vary: Origin
```

Common CORS mistakes:

- Returning `*` while also using credentials.
- Forgetting `Vary: Origin` on cached responses.
- Allowing methods or headers the API does not actually support.
- Thinking CORS protects non-browser clients.

### 1.26.7 HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Transport | TCP | TCP | QUIC over UDP |
| Framing | Textual | Binary | Binary |
| Multiplexing | No | Yes | Yes |
| Header compression | No | HPACK | QPACK |
| Head-of-line blocking | At request level and TCP level | Fixed at HTTP layer, still affected by TCP loss | Avoided across streams by QUIC |
| TLS expectation | Optional | Usually TLS in browsers | Integrated with QUIC and TLS 1.3 |
| Connection setup | Slowest | Similar to HTTP/1.1 over TLS | Often fastest for repeat connections |
| Best fit | Simple legacy systems | General modern web traffic | High-latency and mobile-heavy edge traffic |

#### Real Performance Differences You Actually Notice

| Scenario | HTTP/1.1 behavior | HTTP/2 behavior | HTTP/3 behavior |
|---|---|---|---|
| Page with many small assets | Many connections or queued requests | One connection multiplexes many assets | Similar multiplexing plus better loss resilience |
| Mobile network with packet loss | Latency spikes and stalled requests | Better than HTTP/1.1, but TCP loss still hurts all streams | Often smoother because stream loss is isolated better |
| First secure connection | TCP + TLS handshake | TCP + TLS handshake | QUIC handshake can reduce setup latency |
| CDN edge workload | Works, but less efficient | Excellent general-purpose choice | Often best at the edge for modern browsers |
| Debugging with old tools | Easiest to inspect manually | More abstract because of binary framing | Even more dependent on protocol-aware tooling |

Practical rules:

- HTTP/1.1 is still everywhere and still matters for upstream proxies and legacy clients.
- HTTP/2 is the default sweet spot for most HTTPS sites today.
- HTTP/3 is most compelling at internet edges, mobile-heavy workloads, and global traffic patterns.

### 1.26.8 Keep-Alive, Pipelining, and Multiplexing

#### Keep-Alive

Keep-alive means reusing one connection for multiple requests.

Benefits:

- Less TCP handshake overhead
- Less TLS handshake overhead
- Lower latency for repeated requests
- Better CPU efficiency under normal load

Trade-offs:

- Idle connections still consume resources
- Too-long keep-alives can pin worker slots or file descriptors
- Load balancer and origin timeouts must be aligned

#### Pipelining

HTTP/1.1 pipelining allowed multiple requests to be sent before earlier responses completed.

Why it is rare now:

- Broken intermediary support
- Response ordering constraints
- Head-of-line blocking problems
- HTTP/2 multiplexing solved the problem more cleanly

#### Multiplexing

HTTP/2 and HTTP/3 can interleave multiple streams over one connection.

That means:

- CSS, JS, API calls, and images can share one connection
- A slow response does not force strict serialized request ordering
- Connection counts are reduced significantly

```mermaid
graph LR
    A["HTTP/1.1 keep-alive"] --> B["One reusable connection"]
    B --> C["Request 1 then response 1"]
    C --> D["Request 2 then response 2"]
    E["HTTP/2 multiplexing"] --> F["One connection with many streams"]
    F --> G["Stream 1 HTML"]
    F --> H["Stream 2 CSS"]
    F --> I["Stream 3 API"]
    J["HTTP/3 multiplexing"] --> K["QUIC connection with independent streams"]
    K --> L["Reduced impact from packet loss"]
```

#### Timeout Alignment Example

A common production issue is mismatched keep-alive behavior:

- Load balancer keep-alive: 60 seconds
- Nginx keep-alive: 15 seconds
- App server idle timeout: 5 seconds

This can create:

- Reused sockets that backend has already closed
- Intermittent `502 Bad Gateway`
- Connection reset by peer errors

Align timeouts intentionally and test under load.

### 1.26.9 Chunked Transfer, Streaming, and Ranges

#### Chunked Transfer Encoding

When a server does not know the final body size in advance in HTTP/1.1, it can stream chunks.

Example:

```http
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Content-Type: text/plain

7
Hello, 
6
world!
0
```

You see this with:

- Streaming responses
- App frameworks generating data progressively
- Reverse proxies relaying upstream chunked bodies

#### Range Requests

Clients can request only part of a file.

Example:

```http
GET /videos/demo.mp4 HTTP/1.1
Range: bytes=0-1048575
```

Typical response:

```http
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1048575/73400320
```

You see this with:

- Video scrubbing
- Download resume
- Large object delivery

### 1.26.10 Practical HTTP Debugging Checklist

Use these commands in order when an HTTP service looks broken:

```bash
curl -I http://example.com
curl -vk https://example.com/
curl --http1.1 -I https://example.com
curl --http2 -I https://example.com
curl -H 'Origin: https://app.example.com' -X OPTIONS -i https://api.example.com/profile
curl -H 'If-None-Match: "etag-value"' -i https://cdn.example.com/app.js
```

Ask these questions:

1. Did DNS resolve correctly?
2. Did TCP connect?
3. Did TLS negotiate the expected protocol and certificate?
4. Which HTTP version was selected?
5. Did the request carry the expected headers?
6. Which component generated the status code?
7. Did cache or CORS headers change behavior?
8. Did a proxy add or strip `Authorization`, `Host`, or forwarding headers?

### 1.26.11 Operational Rules of Thumb

- Use `GET` for reads and keep it cache-aware.
- Use `POST` for creates or side effects unless idempotency is engineered explicitly.
- Do not guess from a browser error page; inspect headers with `curl -v`.
- Treat `502`, `503`, and `504` as different classes of upstream failure.
- Keep authentication, caching, compression, and CORS policies explicit.
- Prefer HTTP/2 or HTTP/3 at the public edge, but still understand HTTP/1.1 deeply because many upstream links still use it.

---

# 2. Apache HTTP Server

## 2.1 Overview

Apache HTTP Server is a mature, modular, highly flexible web server.

Strengths:

- Broad module ecosystem
- Strong compatibility
- Per-directory configuration with `.htaccess`
- Reverse proxy support
- Multiple MPM models

Typical package names:

- Debian/Ubuntu: `apache2`
- RHEL/CentOS/Rocky/Alma: `httpd`

## 2.2 Key Directories

### Debian/Ubuntu

| Purpose | Path |
|---|---|
| Main config | `/etc/apache2/apache2.conf` |
| Ports | `/etc/apache2/ports.conf` |
| Sites available | `/etc/apache2/sites-available/` |
| Sites enabled | `/etc/apache2/sites-enabled/` |
| Mods available | `/etc/apache2/mods-available/` |
| Mods enabled | `/etc/apache2/mods-enabled/` |
| Logs | `/var/log/apache2/` |
| Web root | `/var/www/html/` |

### RHEL Family

| Purpose | Path |
|---|---|
| Main config | `/etc/httpd/conf/httpd.conf` |
| Extra configs | `/etc/httpd/conf.d/` |
| Modules | `/etc/httpd/modules/` |
| Logs | `/var/log/httpd/` |
| Web root | `/var/www/html/` |

## 2.3 Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y apache2
sudo systemctl enable --now apache2
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y httpd
sudo systemctl enable --now httpd
```

### Verify

```bash
systemctl status apache2
curl -I http://127.0.0.1
```

## 2.4 Apache Process Model

Apache supports Multi-Processing Modules (MPMs).

Main MPMs:

- prefork
- worker
- event

### 2.4.1 prefork

Characteristics:

- Process-based
- One thread per process
- Compatible with non-thread-safe modules
- Higher memory usage

Use when:

- Legacy modules require it
- Older PHP setups use `mod_php`

### 2.4.2 worker

Characteristics:

- Multi-process, multi-threaded
- Better memory efficiency than prefork
- Good for many concurrent connections

### 2.4.3 event

Characteristics:

- Similar to worker
- Better keep-alive handling
- Recommended in many modern deployments

### 2.4.4 Check Active MPM

```bash
apachectl -V | grep -i mpm
```

## 2.5 Basic Service Commands

```bash
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo apachectl configtest
sudo apachectl -S
```

## 2.6 Basic Virtual Host Example

Apache virtual hosts allow multiple sites on one server.

### Debian/Ubuntu Example

File:

```text
/etc/apache2/sites-available/example.com.conf
```

Content:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-access.log combined

    <Directory /var/www/example.com/public>
        AllowOverride All
        Require all granted
        Options FollowSymLinks
    </Directory>
</VirtualHost>
```

Enable site:

```bash
sudo a2ensite example.com.conf
sudo apachectl configtest
sudo systemctl reload apache2
```

## 2.7 Name-Based Virtual Hosting

Apache uses the `Host` header to determine which virtual host should serve the request.

Best practices:

- Define explicit `ServerName`
- Add `ServerAlias` only when needed
- Create a catch-all default vhost

## 2.8 HTTPS Virtual Host Example

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example.com/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    Protocols h2 http/1.1

    ErrorLog ${APACHE_LOG_DIR}/example.com-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/example.com-ssl-access.log combined

    <Directory /var/www/example.com/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Redirect HTTP to HTTPS:

```apache
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

## 2.9 Mermaid Diagram: Apache Request Processing

```mermaid
graph TD
    A["Client request"] --> B["Apache listener"]
    B --> C["VirtualHost selection"]
    C --> D["Module processing chain"]
    D --> E["Static file or proxy handler"]
    E --> F["Response filters and logging"]
    F --> G["Client response"]
```

## 2.10 Important Apache Modules

| Module | Purpose |
|---|---|
| `mod_ssl` | TLS/SSL support |
| `mod_rewrite` | URL rewriting |
| `mod_proxy` | Reverse proxy core |
| `mod_proxy_http` | HTTP proxying |
| `mod_proxy_fcgi` | FastCGI proxy |
| `mod_headers` | Header manipulation |
| `mod_deflate` | Compression |
| `mod_brotli` | Brotli compression |
| `mod_expires` | Cache headers |
| `mod_status` | Server status |
| `mod_security2` | WAF integration |
| `mod_remoteip` | Real client IP handling |

### Enable Common Modules on Debian/Ubuntu

```bash
sudo a2enmod ssl rewrite headers expires proxy proxy_http proxy_fcgi status remoteip
sudo apachectl configtest
sudo systemctl reload apache2
```

## 2.11 mod_rewrite Basics

`mod_rewrite` is powerful but easy to misuse.

Typical uses:

- Redirect HTTP to HTTPS
- Remove trailing slashes
- Route all requests to a front controller
- Canonicalize domain names

Example front-controller rewrite:

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.php [L]
</IfModule>
```

Example force non-www to www:

```apache
RewriteEngine On
RewriteCond %{HTTP_HOST} ^example\.com$ [NC]
RewriteRule ^ https://www.example.com%{REQUEST_URI} [L,R=301]
```

## 2.12 .htaccess

`.htaccess` allows per-directory overrides.

Pros:

- Convenience in shared hosting
- No full server config access needed

Cons:

- Performance overhead
- Harder centralized management
- Potentially inconsistent security settings

Recommendation:

- Prefer central config when you control the server
- Use `.htaccess` only when necessary

### Example `.htaccess`

```apache
Options -Indexes

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^old-page$ /new-page [R=301,L]
</IfModule>

<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
</IfModule>
```

## 2.13 Directory Context Controls

Important directives:

- `AllowOverride`
- `Require`
- `Options`
- `DirectoryIndex`

Example:

```apache
<Directory /var/www/example.com/public>
    AllowOverride None
    Require all granted
    Options FollowSymLinks
    DirectoryIndex index.php index.html
</Directory>
```

## 2.14 Authentication in Apache

### Basic Auth Example

```bash
sudo apt install -y apache2-utils
sudo htpasswd -c /etc/apache2/.htpasswd admin
```

```apache
<Location /admin>
    AuthType Basic
    AuthName "Restricted Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Location>
```

## 2.15 Reverse Proxy with Apache

Example proxy to an application server:

```apache
<VirtualHost *:80>
    ServerName app.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Forwarded-Port "80"
</VirtualHost>
```

HTTPS reverse proxy example:

```apache
<VirtualHost *:443>
    ServerName app.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/app.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/app.example.com/privkey.pem

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
</VirtualHost>
```

## 2.16 Apache as FastCGI Front End for PHP-FPM

Example:

```apache
<FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

Benefits of PHP-FPM over `mod_php`:

- Better isolation
- Better resource control
- Improved compatibility with threaded MPMs

## 2.17 Logging

### Access Log Formats

Common formats:

- `common`
- `combined`
- custom JSON-style lines

Example custom format:

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" vhost_combined
CustomLog ${APACHE_LOG_DIR}/example-access.log vhost_combined
```

### Error Logging

```apache
ErrorLog ${APACHE_LOG_DIR}/example-error.log
LogLevel warn
```

Useful log levels:

- emerg
- alert
- crit
- error
- warn
- notice
- info
- debug

## 2.18 Log Rotation

Linux systems commonly use `logrotate`.

Check:

```bash
cat /etc/logrotate.d/apache2
cat /etc/logrotate.d/httpd
```

Important settings:

- `daily`
- `rotate`
- `compress`
- `missingok`
- `notifempty`
- `postrotate`

## 2.19 Performance Tuning Fundamentals

Key directives depend on the MPM in use.

Common tuning areas:

- Max request workers
- Spare servers/threads
- KeepAlive settings
- Timeout values
- Compression
- Static asset caching
- Offloading dynamic work to app servers

### 2.19.1 Global Example

```apache
Timeout 60
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 2
ServerTokens Prod
ServerSignature Off
TraceEnable Off
```

### 2.19.2 prefork Tuning Example

```apache
<IfModule mpm_prefork_module>
    StartServers             4
    MinSpareServers          4
    MaxSpareServers          8
    MaxRequestWorkers      150
    MaxConnectionsPerChild 1000
</IfModule>
```

### 2.19.3 worker Tuning Example

```apache
<IfModule mpm_worker_module>
    StartServers             2
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild 1000
</IfModule>
```

### 2.19.4 event Tuning Example

```apache
<IfModule mpm_event_module>
    StartServers             2
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild 1000
</IfModule>
```

## 2.20 Compression

### Gzip via mod_deflate

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/css application/javascript application/json application/xml
</IfModule>
```

### Brotli via mod_brotli

```apache
<IfModule mod_brotli.c>
    AddOutputFilterByType BROTLI_COMPRESS text/html text/plain text/css application/javascript application/json application/xml
</IfModule>
```

## 2.21 Cache Headers for Static Assets

```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/css "access plus 30 days"
    ExpiresByType application/javascript "access plus 30 days"
    ExpiresByType image/png "access plus 90 days"
    ExpiresByType image/jpeg "access plus 90 days"
</IfModule>
```

## 2.22 Security Hardening

Checklist:

- Disable unnecessary modules
- Hide version details
- Disable TRACE
- Restrict filesystem access
- Use TLS 1.2 and 1.3 only when possible
- Add security headers
- Run ModSecurity if appropriate
- Monitor 4xx and 5xx spikes
- Protect admin paths with auth and IP allowlists

Example:

```apache
ServerTokens Prod
ServerSignature Off
TraceEnable Off
FileETag None
```

## 2.23 mod_status

Enable status endpoint carefully.

```apache
<Location /server-status>
    SetHandler server-status
    Require local
</Location>

ExtendedStatus On
```

Query:

```bash
curl http://127.0.0.1/server-status?auto
```

## 2.24 Real Client IP Behind Proxy

If Apache is behind another proxy or load balancer, configure `mod_remoteip`.

```apache
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.10
```

## 2.25 SELinux Notes for RHEL Systems

Common actions:

```bash
getenforce
sudo setsebool -P httpd_can_network_connect 1
sudo restorecon -Rv /var/www/html
```

## 2.26 Firewall Notes

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

or:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

## 2.27 Common Troubleshooting Commands

```bash
sudo apachectl configtest
sudo apachectl -S
sudo apachectl -M
sudo tail -f /var/log/apache2/error.log
sudo tail -f /var/log/httpd/error_log
curl -I http://127.0.0.1
ss -tulpn | grep ':80\|:443'
```

## 2.28 Example Production-Oriented Apache vHost

```apache
<VirtualHost *:443>
    ServerName www.example.com
    ServerAlias example.com
    DocumentRoot /var/www/example.com/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/www.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/www.example.com/privkey.pem
    SSLProtocol -all +TLSv1.2 +TLSv1.3
    SSLHonorCipherOrder Off
    Protocols h2 http/1.1

    ServerTokens Prod
    ServerSignature Off

    ErrorLog ${APACHE_LOG_DIR}/www-example-error.log
    CustomLog ${APACHE_LOG_DIR}/www-example-access.log combined

    <Directory /var/www/example.com/public>
        AllowOverride None
        Require all granted
        Options FollowSymLinks
    </Directory>

    <IfModule mod_headers.c>
        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
        Header always set X-Content-Type-Options "nosniff"
        Header always set X-Frame-Options "SAMEORIGIN"
        Header always set Referrer-Policy "strict-origin-when-cross-origin"
    </IfModule>

    <IfModule mod_expires.c>
        ExpiresActive On
        ExpiresByType text/css "access plus 30 days"
        ExpiresByType application/javascript "access plus 30 days"
        ExpiresByType image/png "access plus 90 days"
        ExpiresByType image/jpeg "access plus 90 days"
    </IfModule>
</VirtualHost>
```

## 2.29 Apache Best Practices Summary

- Prefer `event` MPM unless a dependency prevents it
- Use PHP-FPM instead of `mod_php` when practical
- Keep vhost configs clean and explicit
- Avoid excessive `.htaccess` usage
- Test config with `apachectl configtest`
- Use separate logs per site when needed
- Monitor worker exhaustion and timeouts

---

# 3. Nginx

## 3.1 Overview

Nginx is a high-performance web server, reverse proxy, and load balancer.

Strengths:

- Event-driven architecture
- Excellent reverse proxy performance
- Efficient memory usage
- Strong TLS support
- Common choice for API and static asset delivery

## 3.2 Key Directories

### Debian/Ubuntu

| Purpose | Path |
|---|---|
| Main config | `/etc/nginx/nginx.conf` |
| Sites available | `/etc/nginx/sites-available/` |
| Sites enabled | `/etc/nginx/sites-enabled/` |
| Snippets | `/etc/nginx/snippets/` |
| Logs | `/var/log/nginx/` |
| Web root | `/var/www/html/` |

### RHEL Family

| Purpose | Path |
|---|---|
| Main config | `/etc/nginx/nginx.conf` |
| Extra configs | `/etc/nginx/conf.d/` |
| Logs | `/var/log/nginx/` |
| Web root | `/usr/share/nginx/html/` |

## 3.3 Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx
```

### Verify

```bash
nginx -v
sudo nginx -t
curl -I http://127.0.0.1
```

## 3.4 Nginx Architecture

Nginx uses:

- One master process
- Multiple worker processes
- Event-driven non-blocking I/O

### Mermaid Diagram: Nginx Architecture

```mermaid
graph TD
    A["Master process"] --> B["Reads config"]
    A --> C["Spawns worker 1"]
    A --> D["Spawns worker 2"]
    A --> E["Spawns worker N"]
    C --> F["Handles connections"]
    D --> F
    E --> F
```

## 3.5 Basic Commands

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo nginx -t
sudo nginx -T
```

## 3.6 Understanding Nginx Contexts

Common contexts:

- `main`
- `events`
- `http`
- `server`
- `location`
- `upstream`

Example skeleton:

```nginx
user www-data;
worker_processes auto;

error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name example.com;
        root /var/www/example.com/public;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

## 3.7 Server Blocks

A server block is similar to an Apache virtual host.

Example:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com/public;
    index index.html index.htm index.php;

    access_log /var/log/nginx/example-access.log;
    error_log /var/log/nginx/example-error.log warn;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Enable on Debian/Ubuntu:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 3.8 Location Matching

Nginx location matching is critical to understand.

Types:

- Prefix match: `location /images/`
- Exact match: `location = /health`
- Regex match: `location ~ \.php$`
- Case-insensitive regex: `location ~* \.(jpg|png)$`
- Preferential prefix: `location ^~ /static/`

Example:

```nginx
location = /health {
    access_log off;
    return 200 'ok';
}

location ^~ /static/ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
}
```

## 3.9 Basic Static Site Example

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static.example.com/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## 3.10 HTTPS Server Block Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    root /var/www/example.com/public;
    index index.html index.php;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options SAMEORIGIN always;
    add_header Referrer-Policy strict-origin-when-cross-origin always;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

## 3.11 Reverse Proxy Basics

Example proxy to an upstream application:

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
    }
}
```

## 3.12 Mermaid Diagram: Reverse Proxy Flow

```mermaid
graph LR
    A["Client"] --> B["Nginx reverse proxy"]
    B --> C["App server 1"]
    B --> D["App server 2"]
    C --> B
    D --> B
    B --> A
```

## 3.13 PHP with Nginx and PHP-FPM

```nginx
server {
    listen 80;
    server_name php.example.com;
    root /var/www/php.example.com/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

## 3.14 Upstreams and Load Balancing

Basic upstream block:

```nginx
upstream app_backend {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
    server 10.0.0.13:8080;
}
```

Use it:

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3.14.1 Load-Balancing Algorithms

#### Round Robin

Default behavior.

```nginx
upstream app_backend {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}
```

#### Least Connections

```nginx
upstream app_backend {
    least_conn;
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}
```

#### IP Hash

```nginx
upstream app_backend {
    ip_hash;
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}
```

#### Weighted Servers

```nginx
upstream app_backend {
    server 10.0.0.11:8080 weight=3;
    server 10.0.0.12:8080 weight=1;
}
```

## 3.15 Passive Health Checks

Open-source Nginx mainly provides passive health checking.

Example tuning:

```nginx
upstream app_backend {
    server 10.0.0.11:8080 max_fails=3 fail_timeout=30s;
    server 10.0.0.12:8080 max_fails=3 fail_timeout=30s;
}
```

## 3.16 Timeouts

Key directives:

- `client_header_timeout`
- `client_body_timeout`
- `keepalive_timeout`
- `send_timeout`
- `proxy_connect_timeout`
- `proxy_send_timeout`
- `proxy_read_timeout`

Example:

```nginx
http {
    client_header_timeout 10s;
    client_body_timeout 10s;
    keepalive_timeout 15s;
    send_timeout 30s;
}
```

## 3.17 Buffering

Useful proxy buffering directives:

- `proxy_buffering`
- `proxy_buffers`
- `proxy_buffer_size`
- `proxy_busy_buffers_size`
- `proxy_max_temp_file_size`

Example:

```nginx
location / {
    proxy_pass http://app_backend;
    proxy_buffering on;
    proxy_buffer_size 8k;
    proxy_buffers 16 8k;
}
```

## 3.18 Nginx Caching

Define cache zone:

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=app_cache:100m max_size=5g inactive=60m use_temp_path=off;
```

Use it:

```nginx
location /api/ {
    proxy_pass http://app_backend;
    proxy_cache app_cache;
    proxy_cache_valid 200 1m;
    proxy_cache_valid 404 30s;
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    add_header X-Cache-Status $upstream_cache_status always;
}
```

## 3.19 Cache Bypass Example

```nginx
map $http_cache_control $skip_cache {
    default 0;
    ~*no-cache 1;
}

location /api/ {
    proxy_pass http://app_backend;
    proxy_cache app_cache;
    proxy_no_cache $skip_cache;
    proxy_cache_bypass $skip_cache;
}
```

## 3.20 Rate Limiting

Define zones:

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
```

Use them:

```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    limit_conn conn_limit 20;
    proxy_pass http://app_backend;
}
```

## 3.21 Static Asset Optimization

```nginx
location ~* \.(css|js|jpg|jpeg|png|gif|svg|ico|woff|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, max-age=2592000, immutable";
    access_log off;
}
```

## 3.22 Gzip Compression

```nginx
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 5;
gzip_types text/plain text/css application/json application/javascript application/xml+rss application/xml image/svg+xml;
```

## 3.23 Brotli Compression

If Brotli module is installed:

```nginx
brotli on;
brotli_comp_level 5;
brotli_types text/plain text/css application/javascript application/json application/xml image/svg+xml;
```

## 3.24 Access Logging

Default combined-style logging is common.

Custom JSON-ish format example:

```nginx
log_format structured escape=json
    '{'
        '"time":"$time_iso8601",'
        '"remote_addr":"$remote_addr",'
        '"request":"$request",'
        '"status":$status,'
        '"body_bytes_sent":$body_bytes_sent,'
        '"referer":"$http_referer",'
        '"user_agent":"$http_user_agent",'
        '"request_time":$request_time,'
        '"upstream_time":"$upstream_response_time"'
    '}';

access_log /var/log/nginx/access.log structured;
```

## 3.25 Error Logging

```nginx
error_log /var/log/nginx/error.log warn;
```

Log levels:

- debug
- info
- notice
- warn
- error
- crit
- alert
- emerg

## 3.26 Worker Tuning

Important directives:

- `worker_processes`
- `worker_connections`
- `worker_rlimit_nofile`
- `multi_accept`
- `use epoll` on Linux where applicable

Example:

```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 4096;
    multi_accept on;
}
```

Approximate max connections:

```text
worker_processes × worker_connections
```

Real capacity is lower because upstreams and system limits also matter.

## 3.27 Open File and Kernel Tuning Notes

Useful system tuning often accompanies Nginx:

- `ulimit -n`
- `net.core.somaxconn`
- `net.ipv4.ip_local_port_range`
- `net.ipv4.tcp_tw_reuse`
- `fs.file-max`

Example sysctl snippet:

```conf
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 8192
fs.file-max = 1000000
```

## 3.28 Real IP Configuration Behind Load Balancer

```nginx
set_real_ip_from 10.0.0.10;
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

## 3.29 Security Hardening

Baseline ideas:

- Hide version with `server_tokens off`
- Restrict methods where possible
- Disable autoindex
- Add security headers
- Limit upload size
- Protect internal locations
- Restrict admin paths

Example:

```nginx
server_tokens off;
client_max_body_size 10m;
autoindex off;
```

### Restrict Methods Example

```nginx
location /api/ {
    limit_except GET POST {
        deny all;
    }
    proxy_pass http://app_backend;
}
```

### Internal Location Example

```nginx
location /internal/ {
    internal;
    alias /srv/private-downloads/;
}
```

## 3.30 HTTP/2

Benefits:

- Multiplexing
- Header compression
- Better use of a single connection

Enable in `listen` directive:

```nginx
listen 443 ssl http2;
```

## 3.31 OCSP Stapling Example

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

## 3.32 WebSocket Proxying

```nginx
location /ws/ {
    proxy_pass http://app_backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

## 3.33 Health Endpoint Example

```nginx
location = /health {
    access_log off;
    default_type text/plain;
    return 200 'ok';
}
```

## 3.34 Production Reverse Proxy Example

```nginx
upstream api_backend {
    least_conn;
    server 10.0.1.11:8080 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:8080 max_fails=3 fail_timeout=30s;
    keepalive 64;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    server_tokens off;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options SAMEORIGIN always;

    location / {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 5s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    location = /health {
        access_log off;
        return 200 'ok';
    }
}
```

## 3.35 Common Troubleshooting Commands

```bash
sudo nginx -t
sudo nginx -T | less
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
curl -I http://127.0.0.1
curl -I https://127.0.0.1 --insecure
ss -tulpn | grep ':80\|:443'
```

## 3.36 Nginx Best Practices Summary

- Keep configs modular and predictable
- Use `try_files` carefully
- Always test with `nginx -t`
- Tune timeouts deliberately
- Forward essential headers to upstreams
- Add observability with structured logs
- Use rate limiting on public APIs
- Cache only cache-safe content

## 3.37 Nginx Production Configuration

This section consolidates the earlier Nginx examples into a production-oriented baseline suitable for a modern web application or API edge.

### 3.37.1 Production Goals

A production-grade Nginx configuration should:

- Use all CPU cores sensibly
- Reuse connections efficiently
- Terminate TLS safely
- Add security headers consistently
- Rate-limit abusive traffic
- Cache safe responses
- Proxy WebSocket traffic correctly
- Expose a health endpoint
- Produce logs that are useful during incidents

### 3.37.2 Full Production `nginx.conf`

```nginx
user www-data;
worker_processes auto;
worker_rlimit_nofile 200000;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;

include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 8192;
    multi_accept on;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main_ext escape=json
        '{'
            '"time":"$time_iso8601",'
            '"remote_addr":"$remote_addr",'
            '"request_id":"$request_id",'
            '"host":"$host",'
            '"method":"$request_method",'
            '"uri":"$request_uri",'
            '"status":$status,'
            '"bytes_sent":$body_bytes_sent,'
            '"request_time":$request_time,'
            '"upstream_time":"$upstream_response_time",'
            '"upstream_addr":"$upstream_addr",'
            '"referer":"$http_referer",'
            '"user_agent":"$http_user_agent"'
        '}';

    access_log /var/log/nginx/access.log main_ext;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    types_hash_max_size 4096;
    server_tokens off;
    keepalive_timeout 20s;
    keepalive_requests 1000;
    client_body_timeout 15s;
    client_header_timeout 15s;
    send_timeout 30s;
    client_max_body_size 20m;
    reset_timedout_connection on;

    include /etc/nginx/conf.d/*.conf;

    gzip on;
    gzip_comp_level 5;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_vary on;
    gzip_types
        text/plain
        text/css
        text/xml
        application/json
        application/javascript
        application/xml
        application/rss+xml
        image/svg+xml;

    proxy_cache_path /var/cache/nginx/app_cache levels=1:2 keys_zone=app_cache:256m max_size=10g inactive=60m use_temp_path=off;

    limit_req_zone $binary_remote_addr zone=api_req_limit:20m rate=20r/s;
    limit_conn_zone $binary_remote_addr zone=perip_conn_limit:20m;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }

    upstream app_backend {
        least_conn;
        keepalive 128;
        server 10.0.1.21:8000 max_fails=3 fail_timeout=10s;
        server 10.0.1.22:8000 max_fails=3 fail_timeout=10s;
        server 10.0.1.23:8000 backup;
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

        root /srv/app/current/public;
        index index.html;

        ssl_certificate /etc/letsencrypt/live/app.example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;
        ssl_prefer_server_ciphers off;
        ssl_stapling on;
        ssl_stapling_verify on;
        resolver 1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 valid=300s ipv6=off;
        resolver_timeout 5s;

        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;
        add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
        add_header Cross-Origin-Opener-Policy "same-origin" always;
        add_header Cross-Origin-Resource-Policy "same-origin" always;
        add_header Content-Security-Policy "default-src 'self'; img-src 'self' data: https:; style-src 'self' 'unsafe-inline'; script-src 'self'; connect-src 'self' https://api.example.com; frame-ancestors 'self'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests" always;

        access_log /var/log/nginx/app-example-access.log main_ext;
        error_log /var/log/nginx/app-example-error.log warn;

        limit_conn perip_conn_limit 40;

        location = /health {
            access_log off;
            default_type text/plain;
            return 200 'ok';
        }

        location /static/ {
            alias /srv/app/current/static/;
            expires 30d;
            add_header Cache-Control "public, max-age=2592000, immutable" always;
            access_log off;
        }

        location /media/ {
            alias /srv/app/shared/media/;
            expires 1h;
            add_header Cache-Control "public, max-age=3600" always;
        }

        location /api/ {
            limit_req zone=api_req_limit burst=40 nodelay;
            proxy_pass http://app_backend;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Request-ID $request_id;
            proxy_connect_timeout 5s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
            proxy_buffering on;
            proxy_buffer_size 16k;
            proxy_buffers 16 16k;
            proxy_busy_buffers_size 64k;
            proxy_cache app_cache;
            proxy_cache_methods GET HEAD;
            proxy_cache_valid 200 1m;
            proxy_cache_valid 404 30s;
            proxy_cache_bypass $http_authorization;
            proxy_no_cache $http_authorization;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            add_header X-Cache-Status $upstream_cache_status always;
        }

        location /ws/ {
            proxy_pass http://app_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_read_timeout 300s;
            proxy_send_timeout 300s;
            proxy_buffering off;
        }

        location / {
            try_files $uri @app;
        }

        location @app {
            proxy_pass http://app_backend;
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

        location ~ /\. {
            deny all;
            access_log off;
            log_not_found off;
        }
    }
}
```

### 3.37.3 Why Each Block Exists

| Directive or block | Why it exists | What goes wrong if omitted |
|---|---|---|
| `worker_processes auto` | Uses available CPU cores | Single worker can bottleneck throughput |
| `worker_connections 8192` | Raises concurrent socket capacity | Busy sites can run out of connections |
| `worker_rlimit_nofile` | Raises usable file descriptors | High-concurrency workloads hit OS limits |
| `keepalive_timeout 20s` | Reuses sockets without keeping them forever | Too low wastes handshakes, too high wastes memory |
| `gzip` | Reduces text payload size | Slower downloads and higher bandwidth usage |
| `limit_req_zone` | Throttles abusive request rates | Login and API endpoints are easier to abuse |
| `proxy_cache_path` | Enables reverse-proxy caching | Every cacheable request hits the app |
| `upstream ... keepalive` | Reuses upstream connections | More TCP churn between proxy and app |
| `ssl_stapling on` | Speeds revocation checks | Slower TLS validation in some clients |
| `add_header Strict-Transport-Security` | Locks browsers to HTTPS | Users can still downgrade on later visits |
| `proxy_set_header X-Forwarded-Proto` | Tells app original scheme | Apps generate wrong callback URLs or insecure links |
| `location /ws/` WebSocket settings | Preserves upgrade behavior | WebSocket handshakes fail or disconnect early |

### 3.37.4 Worker Processes and Connection Tuning

A simple sizing model:

```text
max_client_connections ≈ worker_processes × worker_connections
```

But that is not the full story because:

- Upstream sockets also count
- Idle keep-alives still consume descriptors
- Open files and logs count too
- Kernel backlog, ephemeral ports, and application worker limits still matter

Production tuning checklist:

1. Set `worker_processes auto`.
2. Raise `ulimit -n` for the Nginx service.
3. Set `worker_rlimit_nofile` to match realistic fd needs.
4. Check `sysctl net.core.somaxconn` for backlog pressure.
5. Load-test before raising values further.

### 3.37.5 Gzip Compression Guidelines

Good candidates for gzip:

- HTML
- CSS
- JavaScript
- JSON
- XML
- SVG

Usually avoid gzip for:

- JPEG
- PNG
- MP4
- ZIP
- Already-compressed archives

Compression rules of thumb:

- `gzip_comp_level 4` to `6` is often a good production balance.
- Higher levels burn more CPU for diminishing bandwidth savings.
- Always send `Vary: Accept-Encoding` for compressible assets.

### 3.37.6 Security Headers Baseline

Recommended browser-facing baseline:

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
```

Notes:

- Use HSTS only after HTTPS is known-good everywhere.
- Prefer CSP over `X-XSS-Protection`; modern browsers largely ignore the latter.
- `X-Frame-Options` can be replaced or complemented by `frame-ancestors` in CSP.

### 3.37.7 Rate Limiting Patterns

Burst-friendly API protection:

```nginx
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

location /login/ {
    limit_req zone=login_limit burst=10 nodelay;
    proxy_pass http://app_backend;
}
```

Good uses for rate limiting:

- Login endpoints
- Password reset endpoints
- Public APIs
- Expensive search endpoints
- File upload APIs

Avoid being too aggressive on:

- Health checks
- Internal service mesh traffic
- Admin bulk operations unless separately designed

### 3.37.8 SSL Configuration for an A+-Style Baseline

A practical Nginx TLS baseline today is:

- TLS 1.2 and TLS 1.3 only
- Valid full chain
- HSTS enabled intentionally
- OCSP stapling enabled
- Session tickets disabled unless you manage keys carefully
- Strong certificates and modern OpenSSL

Example reusable snippet:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;
ssl_prefer_server_ciphers off;
ssl_stapling on;
ssl_stapling_verify on;
```

### 3.37.9 Upstream Load Balancing Patterns

| Strategy | Good for | Example |
|---|---|---|
| Round robin | Similar nodes | Default behavior |
| `least_conn` | Uneven request duration | APIs with mixed latency |
| `ip_hash` | Session affinity | Legacy stateful apps |
| Weighted servers | Canary or stronger nodes | `server 10.0.1.21:8000 weight=3;` |
| Backup server | Standby capacity | `server 10.0.1.23:8000 backup;` |

Production advice:

- Prefer stateless apps over sticky routing.
- Use a backup server for emergency overflow, not as a hidden primary.
- Monitor upstream latency, not just status code.

### 3.37.10 Caching Configuration Notes

Only cache when all of the following are true:

- Response is safe to reuse
- Authentication does not personalize the payload
- Vary dimensions are understood
- Expiration or revalidation policy is explicit

Good cache candidates:

- Public API GET responses
- Documentation pages
- Computed but anonymous catalog pages
- Expensive upstream responses with short TTLs

Bad cache candidates unless carefully keyed:

- Authenticated dashboards
- Session-dependent HTML
- Responses that vary by Authorization or Cookie

### 3.37.11 WebSocket Proxy Requirements

WebSocket proxying breaks if you forget any of these:

- `proxy_http_version 1.1`
- `Upgrade` header
- `Connection` header
- Long enough read timeout
- Buffering disabled when appropriate

Diagnostic command:

```bash
curl -i -N \
  -H 'Connection: Upgrade' \
  -H 'Upgrade: websocket' \
  -H 'Host: app.example.com' \
  -H 'Origin: https://app.example.com' \
  https://app.example.com/ws/
```

### 3.37.12 Production Validation Commands

```bash
sudo nginx -t
sudo nginx -T | grep -E 'ssl_|limit_req|proxy_cache|upstream|http2'
systemctl status nginx --no-pager
curl -I http://app.example.com
curl -I https://app.example.com
curl -vk https://app.example.com/health
openssl s_client -connect app.example.com:443 -servername app.example.com </dev/null
```

### 3.38 Apache vs Nginx Decision Guide

Both are excellent. The right answer depends on traffic shape, application model, operations habits, and how much you value flexibility versus efficiency.

### 3.38.1 Detailed Comparison Table

| Decision area | Apache | Nginx | Practical takeaway |
|---|---|---|---|
| Core model | Process or thread based depending on MPM | Event-driven, non-blocking | Nginx is usually more memory-efficient at high concurrency |
| Static file serving | Good | Excellent | Nginx is often preferred at the edge |
| Reverse proxy performance | Strong | Excellent | Nginx is a very common API and TLS front end |
| `.htaccess` support | Yes | No | Apache is useful when per-directory overrides are required |
| Dynamic module flexibility | Very mature module ecosystem | Strong, but different operational style | Apache can be easier for legacy app stacks |
| PHP traditional hosting | Historically dominant | Usually via PHP-FPM | Apache still appears often in shared hosting |
| Config inheritance | Rich directory and vhost model | Explicit block-based routing | Nginx tends to be more predictable once learned |
| Memory use under many idle connections | Higher | Lower | Nginx usually scales better for many concurrent keep-alives |
| Learning curve | Familiar to many admins | Simpler concepts, but matching rules matter | Depends on team background |
| WebSocket and API edge use | Good | Excellent | Nginx is the common default here |
| Legacy application compatibility | Excellent | Sometimes requires adaptation | Apache wins for some older apps |
| Container-era edge proxy role | Less common but viable | Very common | Nginx is a standard ingress-style choice |
| Fine-grained per-directory overrides | Strong | Weak by design | Apache if delegating config to app owners |
| High-concurrency TLS termination | Good | Excellent | Nginx typically wins |
| Shared hosting environments | Very common | Less common | Apache remains popular |

### 3.38.2 Quick Decision Rules

Choose Apache first when:

- You need `.htaccess` compatibility.
- You run older apps built around Apache behavior.
- Your team already has deep Apache operational knowledge.
- You want rich per-directory overrides on multi-tenant systems.

Choose Nginx first when:

- You need a reverse proxy or API gateway.
- You expect high concurrency with many idle keep-alives.
- You want a lightweight TLS terminator in front of app servers.
- You serve a lot of static assets.

Use both together when:

- Nginx handles TLS, static files, caching, and rate limiting.
- Apache sits behind it for app compatibility or existing vhost logic.

### 3.38.3 Mermaid Decision Tree

```mermaid
graph TD
    A["Starting point"] --> B["Need .htaccess compatibility?"]
    B -->|Yes| C["Choose Apache"]
    B -->|No| D["Need high-concurrency reverse proxy or API edge?"]
    D -->|Yes| E["Choose Nginx"]
    D -->|No| F["Running legacy app tightly coupled to Apache modules?"]
    F -->|Yes| G["Choose Apache"]
    F -->|No| H["Need both compatibility and edge efficiency?"]
    H -->|Yes| I["Use Nginx in front of Apache"]
    H -->|No| J["Choose Nginx for simpler modern edge deployments"]
```

### 3.38.4 Common Deployment Patterns

| Pattern | Description | Good fit |
|---|---|---|
| Apache only | Apache serves app directly | Legacy LAMP app, shared hosting |
| Nginx only | Nginx serves static files and proxies app | Modern Django, Node.js, Go, API stack |
| Nginx in front of Apache | Nginx handles TLS and static delivery, Apache handles app | Migration path for older Apache apps |
| Nginx plus app runtime | Nginx with Gunicorn, uWSGI, Node.js, or Java upstream | Most modern Linux web stacks |

### 3.38.5 Final Recommendation Cheat Sheet

- For new greenfield API and app deployments: start with Nginx unless there is a specific Apache dependency.
- For legacy CMS or apps using `.htaccess`: Apache may be the lower-risk option.
- For internet-facing performance and TLS edge concerns: Nginx usually has the operational advantage.
- For migrations: put Nginx in front first, then reduce Apache responsibilities over time.

---

# 4. SSL/TLS

## 4.1 Overview

TLS secures communications between clients and servers.

Goals:

- Confidentiality
- Integrity
- Authentication

TLS replaced SSL, but the term SSL is still widely used in practice.

## 4.2 Key Terms

| Term | Meaning |
|---|---|
| Certificate | Public identity document for a host or entity |
| Private key | Secret key corresponding to certificate |
| CSR | Certificate Signing Request |
| CA | Certificate Authority |
| SAN | Subject Alternative Name |
| Cipher suite | Set of algorithms used in TLS |
| OCSP | Online Certificate Status Protocol |
| Stapling | Server includes OCSP proof in handshake |

## 4.3 Certificate Types

| Type | Description |
|---|---|
| Self-signed | Signed by same entity that created it |
| DV | Domain Validation |
| OV | Organization Validation |
| EV | Extended Validation |
| Wildcard | Covers subdomains like `*.example.com` |
| Multi-domain | Covers multiple SANs |

## 4.4 Private Key Types

Common key algorithms:

- RSA
- ECDSA

### RSA

Pros:

- Broad compatibility

Cons:

- Larger key sizes
- More CPU overhead than ECDSA in many cases

### ECDSA

Pros:

- Smaller keys
- Efficient performance

Cons:

- Legacy compatibility can be weaker in very old clients

## 4.5 TLS Handshake Explained

### Steps

1. ClientHello
2. ServerHello
3. Certificate exchange
4. Key exchange
5. Session keys established
6. Encrypted application data begins

### Mermaid Diagram: TLS Handshake

```mermaid
graph TD
    A["ClientHello"] --> B["ServerHello"]
    B --> C["Certificate sent"]
    C --> D["Key exchange"]
    D --> E["Session keys established"]
    E --> F["Encrypted HTTP traffic"]
```

## 4.6 Generate a Private Key and CSR

RSA example:

```bash
openssl genrsa -out example.com.key 4096
openssl req -new -key example.com.key -out example.com.csr
```

ECDSA example:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out example.com-ecdsa.key
openssl req -new -key example.com-ecdsa.key -out example.com-ecdsa.csr
```

## 4.7 Inspect CSR

```bash
openssl req -in example.com.csr -noout -text
```

## 4.8 Generate Self-Signed Certificate

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout selfsigned.key \
  -out selfsigned.crt
```

Use cases:

- Internal labs
- Development
- Temporary testing

Avoid for public production sites.

## 4.9 Verify Certificate Details

```bash
openssl x509 -in fullchain.pem -noout -text
openssl x509 -in fullchain.pem -noout -dates
openssl x509 -in fullchain.pem -noout -issuer -subject
```

## 4.10 Test Remote TLS

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Useful flags:

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts
openssl s_client -connect example.com:443 -servername example.com -tls1_2
```

## 4.11 Let's Encrypt Overview

Let's Encrypt provides free DV certificates.

Common client:

- Certbot

Benefits:

- Free
- Automated renewals
- Broad ecosystem support

## 4.12 Install Certbot

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-nginx python3-certbot-apache
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y certbot python3-certbot-nginx python3-certbot-apache
```

## 4.13 Request Certificate for Nginx

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

## 4.14 Request Certificate for Apache

```bash
sudo certbot --apache -d example.com -d www.example.com
```

## 4.15 Standalone Mode

Useful when no web server plugin is available.

```bash
sudo certbot certonly --standalone -d example.com
```

## 4.16 Webroot Mode

```bash
sudo certbot certonly --webroot -w /var/www/example.com/public -d example.com -d www.example.com
```

## 4.17 Test Renewal

```bash
sudo certbot renew --dry-run
```

## 4.18 Let's Encrypt File Layout

| Purpose | Path |
|---|---|
| Live certs | `/etc/letsencrypt/live/` |
| Archived certs | `/etc/letsencrypt/archive/` |
| Renewal configs | `/etc/letsencrypt/renewal/` |

## 4.19 Recommended TLS Versions

Production baseline today usually means:

- TLS 1.2
- TLS 1.3

Disable:

- SSLv2
- SSLv3
- TLS 1.0
- TLS 1.1

## 4.20 Cipher Suites

TLS 1.3 uses modern cipher handling.

Focus on:

- Keeping software updated
- Disabling obsolete protocols
- Avoiding manual over-tuning unless justified

## 4.21 OCSP Stapling

OCSP stapling improves certificate revocation checking performance.

### Nginx Example

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

### Apache Example

```apache
SSLUseStapling On
SSLStaplingCache shmcb:/var/run/ocsp(128000)
```

## 4.22 HSTS

HTTP Strict Transport Security tells browsers to use HTTPS only.

Example:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

Use carefully:

- Confirm HTTPS works for all subdomains before `includeSubDomains`
- Understand preload implications before submitting to preload lists

## 4.23 Mutual TLS

Mutual TLS requires clients to present certificates.

Common use cases:

- Internal APIs
- Service-to-service auth
- Administrative interfaces

### Nginx Example

```nginx
ssl_client_certificate /etc/nginx/client-ca.crt;
ssl_verify_client on;
```

### Apache Example

```apache
SSLVerifyClient require
SSLVerifyDepth 2
SSLCACertificateFile /etc/apache2/client-ca.crt
```

## 4.24 SSL Labs Scoring Considerations

The SSL Labs report commonly evaluates:

- Supported protocols
- Key exchange quality
- Cipher strength
- Certificate chain
- HSTS
- Vulnerability exposure

Ways to improve score:

- Use current TLS versions only
- Configure full certificate chain properly
- Enable HSTS carefully
- Remove weak ciphers and protocols
- Keep software patched

## 4.25 Common TLS Mistakes

- Missing intermediate certificates
- Expired certificates
- Wrong certificate for hostname
- Outdated protocols enabled
- Not reloading service after renewal
- Using weak private key sizes
- Forgetting SAN entries

## 4.26 Troubleshooting Certificate Problems

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts
curl -Iv https://example.com/
openssl verify -CAfile chain.pem fullchain.pem
journalctl -u nginx -xe
journalctl -u apache2 -xe
```

## 4.27 Certificate Renewal Hooks

Example post-renew hook:

```bash
sudo certbot renew --deploy-hook "systemctl reload nginx"
```

or:

```bash
sudo certbot renew --deploy-hook "systemctl reload apache2"
```

## 4.28 Example Hardened Nginx TLS Snippet

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_prefer_server_ciphers off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

## 4.29 Example Hardened Apache TLS Snippet

```apache
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLHonorCipherOrder Off
SSLUseStapling On
SSLStaplingCache shmcb:/var/run/ocsp(128000)
Protocols h2 http/1.1
```

## 4.30 TLS Operational Checklist

- Certificate matches hostname
- Private key permissions are restricted
- Renewal is automated
- Service reload after renew is tested
- HSTS is set intentionally
- OCSP stapling works
- TLS 1.2 and 1.3 only
- Monitoring tracks certificate expiration

## 4.31 🔐 SSL/TLS — Complete Guide

This section expands the earlier TLS material into a production-oriented reference focused on security posture, certificate lifecycle, and day-2 operations.

### 4.31.1 History: SSL vs TLS

#### SSL 2.0

- Very old and insecure
- Lacks modern handshake protections
- Should never be enabled

#### SSL 3.0

- Important historically, but deprecated
- Vulnerable to POODLE-style downgrade attacks
- Must be disabled everywhere

#### TLS 1.0 and TLS 1.1

- Successors to SSL
- Improved on SSL, but no longer acceptable for modern public production
- Deprecated by major platforms and compliance baselines

#### TLS 1.2

- Long-standing production standard
- Still widely deployed
- Supports strong cipher suites and good interoperability

#### TLS 1.3

- Current preferred version
- Faster handshake
- Removes many legacy options
- Improves security and simplifies configuration

#### Protocol Timeline Summary

| Protocol | Current status | Why it matters |
|---|---|---|
| SSL 2.0 | Dead | Broken and obsolete |
| SSL 3.0 | Dead | Vulnerable and deprecated |
| TLS 1.0 | Deprecated | Too old for modern baselines |
| TLS 1.1 | Deprecated | Same problem as TLS 1.0 |
| TLS 1.2 | Supported and widely used | Safe baseline when configured correctly |
| TLS 1.3 | Preferred | Best default for current deployments |

### 4.31.2 How TLS Works Step by Step

A useful mental model is: asymmetric cryptography is used to establish trust and shared secrets, then symmetric cryptography carries the actual application data efficiently.

#### Step 1: Client Hello

The client sends:

- Supported TLS versions
- Supported cipher suites
- Supported key exchange groups
- Random number
- Extensions such as SNI and ALPN

Why you care:

- SNI decides which certificate multi-tenant servers return.
- ALPN helps negotiate HTTP/1.1 versus HTTP/2.
- Cipher support affects both security and compatibility.

#### Step 2: Server Hello

The server replies with:

- Selected TLS version
- Selected cipher suite
- Server random number
- Certificate chain
- ALPN selection when applicable

Why you care:

- You can confirm whether TLS 1.3 is actually in use.
- A wrong certificate often means wrong SNI routing or wrong vhost.

#### Step 3: Certificate Verification

The client checks:

- Certificate subject or SAN matches the hostname
- Certificate is within validity dates
- Certificate chain leads to a trusted root CA
- Certificate has not been revoked or obviously invalid

If this fails, you see:

- Browser certificate warnings
- `curl` certificate validation errors
- TLS handshake failures in strict clients

#### Step 4: Key Exchange

Modern deployments usually use ECDHE.

That means:

- The server and client derive a shared secret
- The private key signs or authenticates the exchange rather than directly encrypting all traffic
- Sessions get Perfect Forward Secrecy when configured normally

#### Step 5: Symmetric Encryption Begins

Once handshake completes:

- Both sides use symmetric session keys
- HTTP data becomes encrypted application data
- The rest of the session is much faster than the asymmetric setup phase

```mermaid
graph TD
    A["Client Hello"] --> B["Server Hello"]
    B --> C["Certificate chain sent"]
    C --> D["Client verifies hostname and CA trust"]
    D --> E["ECDHE key exchange"]
    E --> F["Shared session keys derived"]
    F --> G["Encrypted HTTP traffic begins"]
```

### 4.31.3 Chain of Trust Explained

A public certificate is rarely trusted by itself.

Typical chain:

1. Server certificate
2. Intermediate CA certificate
3. Root CA certificate trusted by the operating system or browser

Operational consequences:

- If you install only the leaf certificate and omit the intermediate, many clients fail.
- That is why Nginx commonly uses `fullchain.pem` instead of the leaf certificate alone.

### 4.31.4 Certificate Types

| Type | What it verifies | Typical use |
|---|---|---|
| DV | Domain control only | Standard websites, APIs, automated issuance |
| OV | Domain plus organization details | Businesses wanting stronger identity assertions |
| EV | Extended validation | Historically used for visible identity, less emphasized by browsers now |
| Wildcard | One subdomain pattern such as `*.example.com` | Many subdomains under one zone |
| SAN | Multiple names in one certificate | `example.com`, `www.example.com`, `api.example.com` |

#### DV: Domain Validation

- Fastest and easiest to automate
- Standard for Let's Encrypt
- Good fit for most modern deployments

#### OV: Organization Validation

- Includes business verification by CA
- More paperwork than DV
- Less common for general-purpose internet properties today

#### EV: Extended Validation

- Historically showed a stronger visual browser identity signal
- Modern browsers largely removed the special green-bar treatment
- Usually not worth the extra cost for most teams now

#### Wildcard Certificates

Example:

```text
*.example.com
```

Covers:

- `api.example.com`
- `app.example.com`
- `cdn.example.com`

Does not cover:

- `example.com`
- `a.b.example.com`

#### SAN Certificates

Example SAN set:

```text
example.com
www.example.com
api.example.com
admin.example.com
```

Good when:

- One service or load balancer serves many hostnames
- You want one cert per application boundary instead of many individual certs

### 4.31.5 Let's Encrypt Setup Complete

Install Certbot on Debian or Ubuntu:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx python3-certbot-apache
```

Issue a certificate for Nginx:

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Test auto-renewal:

```bash
sudo certbot renew --dry-run
```

Cron example if needed:

```cron
0 12 * * * /usr/bin/certbot renew --quiet
```

Practical notes:

- On many systems, a systemd timer already handles renewal.
- Always test that renewal also reloads Nginx or Apache safely.
- Make sure port 80 is reachable for HTTP-01 unless you use DNS-01.

#### Typical File Layout

| Path | Purpose |
|---|---|
| `/etc/letsencrypt/live/example.com/fullchain.pem` | Leaf plus intermediate chain |
| `/etc/letsencrypt/live/example.com/privkey.pem` | Private key |
| `/etc/letsencrypt/renewal/example.com.conf` | Renewal metadata |
| `/var/log/letsencrypt/` | Certbot logs |

### 4.31.6 Self-Signed Certificates for Dev and Internal Use

Create a self-signed certificate:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/selfsigned.key \
  -out /etc/ssl/certs/selfsigned.crt
```

Good fits:

- Local development
- Internal labs
- Temporary internal services where your own CA is trusted

Poor fits:

- Public production websites
- Partner-facing APIs unless they explicitly trust your CA

### 4.31.7 OpenSSL Commands Cheat Sheet

#### Generate a Private Key

RSA:

```bash
openssl genrsa -out server.key 4096
```

ECDSA:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out server-ecdsa.key
```

#### Generate a CSR

```bash
openssl req -new -key server.key -out server.csr
```

#### View Certificate Details

```bash
openssl x509 -in server.crt -noout -text
openssl x509 -in server.crt -noout -issuer -subject -dates
```

#### Test an SSL Connection

```bash
openssl s_client -connect example.com:443 -servername example.com
```

#### Show Full Presented Chain

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts
```

#### Verify a Certificate Chain

```bash
openssl verify -CAfile ca-bundle.pem fullchain.pem
```

#### Convert PEM to DER

```bash
openssl x509 -in server.crt -outform der -out server.der
```

#### Convert DER to PEM

```bash
openssl x509 -inform der -in server.der -out server.pem
```

#### Export PKCS#12 Bundle

```bash
openssl pkcs12 -export -out server.p12 -inkey server.key -in server.crt -certfile chain.pem
```

#### Inspect PKCS#12 Bundle

```bash
openssl pkcs12 -info -in server.p12
```

### 4.31.8 TLS 1.2 vs TLS 1.3 Operational Differences

| Area | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Handshake | More round trips | Reduced handshake overhead |
| Cipher flexibility | More legacy complexity | Simpler and safer set of options |
| Performance | Good | Better first-connection latency in many cases |
| Misconfiguration risk | Higher | Lower because weak options were removed |
| Forward secrecy | Common with ECDHE | Standard design expectation |

In real operations, TLS 1.3 often gives:

- Slightly lower handshake latency
- Cleaner configuration
- Fewer legacy cipher decisions
- Better default security posture

### 4.31.9 Certificate Deployment in Nginx and Apache

#### Nginx Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
}
```

#### Apache Example

```apache
<VirtualHost *:443>
    ServerName example.com
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
</VirtualHost>
```

### 4.31.10 SSL/TLS Best Practices

- Disable SSLv3, TLS 1.0, and TLS 1.1.
- Prefer TLS 1.2 and TLS 1.3 only.
- Use strong keys and modern OpenSSL.
- Use `fullchain.pem`, not only the leaf certificate.
- Protect private key file permissions.
- Enable HSTS once HTTPS is stable.
- Enable OCSP stapling.
- Prefer ECDHE-based modern defaults.
- Rotate and monitor certificates before expiry.
- Test from the outside, not only from localhost.

### 4.31.11 Strong Nginx TLS Baseline

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;
ssl_prefer_server_ciphers off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 valid=300s ipv6=off;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

Why it helps:

- Limits protocols to modern versions
- Enables stapling for faster revocation checks
- Disables session tickets unless you manage rotation centrally
- Enables HSTS for browser HTTPS enforcement

### 4.31.12 OCSP Stapling

OCSP stapling means the server includes revocation proof during the handshake.

Benefits:

- Faster client validation
- Less client dependency on direct CA responder availability
- Better overall TLS user experience

Nginx:

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

Apache:

```apache
SSLUseStapling On
SSLStaplingCache shmcb:/var/run/ocsp(128000)
```

### 4.31.13 HSTS

HSTS tells browsers to remember that a host must be contacted with HTTPS only.

Example:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Use carefully because:

- Browsers cache this policy
- Bad HTTPS after rollout can lock users out until fixed
- `includeSubDomains` affects every subdomain

### 4.31.14 Perfect Forward Secrecy

Perfect Forward Secrecy means that compromising the server private key later does not decrypt old captured sessions.

You get this in practice by using modern ephemeral key exchange such as ECDHE.

This matters because:

- Long-term key compromise does less historical damage
- It is part of a modern TLS baseline expectation

### 4.31.15 SSL Labs A+ Configuration Mindset for Nginx

An A+ style score usually depends on:

- Valid certificate chain
- TLS 1.2 and TLS 1.3 only
- Strong protocol and key configuration
- No obvious weak legacy ciphers
- HSTS enabled
- Good server behavior under scan

That should be treated as a helpful target, not the only goal. Real production success also depends on renewal reliability, certificate monitoring, and no surprise regressions during deploys.

### 4.31.16 Common TLS Failure Modes

| Symptom | Likely cause | First check |
|---|---|---|
| Browser says cert does not match host | Wrong SAN or wrong vhost/SNI route | `openssl s_client -servername host ...` |
| Some clients fail, others work | Missing intermediate or legacy client issue | Verify full chain |
| `curl: (60) SSL certificate problem` | CA trust or hostname mismatch | Compare hostname and chain |
| TLS handshake timeout | Firewall, packet inspection, or overloaded service | Test from another network and inspect logs |
| `ERR_SSL_PROTOCOL_ERROR` | Bad protocol mix or broken server block | Check enabled protocols and listener config |
| OCSP stapling warnings | Resolver or stapling fetch issue | Check DNS resolver and OpenSSL logs |
| Renewals succeed but site still serves old cert | Service reload missing | Add deploy hook or timer reload |

### 4.31.17 Certificate Renewal Workflow

A production renewal flow should include:

1. Renewal attempt
2. Validation that new files exist
3. Safe reload of Nginx or Apache
4. External verification that the new certificate is served
5. Alerting if expiry is within your threshold

Deploy hook example:

```bash
sudo certbot renew \
  --deploy-hook "systemctl reload nginx"
```

### 4.31.18 Useful Validation Commands

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null
openssl s_client -connect example.com:443 -servername example.com -showcerts </dev/null
curl -Iv https://example.com
sudo nginx -t
sudo apachectl configtest
```

### 4.31.19 Practical TLS Operations Checklist

- DNS points to the correct edge host.
- The certificate SAN includes every served hostname.
- Nginx or Apache references `fullchain.pem` and the right private key.
- Private key permissions are restricted.
- TLS 1.0 and 1.1 are disabled.
- HSTS is enabled intentionally.
- OCSP stapling is on and working.
- Renewal is automated.
- Renewal triggers reload correctly.
- Expiry is monitored with alerts.

---

# 5. Reverse Proxy & Load Balancing

## 5.1 Overview

A reverse proxy sits in front of one or more backend services.

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

# 6. Caching

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

# 7. Database Servers

## 7.1 Overview

Web stacks usually rely on databases for persistent data.

Focus here:

- MySQL/MariaDB
- PostgreSQL

## 7.2 General Database Design Considerations

- Choose correct data types
- Create appropriate indexes
- Normalize where sensible
- Denormalize deliberately when justified
- Size memory settings according to workload
- Back up regularly and test restores
- Secure network access

## 7.3 MySQL vs MariaDB vs PostgreSQL

| Database | Strengths |
|---|---|
| MySQL | Broad ecosystem, common in LAMP stacks |
| MariaDB | MySQL fork with compatible tooling in many cases |
| PostgreSQL | Powerful SQL features, strong extensibility, robust concurrency |

## 7.4 MySQL/MariaDB Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb
```

or:

```bash
sudo apt install -y mysql-server
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y mariadb-server
sudo systemctl enable --now mariadb
```

## 7.5 Initial Hardening for MySQL/MariaDB

```bash
sudo mysql_secure_installation
```

Typical actions:

- Set root password or auth method
- Remove anonymous users
- Disallow remote root login
- Remove test database
- Reload privilege tables

## 7.6 Basic MySQL Commands

```bash
mysql -u root -p
mysql -e "SHOW DATABASES;"
mysqladmin -u root -p status
```

## 7.7 Important MySQL Configuration Files

| Distro Family | Common Files |
|---|---|
| Debian/Ubuntu | `/etc/mysql/my.cnf`, `/etc/mysql/mariadb.conf.d/` |
| RHEL family | `/etc/my.cnf`, `/etc/my.cnf.d/` |

## 7.8 Basic MySQL Configuration Concepts

Key settings:

- `bind-address`
- `max_connections`
- `innodb_buffer_pool_size`
- `innodb_log_file_size`
- `slow_query_log`
- `long_query_time`

Example:

```ini
[mysqld]
bind-address = 127.0.0.1
max_connections = 300
innodb_buffer_pool_size = 1G
slow_query_log = 1
long_query_time = 1
```

## 7.9 Create Database and User

```sql
CREATE DATABASE appdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'appuser'@'10.%' IDENTIFIED BY 'StrongPasswordHere';
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'10.%';
FLUSH PRIVILEGES;
```

## 7.10 MySQL Backup Basics

### Logical Backup

```bash
mysqldump -u root -p --single-transaction --routines --triggers appdb > appdb.sql
```

### Restore

```bash
mysql -u root -p appdb < appdb.sql
```

## 7.11 MySQL Replication Concepts

Common topology:

- Primary/replica
- Semi-synchronous variants
- Group replication for advanced use cases

### Mermaid Diagram: Master-Slave Replication

```mermaid
graph LR
    A["Primary database"] --> B["Binary log stream"]
    B --> C["Replica 1"]
    B --> D["Replica 2"]
```

## 7.12 Basic MySQL Replication Steps

On primary:

- Enable binary logging
- Set unique server ID
- Create replication user
- Take consistent backup
- Record binlog position

Primary example:

```ini
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW
```

Create replication user:

```sql
CREATE USER 'repl'@'10.%' IDENTIFIED BY 'StrongReplPassword';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'10.%';
FLUSH PRIVILEGES;
```

On replica:

```ini
[mysqld]
server-id = 2
relay_log = relay-bin
read_only = 1
```

Then configure source connection using modern syntax appropriate to version.

## 7.13 MySQL Performance Tuning Basics

Watch:

- Buffer pool hit ratio
- Slow query log
- Temporary tables on disk
- Lock waits
- Connection count
- Replication lag

Best practice:

- Fix queries before over-tuning memory
- Add indexes carefully
- Avoid `SELECT *` in hot paths

## 7.14 PostgreSQL Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable --now postgresql
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql
```

## 7.15 Basic PostgreSQL Commands

```bash
sudo -u postgres psql
sudo -u postgres psql -c "\l"
sudo -u postgres psql -c "SELECT version();"
```

## 7.16 PostgreSQL Key Files

| Purpose | Common Path |
|---|---|
| Main config | `/etc/postgresql/*/main/postgresql.conf` or data dir |
| Client auth | `pg_hba.conf` |
| Data directory | `/var/lib/postgresql/` or `/var/lib/pgsql/` |
| Logs | Distribution dependent |

## 7.17 PostgreSQL Access Control

Two major controls:

- `listen_addresses` in `postgresql.conf`
- host auth rules in `pg_hba.conf`

Example:

```conf
listen_addresses = '127.0.0.1,10.0.0.10'
max_connections = 300
shared_buffers = 1GB
work_mem = 16MB
maintenance_work_mem = 256MB
log_min_duration_statement = 1000
```

Example `pg_hba.conf` entry:

```conf
host    appdb    appuser    10.0.0.0/24    scram-sha-256
```

## 7.18 Create PostgreSQL Database and User

```sql
CREATE ROLE appuser LOGIN PASSWORD 'StrongPasswordHere';
CREATE DATABASE appdb OWNER appuser;
GRANT ALL PRIVILEGES ON DATABASE appdb TO appuser;
```

## 7.19 PostgreSQL Backup Basics

### Logical Backup

```bash
pg_dump -U appuser -F c -d appdb -f appdb.dump
```

### Restore

```bash
pg_restore -U appuser -d appdb appdb.dump
```

## 7.20 PostgreSQL Replication Concepts

Options include:

- Streaming replication
- Logical replication
- Replication slots
- WAL archiving

## 7.21 Basic PostgreSQL Streaming Replication Outline

Primary settings example:

```conf
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
```

Create replication user:

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'StrongReplPassword';
```

## 7.22 PostgreSQL Performance Tuning Basics

Common areas:

- `shared_buffers`
- `work_mem`
- `maintenance_work_mem`
- `effective_cache_size`
- autovacuum tuning
- checkpoint tuning

Always validate changes with workload-specific benchmarks.

## 7.23 Security Best Practices for Databases

- Bind only to required interfaces
- Restrict access with firewalls
- Use strong authentication
- Rotate credentials
- Encrypt backups
- Patch regularly
- Avoid app use of admin accounts
- Audit access where required

## 7.24 Connection Pooling

Why pool connections:

- Databases dislike excessive connection churn
- Pools improve app efficiency
- Pools smooth spikes

Popular poolers:

- MySQL: ProxySQL, app-layer pools
- PostgreSQL: PgBouncer

### PgBouncer Example Concept

- Applications connect to PgBouncer
- PgBouncer manages backend PostgreSQL sessions

## 7.25 Slow Query Analysis

MySQL:

- Enable slow query log
- Use `mysqldumpslow` or Percona tools

PostgreSQL:

- Use `log_min_duration_statement`
- Use `pg_stat_statements`

## 7.26 Database Monitoring Metrics

Track:

- Queries per second
- Connections
- Slow queries
- Replication lag
- Buffer hit rate
- Lock waits
- Disk space
- Backup success
- Checkpoint frequency

## 7.27 Backup Strategy Guidance

Follow 3-2-1 thinking where appropriate:

- 3 copies of data
- 2 media types
- 1 offsite copy

Always test restore procedures.

## 7.28 Example Web App Database Security Pattern

- App user has only needed schema permissions
- Separate migration user if needed
- Replica used for read-heavy reporting
- Backups encrypted and shipped offsite

## 7.29 Database Best Practices Summary

- Tune from measurements, not guesses
- Backup and restore test regularly
- Keep network exposure minimal
- Use replication intentionally, not as a backup substitute
- Monitor lag, disk, and slow query growth

---

# 8. Mail Servers

## 8.1 Overview

Mail infrastructure often includes:

- SMTP server for sending/receiving mail
- IMAP/POP3 server for mailbox access
- DNS records for identity and routing
- Anti-spam and anti-malware controls

Common Linux components:

- Postfix
- Dovecot
- OpenDKIM
- SpamAssassin
- Rspamd

## 8.2 Protocol Summary

| Protocol | Purpose | Default Ports |
|---|---|---|
| SMTP | Mail transfer/submission | 25, 587, 465 |
| IMAP | Mailbox access | 143, 993 |
| POP3 | Mailbox access | 110, 995 |

## 8.3 Postfix Overview

Postfix is a popular Mail Transfer Agent (MTA).

Roles:

- Receive inbound mail
- Relay outbound mail
- Accept authenticated submission

## 8.4 Dovecot Overview

Dovecot commonly provides:

- IMAP server
- POP3 server
- Local mail delivery in some designs
- Authentication services

## 8.5 Basic Installation Example

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y postfix dovecot-imapd dovecot-pop3d
sudo systemctl enable --now postfix dovecot
```

## 8.6 Postfix Main Files

| Purpose | Path |
|---|---|
| Main config | `/etc/postfix/main.cf` |
| Master services | `/etc/postfix/master.cf` |
| Mail queue | `/var/spool/postfix/` |
| Logs | `/var/log/mail.log` or journal |

## 8.7 Dovecot Main Files

| Purpose | Path |
|---|---|
| Main config dir | `/etc/dovecot/` |
| Protocol config | `/etc/dovecot/conf.d/` |
| Logs | journal or mail log |

## 8.8 Minimal Postfix Concepts

Important settings:

- `myhostname`
- `mydomain`
- `myorigin`
- `inet_interfaces`
- `mydestination`
- `relayhost`
- `smtpd_tls_cert_file`
- `smtpd_tls_key_file`

Example:

```conf
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem
smtpd_use_tls = yes
smtpd_tls_security_level = may
smtp_tls_security_level = may
```

## 8.9 Submission Service on Port 587

Enable authenticated submission in `master.cf`.

Conceptual example:

```conf
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
```

## 8.10 Dovecot Maildir Example

```conf
mail_location = maildir:~/Maildir
protocols = imap pop3
```

## 8.11 TLS for Mail

Use TLS for:

- SMTP submission
- IMAP/POP3 secure ports
- Server-to-server mail when supported

## 8.12 SPF

SPF helps receiving servers verify which hosts may send mail for a domain.

Example TXT record:

```dns
example.com. IN TXT "v=spf1 mx ip4:203.0.113.10 include:_spf.example.net -all"
```

## 8.13 DKIM

DKIM signs outgoing mail with a private key.

Receiving servers validate with a public DNS TXT record.

Example selector record concept:

```dns
mail2025._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkq..."
```

## 8.14 DMARC

DMARC builds on SPF and DKIM alignment and provides reporting.

Example:

```dns
_dmarc.example.com. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; adkim=s; aspf=s"
```

Policy options:

- `none`
- `quarantine`
- `reject`

## 8.15 Reverse DNS for Mail

Public mail servers should have proper PTR records.

Common expectation:

- IP resolves to mail hostname
- Hostname resolves back to same IP

## 8.16 Common Mail Security Practices

- Use authenticated submission on 587
- Avoid open relay configurations
- Publish SPF, DKIM, DMARC
- Use TLS certificates for mail hostnames
- Monitor blacklists and reputation
- Protect webmail and admin interfaces

## 8.17 Anti-Spam and Anti-Abuse Notes

Common controls:

- RBL checks
- Rate limiting
- Greylisting
- Spam scoring
- Malware scanning
- DMARC policy enforcement

## 8.18 Mail Troubleshooting Commands

```bash
sudo postfix check
sudo postqueue -p
sudo systemctl status postfix dovecot
sudo tail -f /var/log/mail.log
openssl s_client -connect mail.example.com:587 -starttls smtp
openssl s_client -connect mail.example.com:993
```

## 8.19 Basic Mail Server Setup Summary

1. Set hostname and DNS.
2. Install Postfix and Dovecot.
3. Configure local delivery and authentication.
4. Enable TLS.
5. Publish MX, SPF, DKIM, DMARC, PTR.
6. Test sending and receiving.
7. Monitor logs and reputation.

---

# 9. DNS Servers

## 9.1 Overview

DNS translates names to data such as IPs and service records.

Common server software:

- BIND9
- PowerDNS
- Knot DNS
- Unbound for recursive resolving

## 9.2 DNS Basics

Important concepts:

- Authoritative DNS
- Recursive resolver
- Zone
- Record types
- TTL
- Delegation

## 9.3 Common Record Types

| Record | Purpose |
|---|---|
| A | IPv4 address |
| AAAA | IPv6 address |
| CNAME | Canonical alias |
| MX | Mail exchanger |
| TXT | Free-form text, SPF, verification |
| NS | Name server delegation |
| SRV | Service locator |
| PTR | Reverse mapping |
| SOA | Zone authority data |
| CAA | Certificate authority authorization |

## 9.4 BIND9 Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y bind9 bind9utils bind9-doc dnsutils
sudo systemctl enable --now bind9
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y bind bind-utils
sudo systemctl enable --now named
```

## 9.5 Key BIND Files

| Purpose | Debian/Ubuntu | RHEL Family |
|---|---|---|
| Main config | `/etc/bind/named.conf` | `/etc/named.conf` |
| Zone files | `/etc/bind/` or `/var/cache/bind/` | `/var/named/` |
| Logs | journal/syslog | journal/messages |

## 9.6 Basic named.conf Example

```conf
options {
    directory "/var/cache/bind";
    recursion no;
    allow-query { any; };
    listen-on { any; };
    listen-on-v6 { any; };
    dnssec-validation auto;
};

zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
```

## 9.7 Zone File Example

```dns
$TTL 3600
@   IN  SOA ns1.example.com. admin.example.com. (
        2025010101
        3600
        900
        604800
        86400 )

    IN  NS      ns1.example.com.
    IN  NS      ns2.example.com.

ns1 IN  A       203.0.113.10
ns2 IN  A       203.0.113.11
@   IN  A       203.0.113.20
www IN  CNAME   @
mail IN  A       203.0.113.30
@   IN  MX 10    mail.example.com.
@   IN  TXT      "v=spf1 mx -all"
_sip._tcp IN SRV 10 60 5060 sip.example.com.
```

## 9.8 SOA Fields Explained

- Serial
- Refresh
- Retry
- Expire
- Minimum/negative TTL

Serial best practice:

- Increment on every zone change
- Common format: `YYYYMMDDNN`

## 9.9 Validation Commands

```bash
named-checkconf
named-checkzone example.com /etc/bind/db.example.com
```

## 9.10 Query Testing

```bash
dig @127.0.0.1 example.com A
dig @127.0.0.1 www.example.com CNAME
dig @127.0.0.1 example.com MX
host example.com 127.0.0.1
```

## 9.11 Reverse DNS Zone Example

```dns
$TTL 3600
@   IN  SOA ns1.example.com. admin.example.com. (
        2025010101
        3600
        900
        604800
        86400 )

    IN  NS ns1.example.com.
20  IN  PTR example.com.
30  IN  PTR mail.example.com.
```

## 9.12 Split-Horizon DNS

Split-horizon DNS serves different answers to internal and external clients.

Use cases:

- Internal service addresses
- Private admin panels
- VPN-only resources

Concept:

- Internal view returns private IPs
- External view returns public IPs

### BIND View Example

```conf
acl internal_nets { 10.0.0.0/8; 192.168.0.0/16; };

view "internal" {
    match-clients { internal_nets; };
    recursion yes;
    zone "example.com" {
        type master;
        file "/etc/bind/db.example.com.internal";
    };
};

view "external" {
    match-clients { any; };
    recursion no;
    zone "example.com" {
        type master;
        file "/etc/bind/db.example.com.external";
    };
};
```

## 9.13 DNS Security Basics

- Restrict zone transfers
- Disable recursion on public authoritative servers
- Use TSIG for transfers when appropriate
- Patch BIND regularly
- Limit query access where needed
- Use DNSSEC if required and supported operationally

### Restrict Zone Transfers Example

```conf
allow-transfer { 203.0.113.11; };
```

## 9.14 Secondary DNS

Having at least two authoritative name servers improves resilience.

Typical pattern:

- Primary/master source of truth
- Secondary/slave receives zone transfer

## 9.15 DNS Troubleshooting Tips

Check:

- Zone serial increments
- NS glue records
- TTL behavior
- Propagation expectations
- Firewall allowing TCP and UDP 53
- SOA and MX correctness

## 9.16 Common DNS Commands

```bash
dig example.com ANY
# Some servers restrict ANY queries

dig +short example.com A
dig +trace example.com
rndc reload
systemctl status bind9
systemctl status named
```

## 9.17 DNS Best Practices Summary

- Keep authoritative and recursive roles separate
- Use at least two authoritative servers
- Validate every zone change
- Document TTL choices
- Protect transfers and recursion

---

# 10. Monitoring Services

## 10.1 Overview

Monitoring is essential for reliability, capacity planning, and incident response.

Three pillars:

- Metrics
- Logs
- Traces

Core goals:

- Detect failures quickly
- Understand performance trends
- Troubleshoot incidents efficiently
- Alert on meaningful conditions

## 10.2 Prometheus Overview

Prometheus is a pull-based metrics collection and alerting system.

Strengths:

- Powerful time-series storage
- Flexible PromQL querying
- Common exporters ecosystem
- Alertmanager integration

## 10.3 Grafana Overview

Grafana visualizes metrics, logs, and traces in dashboards.

## 10.4 Nagios and Zabbix Overview

### Nagios

- Plugin-based checks
- Traditional infrastructure monitoring

### Zabbix

- Agent-based and agentless monitoring
- Integrated dashboards and alerting

## 10.5 ELK Stack Overview

ELK means:

- Elasticsearch
- Logstash
- Kibana

Often modernized as Elastic Stack including Beats.

## 10.6 Mermaid Diagram: Monitoring Architecture

```mermaid
graph TD
    A["Servers and applications"] --> B["Exporters and log shippers"]
    B --> C["Prometheus metrics"]
    B --> D["Logstash or Beats"]
    C --> E["Grafana dashboards"]
    C --> F["Alertmanager alerts"]
    D --> G["Elasticsearch"]
    G --> H["Kibana"]
```

## 10.7 Prometheus Installation Concept

### Debian/Ubuntu Example

Use packaged or vendor release depending on standards.

```bash
sudo apt update
sudo apt install -y prometheus
sudo systemctl enable --now prometheus
```

## 10.8 Prometheus Basic Configuration

Example `/etc/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
          - localhost:9090

  - job_name: node
    static_configs:
      - targets:
          - 10.0.0.11:9100
          - 10.0.0.12:9100

  - job_name: nginx
    static_configs:
      - targets:
          - 10.0.0.11:9113
```

## 10.9 Exporters

Common exporters:

- node_exporter
- mysqld_exporter
- postgres_exporter
- blackbox_exporter
- nginx exporter
- apache exporter
- redis exporter

## 10.10 Example Alerts

High CPU alert:

```yaml
groups:
  - name: node-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
```

Low disk space alert:

```yaml
      - alert: LowDiskSpace
        expr: (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay"}) * 100 < 15
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
```

## 10.11 Grafana Deployment Notes

Basic install example:

```bash
sudo apt update
sudo apt install -y grafana
sudo systemctl enable --now grafana-server
```

Default web port:

- `3000`

Common tasks:

- Add data sources
- Import dashboards
- Create alert rules
- Use folders and permissions

## 10.12 Web Server Monitoring Metrics

Track for Apache/Nginx:

- Requests per second
- Active connections
- Response time
- Status code rates
- Upstream errors
- TLS handshake failures
- 4xx/5xx spikes
- Worker saturation

## 10.13 Database Monitoring Metrics

Track:

- Connection usage
- Query latency
- Slow queries
- Replication lag
- Buffer/cache hit rate
- Locks and deadlocks
- Disk and WAL/binlog growth

## 10.14 Mail and DNS Monitoring Metrics

Mail:

- Queue length
- Deferred mail count
- TLS negotiation failures
- Auth failures

DNS:

- Query rate
- SERVFAIL rate
- Response time
- Zone transfer failures
- Recursion abuse signs

## 10.15 Blackbox Monitoring

Blackbox checks probe services externally.

Examples:

- HTTP GET /health
- TLS certificate expiry
- ICMP ping
- TCP connect on ports
- DNS query validation

## 10.16 Log Collection Basics

Common sources:

- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`
- `/var/log/apache2/access.log`
- `/var/log/apache2/error.log`
- database logs
- mail logs
- DNS logs
- system journal

## 10.17 Structured Logging Benefits

Why use it:

- Easier parsing
- Better filtering
- Stronger correlation
- Faster dashboards and alerts

## 10.18 Elastic Stack Pipeline Concept

Flow:

1. Filebeat or agent reads logs.
2. Logstash parses/enriches.
3. Elasticsearch stores/indexes.
4. Kibana visualizes/searches.

## 10.19 Nagios Use Cases

Good for:

- Traditional host/service checks
- Explicit state-based monitoring
- Mature plugin ecosystem

## 10.20 Zabbix Use Cases

Good for:

- Unified infrastructure monitoring
- Agent-driven metrics collection
- Templates for common services

## 10.21 Alerting Best Practices

- Alert on symptoms and causes
- Avoid noisy flapping alerts
- Set reasonable `for:` durations
- Route by severity and ownership
- Include runbook links
- Test alert delivery paths

## 10.22 Sample SLI Ideas

Possible service-level indicators:

- Availability percentage
- p95 latency
- Error rate
- Successful request ratio
- Freshness of data pipeline

## 10.23 Monitoring Security

- Restrict dashboards and admin interfaces
- Use TLS for exporters where needed
- Protect metrics endpoints
- Avoid exposing internals publicly
- Scrub secrets from logs

## 10.24 Basic Monitoring Rollout Plan

1. Node health metrics
2. Web server metrics
3. Database metrics
4. Blackbox checks
5. Centralized logs
6. Alert tuning
7. Capacity dashboards

## 10.25 Monitoring Best Practices Summary

- Start small but consistent
- Prefer useful alerts over many alerts
- Correlate metrics and logs
- Monitor dependencies, not just servers
- Track certificate expiration and backups

---

# 11. High Availability

## 11.1 Overview

High availability reduces downtime through redundancy and failover.

Common goals:

- Remove single points of failure
- Automate failover
- Preserve service continuity
- Support maintenance with minimal downtime

## 11.2 HA Concepts

| Concept | Meaning |
|---|---|
| Active-passive | One node serves, another waits |
| Active-active | Multiple nodes serve simultaneously |
| VIP | Virtual IP address moved during failover |
| Quorum | Minimum votes required for cluster action |
| Fencing | Isolating a faulty node |
| Split brain | Cluster nodes diverge without coordination |

## 11.3 Mermaid Diagram: HA Cluster Architecture

```mermaid
graph TD
    A["Clients"] --> B["Floating VIP"]
    B --> C["Active node"]
    C --> D["Shared or replicated storage"]
    E["Passive node"] --> D
    C --> F["Health and cluster messaging"]
    E --> F
```

## 11.4 Active-Passive vs Active-Active

### Active-Passive

Pros:

- Simpler design
- Easier consistency management

Cons:

- Standby capacity may sit idle

### Active-Active

Pros:

- Better resource utilization
- Potentially higher throughput

Cons:

- More complexity
- Greater need for state coordination

## 11.5 Keepalived Overview

Keepalived commonly provides:

- VRRP-based floating IP failover
- Health tracking scripts
- Lightweight HA for front-end nodes

## 11.6 Keepalived Installation

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install -y keepalived
sudo systemctl enable --now keepalived
```

### RHEL/Rocky/Alma

```bash
sudo dnf install -y keepalived
sudo systemctl enable --now keepalived
```

## 11.7 Keepalived Example

Primary node config:

```conf
vrrp_script chk_nginx {
    script "pidof nginx"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass StrongPass123
    }
    virtual_ipaddress {
        10.0.0.100/24
    }
    track_script {
        chk_nginx
    }
}
```

Secondary node config changes:

- `state BACKUP`
- lower `priority`

## 11.8 Pacemaker and Corosync Overview

Used for more advanced clustering.

### Corosync

- Cluster messaging and membership

### Pacemaker

- Resource management and failover orchestration

Use cases:

- VIP failover
- Service failover
- Storage-aware clustering
- Complex dependency orchestration

## 11.9 DRBD Overview

DRBD replicates block devices between servers.

Use case:

- Active-passive replicated storage for certain workloads

Important caution:

- Storage-level HA is complex
- Application-level replication is often safer for databases when supported

## 11.10 Fencing and STONITH

STONITH means:

- Shoot The Other Node In The Head

It ensures a failed or isolated node is forcefully fenced to prevent split-brain corruption.

For serious clusters, fencing is not optional.

## 11.11 Floating IP Concepts

A floating IP or VIP is moved between nodes during failover.

Common uses:

- Load balancer HA
- Single-service endpoint HA
- Database VIP for active-passive pair

## 11.12 HAProxy with Keepalived Pattern

Common front-end HA pattern:

- Two HAProxy nodes
- Keepalived advertises one VIP
- VIP moves if active node fails

Benefits:

- Simple
- Effective for reverse proxy layer

## 11.13 Database HA Considerations

For databases, HA must consider:

- Replication lag
- Automatic failover logic
- Read/write role changes
- Connection endpoint changes
- Backup continuity
- Split-brain prevention

## 11.14 Web Tier HA Considerations

Stateless web layers are easiest to scale and make highly available.

Best practices:

- Keep session state external
- Store assets centrally or deploy immutably
- Health-check nodes continuously
- Use graceful drain during maintenance

## 11.15 Example HA Web Architecture

- Two Nginx load balancers with Keepalived VIP
- Multiple app servers behind them
- Database primary with replica
- Shared monitoring and centralized logs

## 11.16 Maintenance and Failover Testing

Never assume HA works until tested.

Runbook tests:

- Stop active service
- Simulate node failure
- Validate VIP movement
- Validate app and DB connectivity
- Confirm alerting fired
- Confirm recovery steps

## 11.17 Common HA Failure Modes

- Split brain
- Stale health check logic
- Shared storage bottleneck
- Unreplicated sessions
- Certificate mismatch after failover
- DNS pointing at non-HA endpoint
- Monitoring not tracking standby health

## 11.18 Example Keepalived + Nginx Checklist

- Same Nginx config on both nodes
- Certificates present on both nodes
- Health script validated
- VRRP traffic allowed in firewall/network
- VIP subnet and interface correct
- Failover tested from client perspective

## 11.19 Example Pacemaker Resource Concepts

Possible resources:

- Virtual IP
- Filesystem mount
- Database service
- Web service
- Health monitor scripts

## 11.20 HA Best Practices Summary

- Prefer stateless app tiers
- Use explicit fencing for serious clusters
- Test failover regularly
- Monitor both active and standby nodes
- Document manual recovery steps

---

# 12. Operational Checklists

## 12.1 New Web Server Build Checklist

- Install minimal OS packages
- Update system packages
- Set hostname and timezone
- Configure NTP/chrony
- Create admin users and SSH hardening
- Configure firewall
- Install web server packages
- Prepare directories and ownership
- Deploy site configuration
- Configure TLS
- Enable logging and rotation
- Add monitoring agents/exporters
- Test health endpoint
- Document service ownership

## 12.2 New Apache Site Checklist

- Create document root
- Create vhost config
- Set `ServerName`
- Set logs
- Define directory permissions
- Enable required modules
- Test with `apachectl configtest`
- Enable site
- Reload service
- Validate HTTP and HTTPS

## 12.3 New Nginx Site Checklist

- Create server block
- Set `server_name`
- Set root or proxy upstream
- Add logs
- Add TLS settings if needed
- Test with `nginx -t`
- Enable config
- Reload service
- Validate headers and redirects

## 12.4 TLS Checklist

- Correct SAN names requested
- Renewal automation configured
- Intermediate chain correct
- HTTP to HTTPS redirect working
- HSTS set intentionally
- Monitoring expiry alert configured

## 12.5 Reverse Proxy Checklist

- Upstream health endpoint exists
- `Host` and `X-Forwarded-*` headers forwarded
- Timeouts match workload
- Rate limiting set if public
- Access logs enabled
- Sticky sessions used only if needed
- Draining plan exists

## 12.6 Database Checklist

- Binds only on required addresses
- App user has least privilege
- Backups scheduled
- Restore test completed
- Slow query logging enabled
- Disk free space monitored
- Replication monitored if used

## 12.7 Mail Checklist

- SMTP, IMAP submission ports planned
- TLS cert for mail hostname installed
- SPF published
- DKIM configured
- DMARC policy published
- PTR configured
- Open relay test performed

## 12.8 DNS Checklist

- SOA serial incremented
- Zone validates
- NS records correct
- Glue records correct if needed
- TTL reasonable
- Public authoritative server recursion disabled

## 12.9 Monitoring Checklist

- Node metrics collected
- Web metrics collected
- Database metrics collected
- Blackbox checks active
- Logs centralized
- Alerts routed to team
- Dashboards reviewed

## 12.10 HA Checklist

- No single point of failure in entry path
- VIP or DNS failover strategy defined
- Health checks validated
- Fencing considered for cluster designs
- Failover tested recently
- Runbook documented

---

# 13. Troubleshooting Quick Reference

## 13.1 HTTP Troubleshooting

```bash
curl -I http://example.com
curl -Iv https://example.com
curl -H 'Host: example.com' http://127.0.0.1
```

Questions to ask:

- Is DNS correct?
- Is service listening?
- Is firewall allowing traffic?
- Is TLS certificate valid?
- Is vhost/server block matching the hostname?
- Is upstream healthy?

## 13.2 Apache Troubleshooting

```bash
apachectl configtest
apachectl -S
apachectl -M
journalctl -u apache2 -xe
```

Look for:

- Syntax errors
- Wrong vhost match
- Missing modules
- Permission or SELinux issues
- Backend connection failures

## 13.3 Nginx Troubleshooting

```bash
nginx -t
nginx -T
journalctl -u nginx -xe
```

Look for:

- Syntax errors
- Duplicate directives
- Wrong `server_name`
- Bad `proxy_pass`
- File permission issues
- Upstream timeout or connection refused

## 13.4 TLS Troubleshooting

```bash
openssl s_client -connect example.com:443 -servername example.com
curl -Iv https://example.com/
```

Look for:

- Hostname mismatch
- Expired certificate
- Missing intermediate chain
- Wrong key pair
- OCSP or HSTS issues

## 13.5 Database Troubleshooting

MySQL/MariaDB:

```bash
mysqladmin -u root -p ping
mysql -e 'SHOW PROCESSLIST;'
```

PostgreSQL:

```bash
sudo -u postgres pg_isready
sudo -u postgres psql -c 'SELECT now();'
```

## 13.6 DNS Troubleshooting

```bash
dig example.com A
dig @ns1.example.com example.com MX
dig +trace example.com
```

Look for:

- Wrong zone serial
- Missing records
- Delegation issues
- Firewall blocking TCP/UDP 53

## 13.7 Mail Troubleshooting

```bash
postqueue -p
postfix check
openssl s_client -connect mail.example.com:587 -starttls smtp
```

## 13.8 Service and Port Troubleshooting

```bash
systemctl status nginx
systemctl status apache2
systemctl status mariadb
systemctl status postgresql
ss -tulpn
lsof -i :80
lsof -i :443
```

## 13.9 Logs to Check

- `/var/log/nginx/error.log`
- `/var/log/nginx/access.log`
- `/var/log/apache2/error.log`
- `/var/log/apache2/access.log`
- `/var/log/mysql/error.log`
- PostgreSQL log path per distro
- `/var/log/mail.log`
- journal via `journalctl`

## 13.10 Common Root Causes

- Incorrect DNS
- Service not restarted/reloaded
- Syntax errors in config
- Permissions on files/directories
- Firewall rules
- SELinux/AppArmor restrictions
- Resource exhaustion
- Expired certificates
- Wrong backend addresses

---

# 14. Appendix: Ports, Files, and Commands

## 14.1 Common Ports

| Service | Port |
|---|---|
| HTTP | 80 |
| HTTPS | 443 |
| SMTP | 25 |
| SMTP Submission | 587 |
| SMTPS | 465 |
| IMAP | 143 |
| IMAPS | 993 |
| POP3 | 110 |
| POP3S | 995 |
| DNS | 53 |
| MySQL/MariaDB | 3306 |
| PostgreSQL | 5432 |
| Redis | 6379 |
| Memcached | 11211 |
| Prometheus | 9090 |
| Grafana | 3000 |
| Kibana | 5601 |
| Elasticsearch | 9200 |
| HAProxy stats/custom | varies |

## 14.2 Common Files by Service

### Apache

- `/etc/apache2/apache2.conf`
- `/etc/apache2/sites-available/`
- `/etc/apache2/sites-enabled/`
- `/var/log/apache2/`

### Nginx

- `/etc/nginx/nginx.conf`
- `/etc/nginx/sites-available/`
- `/etc/nginx/sites-enabled/`
- `/var/log/nginx/`

### TLS

- `/etc/ssl/`
- `/etc/letsencrypt/live/`
- `/etc/letsencrypt/archive/`

### MySQL/MariaDB

- `/etc/mysql/my.cnf`
- `/etc/my.cnf`

### PostgreSQL

- `postgresql.conf`
- `pg_hba.conf`

### Postfix

- `/etc/postfix/main.cf`
- `/etc/postfix/master.cf`

### Dovecot

- `/etc/dovecot/`

### BIND9

- `/etc/bind/named.conf`
- `/etc/named.conf`

### Monitoring

- `/etc/prometheus/prometheus.yml`
- `/etc/grafana/`

## 14.3 Useful Commands by Category

### Service Management

```bash
systemctl status nginx
systemctl status apache2
systemctl restart nginx
systemctl reload apache2
journalctl -u nginx -xe
journalctl -u apache2 -xe
```

### Networking

```bash
ss -tulpn
ip addr
ip route
ping 8.8.8.8
traceroute example.com
```

### DNS

```bash
dig example.com A
dig example.com MX
dig +trace example.com
host example.com
```

### HTTP

```bash
curl -I http://example.com
curl -Iv https://example.com
wget --server-response https://example.com
```

### TLS

```bash
openssl s_client -connect example.com:443 -servername example.com
openssl x509 -in cert.pem -noout -text
openssl req -in req.csr -noout -text
```

### Database

```bash
mysql -u root -p
mysqldump -u root -p appdb > appdb.sql
sudo -u postgres psql
pg_dump -U appuser -F c -d appdb -f appdb.dump
```

### Logs

```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/apache2/error.log
tail -f /var/log/mail.log
journalctl -f
```

---

# 15. Scenario-Based Examples

## 15.1 Scenario: Static Website on Nginx

Requirements:

- Public website
- TLS enabled
- Static assets cached
- Minimal maintenance

Example config:

```nginx
server {
    listen 80;
    server_name docs.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name docs.example.com;
    root /var/www/docs.example.com/public;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/docs.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/docs.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(css|js|png|jpg|jpeg|svg|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
}
```

## 15.2 Scenario: PHP App on Apache with PHP-FPM

Requirements:

- Legacy app
- Rewrite rules
- TLS
- Good compatibility

Example:

```apache
<VirtualHost *:443>
    ServerName legacy.example.com
    DocumentRoot /var/www/legacy/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/legacy.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/legacy.example.com/privkey.pem

    <Directory /var/www/legacy/public>
        AllowOverride None
        Require all granted
    </Directory>

    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.php [L]

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
```

## 15.3 Scenario: API Reverse Proxy with Nginx

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

## 15.4 Scenario: HAProxy in Front of Two App Servers

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

## 15.5 Scenario: Basic Mail Host DNS Records

```dns
example.com.            IN MX 10 mail.example.com.
mail.example.com.       IN A 203.0.113.30
example.com.            IN TXT "v=spf1 mx -all"
_dmarc.example.com.     IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
mail2025._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIB..."
```

## 15.6 Scenario: Prometheus Monitoring for Web Tier

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - web1.example.internal:9100
          - web2.example.internal:9100

  - job_name: nginx
    static_configs:
      - targets:
          - web1.example.internal:9113
          - web2.example.internal:9113
```

---

# 16. Advanced Performance and Security Notes

## 16.1 Capacity Planning Basics

Estimate based on:

- Peak requests per second
- Average and p95 response time
- Bytes transferred
- TLS handshake volume
- Concurrent connections
- Cache hit ratio

## 16.2 Load Testing Tools

Common tools:

- `ab`
- `wrk`
- `hey`
- `k6`
- `siege`

Example with `wrk`:

```bash
wrk -t4 -c200 -d30s https://example.com/
```

## 16.3 Reading a Slow Site Problem

Possible causes:

- DNS latency
- TLS handshake overhead
- Backend saturation
- Database contention
- Missing indexes
- No cache
- Large uncompressed assets
- Packet loss or routing issues

## 16.4 Timeouts Strategy

General rule:

- Edge timeouts shorter than app timeouts only when protecting resources intentionally
- Avoid unbounded waits
- Tune to real SLOs and request patterns

## 16.5 Connection Reuse Strategy

- Use keep-alive from clients to proxy
- Use upstream keepalive where supported
- Avoid excessive idle timeout values

## 16.6 Protecting Against Abuse

Techniques:

- Rate limiting
- WAF
- Geo blocking when justified
- Bot management/CDN protections
- CAPTCHA at app layer
- Header size limits
- Connection limits

## 16.7 Content Security Policy Basics

A CSP example:

```http
Content-Security-Policy: default-src 'self'; img-src 'self' data:; script-src 'self'; style-src 'self' 'unsafe-inline'; object-src 'none'; base-uri 'self'; frame-ancestors 'self';
```

## 16.8 Permissions and Ownership

Best practices:

- Web content readable by service account
- Write access only where app requires it
- Private keys readable only by privileged users/processes
- Avoid running services as root beyond initial bind/start needs

## 16.9 Secrets Handling

- Do not store secrets in public repos
- Restrict config file permissions
- Use secret management tools where possible
- Rotate leaked credentials immediately

## 16.10 Backup Encryption and Retention

- Encrypt backups at rest and in transit
- Define retention classes
- Verify restore paths
- Monitor backup job outcomes

## 16.11 Immutable Infrastructure Pattern

Instead of manual server changes:

- Build images
- Deploy from version-controlled config
- Replace instances rather than drift over time

## 16.12 IaC Tools Commonly Used

- Ansible
- Terraform
- Puppet
- Chef
- Salt

## 16.13 Config Management Best Practices

- Keep configs in version control
- Separate secrets
- Template environment-specific values
- Validate before reload
- Roll out with staged environments

## 16.14 Incident Response Basics

During outage:

1. Confirm scope.
2. Check recent changes.
3. Check health dashboards.
4. Review logs and errors.
5. Roll back or fail over if needed.
6. Communicate status.
7. Capture timeline for postmortem.

## 16.15 Change Management Notes

- Use peer review for production configs
- Test syntax before reload
- Prefer graceful reloads where possible
- Monitor after every change

---

# 17. Quick Reference Tables

## 17.1 Apache vs Nginx

| Topic | Apache | Nginx |
|---|---|---|
| Architecture | Process/thread MPM | Event-driven workers |
| `.htaccess` | Yes | No |
| Static file efficiency | Good | Excellent |
| Reverse proxy use | Good | Excellent |
| Legacy PHP compatibility | Very strong | Usually via PHP-FPM |
| Per-directory overrides | Built-in | Centralized config only |

## 17.2 Load Balancer Comparison

| Tool | Strength |
|---|---|
| Nginx | Simple HTTP reverse proxy and edge server |
| HAProxy | Advanced balancing, health checks, performance |
| Traefik | Dynamic modern environments |

## 17.3 Cache Tool Comparison

| Tool | Best Use |
|---|---|
| Nginx cache | Integrated simple reverse proxy caching |
| Varnish | Dedicated HTTP acceleration |
| Redis | Object/session cache |
| Memcached | Simple in-memory object cache |
| CDN | Global edge caching |

## 17.4 Database Comparison

| Database | Good Fit |
|---|---|
| MySQL/MariaDB | Traditional web apps, many CMS stacks |
| PostgreSQL | Feature-rich SQL, complex applications |

## 17.5 Monitoring Stack Comparison

| Tool | Good Fit |
|---|---|
| Prometheus | Metrics and alerting |
| Grafana | Dashboards and visualization |
| Nagios | Traditional service checks |
| Zabbix | Unified infra monitoring |
| ELK | Centralized logs and search |

---

# 18. Final Best Practices Summary

## 18.1 Security

- Patch fast
- Use TLS everywhere
- Limit network exposure
- Harden headers
- Protect admin paths
- Rotate secrets and certificates

## 18.2 Performance

- Cache wisely
- Compress assets
- Tune keep-alives and timeouts
- Monitor worker and connection limits
- Benchmark before and after major changes

## 18.3 Reliability

- Add health checks
- Remove single points of failure
- Test backups and failovers
- Monitor certificates, disk, and error rates

## 18.4 Operability

- Use structured logs
- Keep configs in version control
- Automate validation and deployment
- Write runbooks and checklists

## 18.5 Simplicity

Production quality often improves when architecture is:

- Understandable
- Observable
- Reproducible
- Easy to recover

A simple, well-monitored design usually beats a complex, barely understood one.

---

# 19. Line Expansion Reference Notes

The following concise notes intentionally expand this guide into an extended operational reference while reinforcing the major concepts above.

## 19.1 Web Server Fundamentals Reinforcement

- HTTP is stateless by default.
- HTTPS wraps HTTP in TLS.
- DNS resolution happens before the TCP connection.
- TLS negotiation happens before encrypted HTTP data exchange.
- Status code families quickly reveal broad problem categories.
- `2xx` means success.
- `3xx` means redirect or cached validation behavior.
- `4xx` usually points to client-side request issues or access policy.
- `5xx` usually points to upstream or server-side failure.
- Idempotent methods matter for retries.
- `GET` should not mutate state.
- `POST` is commonly used for create or submit flows.
- Response headers shape browser and proxy behavior.
- Caching can improve performance dramatically.
- Caching mistakes can leak private data.
- Security headers reduce common browser attack surface.
- Reverse proxies centralize edge logic.
- Observability is part of production readiness.
- Logs without timestamps and context are less useful.
- Time sync matters for debugging distributed services.
- Firewall rules should match the intended exposure model.

## 19.2 Apache Reinforcement

- Apache excels at flexibility.
- Apache can serve static content and proxy dynamic apps.
- `mod_rewrite` is powerful but can become hard to maintain.
- Keep rewrite rules minimal and explicit.
- Prefer central configs over scattered `.htaccess` when possible.
- Separate logs per site if operationally helpful.
- Disable unnecessary modules to reduce risk.
- Test every config change with `apachectl configtest`.
- Use `event` MPM for modern workloads where compatible.
- Move PHP handling to PHP-FPM for better isolation.
- Protect status endpoints.
- Watch for worker exhaustion under traffic spikes.
- Keep timeouts reasonable.
- Avoid revealing server version details.
- Validate TLS chain after certificate renewals.

## 19.3 Nginx Reinforcement

- Nginx is efficient for many concurrent connections.
- `server` and `location` matching logic must be understood precisely.
- `try_files` is central to many static and framework setups.
- Forward `Host` and `X-Forwarded-*` headers for proxied apps.
- Use upstream keepalive where appropriate.
- Rate limiting protects public services from bursts and abuse.
- Passive health checks are not a full replacement for active checks.
- `nginx -t` should be part of every change workflow.
- Cache carefully and expose cache status headers when debugging.
- Use `server_tokens off` in production.
- Monitor error logs for upstream failures and client abuse patterns.
- Avoid giant unbounded request body sizes.
- Ensure both HA nodes have the same certificates if using VIP failover.

## 19.4 TLS Reinforcement

- Certificate automation reduces outage risk.
- Renewal without reload is incomplete.
- HSTS should be enabled intentionally, not casually.
- Self-signed certs are not appropriate for public production.
- Missing SAN entries are a common mistake.
- RSA remains broadly compatible.
- ECDSA can provide performance and smaller keys.
- Track certificate expiration in monitoring.
- Validate chain and hostname with `openssl s_client`.
- Disable old protocols.
- Keep cryptographic policy current by patching software.

## 19.5 Reverse Proxy Reinforcement

- Health checks are essential.
- Stateless backends simplify scaling.
- Sticky sessions are sometimes necessary but often avoidable.
- Connection draining supports safer deployments.
- Host-based and path-based routing are common first steps.
- Trust proxy headers only from known upstreams.
- WebSockets require special header handling.
- Load balancing can improve both scale and resilience.

## 19.6 Caching Reinforcement

- Long TTLs suit versioned assets.
- Dynamic personalized pages require caution.
- `Cache-Control` matters more than guesswork.
- Stale serving can improve resilience during backend failure.
- Observe hit ratio, not just configuration intent.
- Invalidating cache precisely is often preferable to global purges.

## 19.7 Database Reinforcement

- Replication is not a backup.
- Backups are not valid until restore is tested.
- Least-privilege database users reduce blast radius.
- Connection pooling can stabilize application behavior.
- Slow query logs guide optimization.
- Indexes help reads but can slow writes.
- Measure before tuning buffers.
- Watch disk growth on WAL, binlogs, and backups.

## 19.8 Mail Reinforcement

- Mail is more than just SMTP service.
- DNS identity records matter heavily for deliverability.
- Open relays are disastrous.
- Reverse DNS matters.
- DKIM and DMARC help domain reputation.
- TLS for submission is expected.

## 19.9 DNS Reinforcement

- Authoritative and recursive roles should usually be separated.
- TTL affects propagation speed and cache load.
- Zone serial increments must be consistent.
- Always validate zone files before reload.
- Split-horizon DNS can simplify internal naming but increases complexity.

## 19.10 Monitoring Reinforcement

- Alert quality matters more than alert count.
- Metrics, logs, and traces solve different questions.
- Monitor from both inside and outside where possible.
- Blackbox checks catch issues that internal metrics may miss.
- Capacity dashboards help prevent incidents.

## 19.11 HA Reinforcement

- HA adds complexity.
- Complexity without testing creates false confidence.
- VIP failover is simple for edge tiers.
- Databases require stricter split-brain prevention.
- Fencing matters in clustered stateful services.
- Runbooks should include manual failback decisions.

---

# 20. Extended Command Cookbook

## 20.1 Apache Commands

```bash
apachectl -V
apachectl -M
apachectl -S
apachectl configtest
systemctl reload apache2
```

## 20.2 Nginx Commands

```bash
nginx -v
nginx -V
nginx -t
nginx -T
systemctl reload nginx
```

## 20.3 TLS Commands

```bash
openssl s_client -connect example.com:443 -servername example.com
openssl x509 -in cert.pem -noout -dates
openssl req -new -newkey rsa:2048 -nodes -keyout key.pem -out req.csr
```

## 20.4 DNS Commands

```bash
dig example.com A
host example.com
nslookup example.com
rndc reload
```

## 20.5 Database Commands

```bash
mysql -e 'SHOW DATABASES;'
mysqladmin ping
sudo -u postgres psql -c 'SELECT version();'
pg_isready
```

## 20.6 Monitoring Commands

```bash
curl -s http://127.0.0.1:9090/-/ready
curl -s http://127.0.0.1:3000/api/health
```

## 20.7 Mail Commands

```bash
postfix status
postqueue -p
doveadm who
```

## 20.8 Network Commands

```bash
ss -tulpn
ip addr show
ip route show
ethtool eth0
```

---

# 21. Production Readiness Questions

## 21.1 Edge Layer

- Is TLS automated?
- Are redirects correct?
- Are headers secure?
- Is rate limiting enabled where needed?
- Are logs centralized?

## 21.2 Application Layer

- Are apps stateless where possible?
- Are health checks lightweight and meaningful?
- Is deployment draining supported?
- Are upstream timeout values correct?

## 21.3 Data Layer

- Are backups tested?
- Is replication monitored?
- Are database users least privilege?
- Is connection pooling implemented?

## 21.4 Operations

- Are dashboards useful during incidents?
- Are alerts actionable?
- Is failover tested?
- Are certificate expirations monitored?
- Are runbooks available?

---

# 22. Practical Validation Checklist by Service

## 22.1 Validate Apache

```bash
apachectl configtest
apachectl -S
curl -H 'Host: example.com' http://127.0.0.1/
```

## 22.2 Validate Nginx

```bash
nginx -t
curl -H 'Host: example.com' http://127.0.0.1/
curl -kI https://127.0.0.1 -H 'Host: example.com'
```

## 22.3 Validate TLS

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null
```

## 22.4 Validate DNS

```bash
dig @ns1.example.com example.com SOA
dig @ns1.example.com example.com MX
```

## 22.5 Validate Mail

```bash
openssl s_client -connect mail.example.com:587 -starttls smtp
```

## 22.6 Validate Database

```bash
mysqladmin ping
sudo -u postgres pg_isready
```

## 22.7 Validate Monitoring

```bash
curl -s http://127.0.0.1:9090/-/ready
curl -s http://127.0.0.1:3000/api/health
```

## 22.8 Validate HA

- Simulate node service stop.
- Confirm VIP or routing failover.
- Confirm client connectivity.
- Confirm alerts fired.
- Confirm no split-brain symptoms.

---

# 23. Supplemental Notes for Daily Operations

## 23.1 Before a Reload

- Validate syntax.
- Confirm backup of current config exists in version control.
- Know rollback command.
- Watch logs during reload.

## 23.2 After a Reload

- Check service status.
- Test health endpoint.
- Verify main hostname.
- Verify TLS handshake.
- Watch error rate dashboard.

## 23.3 Before a Migration

- Inventory dependencies.
- Lower TTL if DNS cutover involved.
- Confirm certificates on new nodes.
- Verify firewall parity.
- Test from staging.

## 23.4 After a Migration

- Verify logs on new service.
- Compare latency and error trends.
- Validate backups.
- Restore normal TTL values if appropriate.

## 23.5 During an Incident

- Stabilize first.
- Gather evidence second.
- Avoid too many simultaneous changes.
- Communicate clearly.
- Preserve logs and timestamps.

---

# 24. Extended Examples and Snippets

## 24.1 Nginx Redirect Canonical Host

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://www.example.com$request_uri;
}
```

## 24.2 Apache Redirect Canonical Host

```apache
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://www.example.com/
</VirtualHost>
```

## 24.3 Nginx Basic Auth Protected Location

```nginx
location /admin/ {
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://admin_pool;
}
```

## 24.4 Apache Deny by Default for Sensitive Path

```apache
<Directory /var/www/example.com/private>
    Require all denied
</Directory>
```

## 24.5 Nginx Custom Error Pages

```nginx
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

location = /404.html {
    internal;
}
```

## 24.6 Apache Custom Error Pages

```apache
ErrorDocument 404 /404.html
ErrorDocument 500 /50x.html
```

## 24.7 Nginx Client Upload Limit

```nginx
client_max_body_size 20m;
```

## 24.8 Apache Request Body Limit

```apache
LimitRequestBody 20971520
```

## 24.9 PostgreSQL HBA Example for App and Replica

```conf
host    appdb          appuser       10.0.0.0/24      scram-sha-256
host    replication    replicator    10.0.1.0/24      scram-sha-256
```

## 24.10 MySQL Bind and Logging Example

```ini
[mysqld]
bind-address = 10.0.0.10
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
```

## 24.11 BIND Zone Transfer Restriction Example

```conf
allow-transfer { 203.0.113.11; 203.0.113.12; };
```

## 24.12 HAProxy HTTPS Redirect Example

```haproxy
frontend fe_http
    bind *:80
    http-request redirect scheme https code 301 unless { ssl_fc }
    default_backend be_apps
```

## 24.13 Keepalived Script Considerations

- Script should be quick.
- Script should reflect actual service readiness.
- Script failure should meaningfully reduce priority.
- Script path and permissions must be correct.

## 24.14 Prometheus Blackbox Target Example

```yaml
scrape_configs:
  - job_name: blackbox-http
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://www.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

## 24.15 Log Rotation Reminder

If logs grow without rotation:

- disk fills
- services may fail
- monitoring becomes noisy
- incident response becomes harder

---

# 25. Final Operational Principles

1. Prefer clarity over cleverness.
2. Prefer automation over manual repetition.
3. Prefer stateless services at the edge and app tiers.
4. Prefer tested backups over assumed backups.
5. Prefer measured tuning over cargo-cult tuning.
6. Prefer simple HA that is validated over complex HA that is theoretical.
7. Prefer structured logs and clear metrics over guesswork.
8. Prefer least privilege everywhere.
9. Prefer certificate automation with monitoring.
10. Prefer version-controlled configuration and peer review.

---

# 26. Extended Line-Oriented Notes

- Web servers convert network requests into application responses.
- Apache and Nginx are both foundational tools in Linux web infrastructure.
- Reverse proxies often become the operational control point.
- TLS is mandatory for modern public services.
- Health checks drive load balancer quality.
- Caching drives performance when done safely.
- Databases drive durability and consistency.
- Mail and DNS remain core infrastructure services.
- Monitoring turns outages into diagnosable events.
- HA reduces downtime but increases design responsibility.
- Every production service needs observability.
- Every critical data service needs tested recovery.
- Every edge service needs clear ownership.
- Every configuration change should be validated before reload.
- Every reload should be followed by real traffic checks.
- Every certificate should be tracked for expiration.
- Every backup should have a restore test schedule.
- Every cluster should have failure drills.
- Every cache should define what must never be cached.
- Every rate limit should reflect real traffic patterns.
- Every DNS zone should be linted before deployment.
- Every mail server should be checked against open relay behavior.
- Every database should be hardened against unnecessary network exposure.
- Every dashboard should answer an operational question.
- Every alert should tell someone what to do next.
- Every HA design should specify failover and failback rules.
- Every VIP design should specify health ownership.
- Every application behind a proxy should understand forwarded headers.
- Every public service should have a blackbox probe.
- Every system should have log retention planning.
- Every service migration should include rollback planning.
- Every storage choice should reflect recovery objectives.
- Every architecture should account for certificate distribution.
- Every deployment platform should support graceful draining when possible.
- Every admin endpoint should be protected.
- Every server should run only what it needs.
- Every team should maintain runbooks for common failures.
- Every incident should produce learning and improvement.
- Linux web operations reward discipline and consistency.
- Simplicity is a force multiplier in reliability engineering.

---

# 27. Additional Reference Tables

## 27.1 Header Quick Reference

| Header | Category | Typical Use |
|---|---|---|
| Host | Request | Name-based routing |
| Authorization | Request | Auth credentials/token |
| Cookie | Request | Session state |
| Content-Type | Both | Payload type |
| Cache-Control | Both | Caching behavior |
| Set-Cookie | Response | Session creation |
| Strict-Transport-Security | Response | Force HTTPS |
| Content-Security-Policy | Response | Browser security policy |

## 27.2 Common Failure Symptoms

| Symptom | Possible Causes |
|---|---|
| 502 Bad Gateway | Upstream down, timeout, wrong port |
| 503 Service Unavailable | Maintenance, no healthy backends, overload |
| TLS handshake failure | Expired cert, bad chain, protocol mismatch |
| Connection refused | Service down, wrong listen IP, firewall |
| High latency | Backend saturation, database slowness, no cache |
| Mail rejected | SPF/DKIM/DMARC, PTR, reputation issues |
| DNS SERVFAIL | Broken zone, DNSSEC issue, server failure |

## 27.3 Web Service File Permissions Reminders

| Item | Guidance |
|---|---|
| Document root | Readable by web user |
| Upload directory | Writable only where required |
| TLS private keys | Restrictive permissions |
| Log files | Writable by service and rotatable |
| Config files | Root-owned where practical |

---

# 28. Closing Notes

This guide covers foundational through advanced Linux web server and related service operations.

Use it as:

- A study guide
- A deployment reference
- A troubleshooting aid
- A production readiness checklist

The right production design depends on:

- Traffic pattern
- Security requirements
- Team expertise
- Recovery objectives
- Operational maturity

When in doubt:

- Start simple
- Measure real behavior
- Automate carefully
- Test failover and recovery
- Document what you learn
