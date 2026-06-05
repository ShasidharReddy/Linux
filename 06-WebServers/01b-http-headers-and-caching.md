# HTTP Headers, Caching, Cookies, and CORS

← Back to [01-http-fundamentals.md](./01-http-fundamentals.md)

Header behavior, caching decisions, state propagation, and browser cross-origin rules.

---

## 7. HTTP Headers Deep Dive

Headers are where a lot of HTTP behavior lives.

They control:

- content interpretation
- caching
- authentication
- cookies
- CORS
- browser security
- compression

### 7.1 Content-Type

What it does:

- tells the receiver the media type of the body

Examples:

```http
Content-Type: text/html; charset=UTF-8
Content-Type: application/json
Content-Type: image/png
Content-Type: multipart/form-data; boundary=----XYZ
```

Why it matters:

- browsers render based on it
- servers parse request bodies based on it
- wrong values cause parsing or security issues

Request example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John"}'
```

Bad example:

```http
Content-Type: text/plain
```

with a JSON body.

That may cause the API not to parse the body.

### 7.2 Content-Length

What it does:

- tells how many bytes are in the body

Example:

```http
Content-Length: 1234
```

Why it matters:

- important for HTTP/1.1 framing
- progress bars and buffering use it
- mismatches can break parsing

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 27

{"status":"ok","id":10}
```

### 7.3 Content-Encoding

What it does:

- tells how the body was compressed or encoded for transfer

Examples:

```http
Content-Encoding: gzip
Content-Encoding: br
```

Why it matters:

- browser must decode before using body
- improves transfer size
- must match actual encoding

Representative exchange:

```http
Accept-Encoding: gzip, br
```

```http
Content-Encoding: br
Vary: Accept-Encoding
```

### 7.4 Cache-Control

What it does:

- tells browsers and caches how to store or revalidate content

Common examples:

```http
Cache-Control: no-store
Cache-Control: no-cache
Cache-Control: public, max-age=3600
Cache-Control: public, max-age=31536000, immutable
```

How to think about them:

- `no-store` means do not store at all
- `no-cache` means you may store it, but must revalidate before reuse
- `max-age=3600` means fresh for one hour
- `immutable` means versioned assets should never change during that freshness period

Good use cases:

- HTML: often `no-cache` or short TTL
- versioned JS/CSS: long TTL and `immutable`
- auth responses: usually `no-store`

### 7.5 ETag

What it does:

- provides a validator for a representation

Example:

```http
ETag: "appjs-v17"
```

Why it matters:

- browser can ask if resource changed
- enables `304 Not Modified`

Response example:

```http
HTTP/1.1 200 OK
ETag: "profile-v42"
Cache-Control: private, no-cache
```

### 7.6 If-None-Match

What it does:

- client sends previously received `ETag`
- server compares it with current version

Example request:

```http
If-None-Match: "appjs-v17"
```

Possible outcomes:

- unchanged -> `304 Not Modified`
- changed -> `200 OK` with new body

curl example:

```bash
curl -i https://cdn.example.com/app.js \
  -H 'If-None-Match: "appjs-v17"'
```

### 7.7 If-Modified-Since

What it does:

- validator based on timestamp

Example:

```http
If-Modified-Since: Tue, 14 Jan 2025 08:00:00 GMT
```

Why it matters:

- simpler than `ETag`
- useful for static files

Potential weakness:

- timestamps may be less precise than hashes

### 7.8 Authorization

Authentication data often travels in `Authorization`.

#### Basic auth

Example:

```http
Authorization: Basic YWRtaW46c2VjcmV0
```

Meaning:

- base64 of `username:password`
- only safe over HTTPS

curl example:

```bash
curl -i https://api.example.com/admin -u admin:secret
```

#### Bearer token

Example:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Meaning:

- token proves access rights
- common with OAuth2 and JWT-based APIs

curl example:

```bash
curl -i https://api.example.com/me \
  -H 'Authorization: Bearer demo-token'
```

#### API key

Example using `Authorization`:

```http
Authorization: ApiKey 8f9c6f3b8e8c
```

Example using custom header:

```http
X-API-Key: 8f9c6f3b8e8c
```

Operational advice:

- never log secrets in full
- rotate exposed keys immediately
- prefer HTTPS always

### 7.9 Cookie

What it does:

- browser sends state back to server

Example:

```http
Cookie: session=abc123; theme=dark
```

Use cases:

- session identifier
- CSRF token
- user preferences
- analytics values

Why it matters:

- cookies are sent on many requests automatically
- huge cookies waste bandwidth
- cookies affect caching behavior

### 7.10 Set-Cookie

What it does:

- server instructs browser to store a cookie

Example:

```http
Set-Cookie: session=xyz789; Path=/; HttpOnly; Secure; SameSite=Lax
```

Important attributes:

- `Path`
- `Domain`
- `Expires`
- `Max-Age`
- `HttpOnly`
- `Secure`
- `SameSite`

What they mean:

- `HttpOnly` blocks JavaScript access
- `Secure` means HTTPS only
- `SameSite=Lax` helps reduce CSRF risk

### 7.11 CORS headers

Important headers:

- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Headers`
- `Access-Control-Allow-Credentials`
- `Access-Control-Max-Age`

Example response:

```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 600
```

Rule to remember:

If credentials are allowed,

you cannot use wildcard `*` for `Access-Control-Allow-Origin`.

### 7.12 Security headers

#### Content-Security-Policy

What it does:

- tells browser which content sources are allowed

Example:

```http
Content-Security-Policy: default-src 'self'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none'
```

#### X-Frame-Options

What it does:

- controls whether page can be framed

Example:

```http
X-Frame-Options: DENY
```

#### Strict-Transport-Security

What it does:

- tells browser to keep using HTTPS

Example:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

#### X-Content-Type-Options

What it does:

- stops MIME sniffing

Example:

```http
X-Content-Type-Options: nosniff
```

### 7.13 Header debugging checklist

When a response looks wrong,

check:

- `Content-Type`
- `Cache-Control`
- `ETag`
- `Set-Cookie`
- `Vary`
- `Location`
- CORS headers
- security headers

---

## 8. Caching — Visual Flow

Caching is one of the biggest performance wins in web systems.

### 📸 HTTP Caching Flow
![HTTP Caching](https://upload.wikimedia.org/wikipedia/commons/a/a1/HTTPSCachingAction.svg)
> *Source: Wikimedia Commons — HTTP caching decision flow*

It is also a common source of confusing bugs.

```mermaid
graph TD
    REQ["Browser Request"] --> CACHE{"Is it in<br/>browser cache?"}
    CACHE -->|"No"| SERVER["Send to server"]
    CACHE -->|"Yes"| FRESH{"Is cache<br/>still fresh?"}
    FRESH -->|"Yes (max-age ok)"| USE["✅ Use cached version<br/>(no network request!)"]
    FRESH -->|"No (expired)"| VALIDATE{"Validate with server<br/>If-None-Match / If-Modified-Since"}
    VALIDATE -->|"304 Not Modified"| USE2["✅ Use cached version"]
    VALIDATE -->|"200 OK (new content)"| UPDATE["Update cache, use new"]
    SERVER --> STORE["Store in cache<br/>(if cacheable)"]
```

### 8.1 Freshness model

A response can be:

- not cached
- cached and fresh
- cached but stale
- stale and revalidated
- replaced with new content

### 8.2 Strong cache example

Versioned asset:

```http
Cache-Control: public, max-age=31536000, immutable
```

Typical use:

- `/app.4f92ac7.js`
- `/styles.2b8d1aa.css`
- fingerprinted font files

### 8.3 Revalidation example

First response:

```http
HTTP/1.1 200 OK
ETag: "profile-v42"
Cache-Control: private, no-cache
```

Later request:

```http
GET /profile HTTP/1.1
If-None-Match: "profile-v42"
```

Possible reply:

```http
HTTP/1.1 304 Not Modified
ETag: "profile-v42"
```

### 8.4 Cache mistakes to avoid

- caching personalized HTML publicly
- forgetting `Vary: Origin` for CORS-sensitive content
- forgetting `Vary: Accept-Encoding` when compression varies
- serving stale API data without realizing it
- using long max-age on unversioned assets

### 8.5 Browser cache vs CDN cache

| Layer | What it caches | Controlled by |
|---|---|---|
| browser | user-local resources | response headers and browser rules |
| shared proxy | responses for many users | cache headers and proxy policy |
| CDN edge | globally cached content | origin headers and CDN config |

### 8.6 Conditional request flow

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: GET /app.js with If-None-Match
    alt unchanged
        S-->>B: 304 Not Modified
        B->>B: Use cached body
    else changed
        S-->>B: 200 OK + new body + new ETag
        B->>B: Replace cached copy
    end
```

---

## 9. CORS — How Cross-Origin Requests Work

CORS matters only for browsers.

Server-to-server clients like `curl` do not enforce browser CORS rules.

```mermaid
sequenceDiagram
    participant Browser
    participant App as app.example.com
    participant API as api.other.com
    
    Note over Browser,API: Simple Request (GET, POST with simple headers)
    Browser->>API: GET /data<br/>Origin: https://app.example.com
    API-->>Browser: 200 OK<br/>Access-Control-Allow-Origin: https://app.example.com
    
    Note over Browser,API: Preflight Request (PUT, DELETE, custom headers)
    Browser->>API: OPTIONS /data<br/>Origin: https://app.example.com<br/>Access-Control-Request-Method: PUT
    API-->>Browser: 204 No Content<br/>Access-Control-Allow-Origin: ...<br/>Access-Control-Allow-Methods: PUT
    Browser->>API: PUT /data<br/>Origin: https://app.example.com
    API-->>Browser: 200 OK
```

### 9.1 What is an origin?

An origin is:

- scheme
- host
- port

These two are different origins:

- `https://app.example.com`
- `https://api.example.com`

These are also different origins:

- `https://app.example.com`
- `http://app.example.com`

### 9.2 Simple request example

Browser request:

```http
GET /data HTTP/1.1
Origin: https://app.example.com
```

Server response:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Content-Type: application/json

{"message":"ok"}
```

### 9.3 Preflight request example

Browser sends preflight first:

```http
OPTIONS /data HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization, Content-Type
```

Server replies:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 600
```

Then actual request is allowed.

### 9.4 Common CORS failures

- no `Access-Control-Allow-Origin`
- method missing from `Access-Control-Allow-Methods`
- custom header missing from `Access-Control-Allow-Headers`
- wildcard origin combined with credentials
- proxy strips `Origin`

### 9.5 curl testing for CORS

```bash
curl -i https://api.other.com/data \
  -H 'Origin: https://app.example.com'
```

Preflight test:

```bash
curl -i https://api.other.com/data \
  -X OPTIONS \
  -H 'Origin: https://app.example.com' \
  -H 'Access-Control-Request-Method: PUT' \
  -H 'Access-Control-Request-Headers: Authorization, Content-Type'
```

---
