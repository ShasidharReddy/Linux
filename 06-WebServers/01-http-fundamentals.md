# HTTP Fundamentals

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

### 12.1 New Web Server Build Checklist

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

### 13.1 HTTP Troubleshooting

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

### 13.8 Service and Port Troubleshooting

```bash
systemctl status nginx
systemctl status apache2
systemctl status mariadb
systemctl status postgresql
ss -tulpn
lsof -i :80
lsof -i :443
```

### 13.10 Common Root Causes

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

## 14. Appendix: Ports, Files, and Commands

### 14.1 Common Ports

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

### 14.2 Common Files by Service

#### Apache

- `/etc/apache2/apache2.conf`
- `/etc/apache2/sites-available/`
- `/etc/apache2/sites-enabled/`
- `/var/log/apache2/`

#### Nginx

- `/etc/nginx/nginx.conf`
- `/etc/nginx/sites-available/`
- `/etc/nginx/sites-enabled/`
- `/var/log/nginx/`

#### TLS

- `/etc/ssl/`
- `/etc/letsencrypt/live/`
- `/etc/letsencrypt/archive/`

#### MySQL/MariaDB

- `/etc/mysql/my.cnf`
- `/etc/my.cnf`

#### PostgreSQL

- `postgresql.conf`
- `pg_hba.conf`

#### Postfix

- `/etc/postfix/main.cf`
- `/etc/postfix/master.cf`

#### Dovecot

- `/etc/dovecot/`

#### BIND9

- `/etc/bind/named.conf`
- `/etc/named.conf`

#### Monitoring

- `/etc/prometheus/prometheus.yml`
- `/etc/grafana/`

### 14.3 Useful Commands by Category

#### Service Management

```bash
systemctl status nginx
systemctl status apache2
systemctl restart nginx
systemctl reload apache2
journalctl -u nginx -xe
journalctl -u apache2 -xe
```

#### Networking

```bash
ss -tulpn
ip addr
ip route
ping 8.8.8.8
traceroute example.com
```

#### DNS

```bash
dig example.com A
dig example.com MX
dig +trace example.com
host example.com
```

#### HTTP

```bash
curl -I http://example.com
curl -Iv https://example.com
wget --server-response https://example.com
```

#### TLS

```bash
openssl s_client -connect example.com:443 -servername example.com
openssl x509 -in cert.pem -noout -text
openssl req -in req.csr -noout -text
```

#### Database

```bash
mysql -u root -p
mysqldump -u root -p appdb > appdb.sql
sudo -u postgres psql
pg_dump -U appuser -F c -d appdb -f appdb.dump
```

#### Logs

```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/apache2/error.log
tail -f /var/log/mail.log
journalctl -f
```

---

## 16. Advanced Performance and Security Notes

### 16.1 Capacity Planning Basics

Estimate based on:

- Peak requests per second
- Average and p95 response time
- Bytes transferred
- TLS handshake volume
- Concurrent connections
- Cache hit ratio

### 16.2 Load Testing Tools

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

### 16.3 Reading a Slow Site Problem

Possible causes:

- DNS latency
- TLS handshake overhead
- Backend saturation
- Database contention
- Missing indexes
- No cache
- Large uncompressed assets
- Packet loss or routing issues

### 16.4 Timeouts Strategy

General rule:

- Edge timeouts shorter than app timeouts only when protecting resources intentionally
- Avoid unbounded waits
- Tune to real SLOs and request patterns

### 16.5 Connection Reuse Strategy

- Use keep-alive from clients to proxy
- Use upstream keepalive where supported
- Avoid excessive idle timeout values

### 16.6 Protecting Against Abuse

Techniques:

- Rate limiting
- WAF
- Geo blocking when justified
- Bot management/CDN protections
- CAPTCHA at app layer
- Header size limits
- Connection limits

### 16.7 Content Security Policy Basics

A CSP example:

```http
Content-Security-Policy: default-src 'self'; img-src 'self' data:; script-src 'self'; style-src 'self' 'unsafe-inline'; object-src 'none'; base-uri 'self'; frame-ancestors 'self';
```

### 16.8 Permissions and Ownership

Best practices:

- Web content readable by service account
- Write access only where app requires it
- Private keys readable only by privileged users/processes
- Avoid running services as root beyond initial bind/start needs

### 16.9 Secrets Handling

- Do not store secrets in public repos
- Restrict config file permissions
- Use secret management tools where possible
- Rotate leaked credentials immediately

### 16.10 Backup Encryption and Retention

- Encrypt backups at rest and in transit
- Define retention classes
- Verify restore paths
- Monitor backup job outcomes

### 16.11 Immutable Infrastructure Pattern

Instead of manual server changes:

- Build images
- Deploy from version-controlled config
- Replace instances rather than drift over time

### 16.12 IaC Tools Commonly Used

- Ansible
- Terraform
- Puppet
- Chef
- Salt

### 16.13 Config Management Best Practices

- Keep configs in version control
- Separate secrets
- Template environment-specific values
- Validate before reload
- Roll out with staged environments

### 16.14 Incident Response Basics

During outage:

1. Confirm scope.
2. Check recent changes.
3. Check health dashboards.
4. Review logs and errors.
5. Roll back or fail over if needed.
6. Communicate status.
7. Capture timeline for postmortem.

### 16.15 Change Management Notes

- Use peer review for production configs
- Test syntax before reload
- Prefer graceful reloads where possible
- Monitor after every change

---

### 17.1 Apache vs Nginx

| Topic | Apache | Nginx |
|---|---|---|
| Architecture | Process/thread MPM | Event-driven workers |
| `.htaccess` | Yes | No |
| Static file efficiency | Good | Excellent |
| Reverse proxy use | Good | Excellent |
| Legacy PHP compatibility | Very strong | Usually via PHP-FPM |
| Per-directory overrides | Built-in | Centralized config only |

## 18. Final Best Practices Summary

### 18.1 Security

- Patch fast
- Use TLS everywhere
- Limit network exposure
- Harden headers
- Protect admin paths
- Rotate secrets and certificates

### 18.2 Performance

- Cache wisely
- Compress assets
- Tune keep-alives and timeouts
- Monitor worker and connection limits
- Benchmark before and after major changes

### 18.3 Reliability

- Add health checks
- Remove single points of failure
- Test backups and failovers
- Monitor certificates, disk, and error rates

### 18.4 Operability

- Use structured logs
- Keep configs in version control
- Automate validation and deployment
- Write runbooks and checklists

### 18.5 Simplicity

Production quality often improves when architecture is:

- Understandable
- Observable
- Reproducible
- Easy to recover

A simple, well-monitored design usually beats a complex, barely understood one.

---

### 19.1 Web Server Fundamentals Reinforcement

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

### 20.8 Network Commands

```bash
ss -tulpn
ip addr show
ip route show
ethtool eth0
```

---

## 21. Production Readiness Questions

### 21.1 Edge Layer

- Is TLS automated?
- Are redirects correct?
- Are headers secure?
- Is rate limiting enabled where needed?
- Are logs centralized?

### 21.2 Application Layer

- Are apps stateless where possible?
- Are health checks lightweight and meaningful?
- Is deployment draining supported?
- Are upstream timeout values correct?

### 21.3 Data Layer

- Are backups tested?
- Is replication monitored?
- Are database users least privilege?
- Is connection pooling implemented?

### 21.4 Operations

- Are dashboards useful during incidents?
- Are alerts actionable?
- Is failover tested?
- Are certificate expirations monitored?
- Are runbooks available?

---

## 23. Supplemental Notes for Daily Operations

### 23.1 Before a Reload

- Validate syntax.
- Confirm backup of current config exists in version control.
- Know rollback command.
- Watch logs during reload.

### 23.2 After a Reload

- Check service status.
- Test health endpoint.
- Verify main hostname.
- Verify TLS handshake.
- Watch error rate dashboard.

### 23.3 Before a Migration

- Inventory dependencies.
- Lower TTL if DNS cutover involved.
- Confirm certificates on new nodes.
- Verify firewall parity.
- Test from staging.

### 23.4 After a Migration

- Verify logs on new service.
- Compare latency and error trends.
- Validate backups.
- Restore normal TTL values if appropriate.

### 23.5 During an Incident

- Stabilize first.
- Gather evidence second.
- Avoid too many simultaneous changes.
- Communicate clearly.
- Preserve logs and timestamps.

---

## 25. Final Operational Principles

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

## 26. Extended Line-Oriented Notes

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

## 27. Additional Reference Tables

### 27.1 Header Quick Reference

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

### 27.2 Common Failure Symptoms

| Symptom | Possible Causes |
|---|---|
| 502 Bad Gateway | Upstream down, timeout, wrong port |
| 503 Service Unavailable | Maintenance, no healthy backends, overload |
| TLS handshake failure | Expired cert, bad chain, protocol mismatch |
| Connection refused | Service down, wrong listen IP, firewall |
| High latency | Backend saturation, database slowness, no cache |
| Mail rejected | SPF/DKIM/DMARC, PTR, reputation issues |
| DNS SERVFAIL | Broken zone, DNSSEC issue, server failure |

### 27.3 Web Service File Permissions Reminders

| Item | Guidance |
|---|---|
| Document root | Readable by web user |
| Upload directory | Writable only where required |
| TLS private keys | Restrictive permissions |
| Log files | Writable by service and rotatable |
| Config files | Root-owned where practical |

---

## 28. Closing Notes

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
