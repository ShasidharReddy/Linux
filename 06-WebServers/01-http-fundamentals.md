# HTTP Fundamentals

> **📌 Disclaimer**: Any third-party logos, screenshots, or diagrams referenced in this document are used for educational purposes only. All trademarks belong to their respective owners.


A deep visual guide to how HTTP actually works.

This version is intentionally long.

It is designed to be read as:

- a conceptual walkthrough
- a protocol reference
- a debugging guide
- a curl practice workbook
- a systems view of browsers, servers, proxies, caches, and APIs

---

## Table of Contents

1. What Happens When You Type a URL — Complete Visual
2. HTTP Request Anatomy — Visual Breakdown
3. HTTP Response Anatomy — Visual Breakdown
4. HTTP Methods — Visual with Use Cases
5. HTTP Status Codes — Visual Decision Tree
6. HTTP/1.0 vs HTTP/1.1 vs HTTP/2 vs HTTP/3 — Visual Comparison
7. HTTP Headers Deep Dive
8. Caching — Visual Flow
9. CORS — How Cross-Origin Requests Work
10. REST API Patterns — Visual
11. Load Balancing HTTP — Visual
12. HTTP Debugging with curl
13. WebSocket — HTTP Upgrade Visual
14. DNS, TCP, TLS, and HTTP Together
15. Browser Rendering After the Response
16. Proxies, CDNs, and Reverse Proxies
17. Hands-On Debugging Scenarios
18. Glossary and Rapid Review

---

## 1. What Happens When You Type a URL — Complete Visual

A browser does much more than “send a GET request”.

### 📸 HTTP Request-Response Cycle
![HTTP Request Response](https://upload.wikimedia.org/wikipedia/commons/0/09/HTTP-request-response.png)
> *Source: Wikimedia Commons — HTTP request-response model*

A real page load is a chain of dependent events.

If any step fails,

the page fails.

The steps usually look like this.

```mermaid
graph TD
    A["1. User types URL<br/>https://www.example.com/page"] --> B["2. Browser parses URL<br/>Protocol: https<br/>Host: www.example.com<br/>Path: /page"]
    B --> C["3. DNS Resolution<br/>www.example.com → 93.184.216.34"]
    C --> D["4. TCP Connection<br/>3-way handshake to port 443"]
    D --> E["5. TLS Handshake<br/>Certificate verification<br/>Key exchange"]
    E --> F["6. HTTP Request<br/>GET /page HTTP/1.1<br/>Host: www.example.com"]
    F --> G["7. Server Processing<br/>Route → Controller → DB → Template"]
    G --> H["8. HTTP Response<br/>200 OK<br/>Content-Type: text/html"]
    H --> I["9. Browser Rendering<br/>Parse HTML → Load CSS/JS → Paint"]
```

### 1.1 Step-by-step explanation

#### Step 1: User types a URL

Example:

```text
https://www.example.com/page
```

The browser receives a string.

It must decide:

- which protocol to use
- which host to contact
- which port to use
- which path to request
- whether there is a query string
- whether there is a fragment

#### Step 2: Browser parses the URL

For this URL:

```text
https://www.example.com:443/page?lang=en#intro
```

The pieces are:

- Scheme: `https`
- Host: `www.example.com`
- Port: `443`
- Path: `/page`
- Query string: `lang=en`
- Fragment: `intro`

Important note:

The fragment is not sent to the server.

It is handled by the browser.

#### Step 3: DNS resolution

The browser needs an IP address.

It may check:

- browser cache
- OS resolver cache
- local hosts file
- recursive DNS resolver
- authoritative DNS servers

Example lookup:

```bash
dig +short www.example.com
```

Example output:

```text
93.184.216.34
```

If DNS fails,

HTTP never starts.

#### Step 4: TCP connection

For HTTPS,

the browser usually connects to port `443`.

TCP uses a three-way handshake.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: SYN
    S-->>C: SYN-ACK
    C->>S: ACK
    Note over C,S: TCP connection established
```

If the server is not listening,

you may see:

- connection refused
- timeout
- network unreachable

#### Step 5: TLS handshake

HTTPS means HTTP inside TLS.

The client and server negotiate:

- TLS version
- cipher suite
- certificate chain
- server identity
- session keys

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: ClientHello
    S-->>C: ServerHello
    S-->>C: Certificate
    S-->>C: Key exchange params
    C->>S: Key exchange
    C->>S: Finished
    S-->>C: Finished
    Note over C,S: Encrypted HTTP starts now
```

Common TLS failure reasons:

- expired certificate
- hostname mismatch
- untrusted CA
- unsupported TLS version
- SNI misconfiguration

#### Step 6: HTTP request

After the secure channel exists,

the browser sends the request.

Example:

```http
GET /page HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Encoding: gzip, br
Connection: keep-alive
```

#### Step 7: Server processing

A typical application path is:

- edge server accepts the request
- reverse proxy routes it
- application framework matches the route
- controller or handler runs
- cache may be checked
- database may be queried
- HTML or JSON is generated

#### Step 8: HTTP response

The server sends:

- status line
- headers
- body

Example:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1582
Cache-Control: no-cache

<!DOCTYPE html>
<html>...</html>
```

#### Step 9: Browser rendering

The browser then:

- parses HTML
- builds the DOM
- discovers CSS, JS, images, fonts
- sends more HTTP requests
- parses CSS
- builds the CSSOM
- executes JavaScript
- performs layout
- paints pixels

### 1.2 Full end-to-end timeline

```mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant D as DNS
    participant W as Web Server
    participant A as App
    participant DB as Database

    U->>B: Type https://www.example.com/page
    B->>B: Parse URL
    B->>D: Resolve www.example.com
    D-->>B: 93.184.216.34
    B->>W: TCP connect port 443
    B->>W: TLS handshake
    B->>W: GET /page HTTP request
    W->>A: Forward request
    A->>DB: Query data
    DB-->>A: Rows returned
    A-->>W: HTML generated
    W-->>B: 200 OK + HTML
    B->>B: Parse HTML
    B->>W: Fetch CSS, JS, images
    W-->>B: Static assets
    B->>B: Layout and paint
```

### 1.3 What can go wrong at each stage

| Stage | Typical symptom | Example cause |
|---|---|---|
| URL parse | bad URL error | malformed scheme |
| DNS | could not resolve host | missing DNS record |
| TCP | connection refused | server not listening |
| TCP | timeout | firewall drop |
| TLS | certificate error | wrong SAN or expired cert |
| HTTP routing | 404 | path not mapped |
| HTTP auth | 401 or 403 | missing token or denied user |
| Upstream app | 502 or 503 | crashed backend |
| Database | 500 or 504 | query failure or timeout |
| Browser render | blank page | broken JS bundle |

### 1.4 Mental model

Think in layers.

- DNS answers “where?”
- TCP answers “can we connect?”
- TLS answers “is it secure and who are you?”
- HTTP answers “what resource do you want?”
- HTML, CSS, and JS answer “what should the user see?”

---

## 2. HTTP Request Anatomy — Visual Breakdown

An HTTP request has three major parts.

- request line
- headers
- optional body

### 2.1 Visual code block

```text
┌─────────────────────────────────────────────────┐
│ Request Line                                     │
│ GET /api/users?page=1 HTTP/1.1                  │
│ ↑     ↑                  ↑                       │
│ Method  Path+Query    Version                    │
├─────────────────────────────────────────────────┤
│ Headers                                          │
│ Host: api.example.com                           │
│ Accept: application/json                         │
│ Authorization: Bearer eyJhbGciOi...             │
│ User-Agent: Mozilla/5.0                         │
│ Cookie: session=abc123                           │
│ Content-Type: application/json                   │
├─────────────────────────────────────────────────┤
│ Body (for POST/PUT/PATCH)                        │
│ {"name": "John", "email": "john@example.com"}  │
└─────────────────────────────────────────────────┘
```

### 2.2 Mermaid diagram of the same request

```mermaid
graph TD
    A["Request Line<br/>GET /api/users?page=1 HTTP/1.1"] --> B["Headers<br/>Host: api.example.com<br/>Accept: application/json<br/>Authorization: Bearer eyJhbGciOi...<br/>User-Agent: Mozilla/5.0<br/>Cookie: session=abc123<br/>Content-Type: application/json"]
    B --> C["Body<br/>JSON payload with name and email fields"]
```

### 2.3 Raw HTTP example

```http
POST /api/users?page=1 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOi...
User-Agent: Mozilla/5.0
Cookie: session=abc123
Content-Type: application/json
Content-Length: 45

{"name":"John","email":"john@example.com"}
```

### 2.4 Request line explained

The request line is:

```text
GET /api/users?page=1 HTTP/1.1
```

It contains:

- method
- target
- version

#### Method

The method tells the server what kind of action the client wants.

Examples:

- `GET` for read
- `POST` for create or submit
- `PUT` for replace
- `PATCH` for partial update
- `DELETE` for remove

#### Path and query

`/api/users?page=1`

The path identifies the resource.

The query string adds parameters.

Important rule:

A query string is not the same thing as the body.

#### Version

`HTTP/1.1`

This tells the server which HTTP protocol syntax and semantics the client expects.

### 2.5 Headers explained

Headers are metadata.

They describe the request.

Common request headers:

| Header | What it means | Why it matters |
|---|---|---|
| `Host` | target host | required in HTTP/1.1 |
| `Accept` | preferred response type | content negotiation |
| `Authorization` | credentials | API access control |
| `User-Agent` | client identity string | debugging and analytics |
| `Cookie` | browser state | sessions and preferences |
| `Content-Type` | body format | tells server how to parse body |
| `Content-Length` | byte size of body | framing in HTTP/1.1 |

### 2.6 Body explained

The body is optional.

It is commonly used with:

- `POST`
- `PUT`
- `PATCH`

Typical body formats:

- JSON
- form data
- multipart file upload
- XML
- plain text

### 2.7 Request example with curl

Command:

```bash
curl -i https://api.example.com/users \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer demo-token'
```

Representative request seen by the server:

```http
GET /users HTTP/1.1
Host: api.example.com
User-Agent: curl/8.7.1
Accept: application/json
Authorization: Bearer demo-token
```

### 2.8 Request example with JSON body

Command:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John","email":"john@example.com"}'
```

Representative request:

```http
POST /users HTTP/1.1
Host: api.example.com
User-Agent: curl/8.7.1
Accept: */*
Content-Type: application/json
Content-Length: 43

{"name":"John","email":"john@example.com"}
```

### 2.9 Common mistakes in requests

- missing `Host` header in HTTP/1.1
- wrong `Content-Type`
- wrong path
- wrong HTTP method
- invalid JSON body
- missing auth header
- huge cookies causing header size errors

### 2.10 Request processing path inside a server

```mermaid
graph LR
    A["TCP socket"] --> B["TLS termination"] --> C["HTTP parser"] --> D["Router"] --> E["Auth middleware"] --> F["Handler"] --> G["Database or cache"]
```

---

## 3. HTTP Response Anatomy — Visual Breakdown

The response also has three major parts.

- status line
- headers
- optional body

### 3.1 Visual code block

```text
┌─────────────────────────────────────────────────┐
│ Status Line                                      │
│ HTTP/1.1 200 OK                                  │
│ ↑        ↑    ↑                                  │
│ Version  Code Reason                             │
├─────────────────────────────────────────────────┤
│ Headers                                          │
│ Content-Type: text/html; charset=UTF-8          │
│ Content-Length: 1234                              │
│ Set-Cookie: session=xyz789; HttpOnly; Secure    │
│ Cache-Control: max-age=3600                      │
│ X-Frame-Options: DENY                            │
│ Strict-Transport-Security: max-age=31536000     │
├─────────────────────────────────────────────────┤
│ Body                                             │
│ <!DOCTYPE html><html>...</html>                 │
└─────────────────────────────────────────────────┘
```

### 3.2 Mermaid diagram of the same response

```mermaid
graph TD
    A["Status Line<br/>HTTP/1.1 200 OK"] --> B["Headers<br/>Content-Type: text/html; charset=UTF-8<br/>Content-Length: 1234<br/>Set-Cookie: session=xyz789; HttpOnly; Secure<br/>Cache-Control: max-age=3600<br/>X-Frame-Options: DENY<br/>Strict-Transport-Security: max-age=31536000"]
    B --> C["Body<br/>HTML document response"]
```

### 3.3 Raw response example

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Set-Cookie: session=xyz789; HttpOnly; Secure
Cache-Control: max-age=3600
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000

<!DOCTYPE html>
<html>...</html>
```

### 3.4 Status line explained

The status line has:

- version
- status code
- reason phrase

`HTTP/1.1 200 OK`

Means:

- HTTP version is 1.1
- status code is 200
- reason is OK

The client mainly relies on the status code.

### 3.5 Response headers explained

Headers tell the client how to interpret the response.

| Header | Meaning | Why it matters |
|---|---|---|
| `Content-Type` | body MIME type | parsing behavior |
| `Content-Length` | body size | framing and progress |
| `Set-Cookie` | set cookie in browser | session or app state |
| `Cache-Control` | cache rules | browser and CDN behavior |
| `X-Frame-Options` | anti-clickjacking | browser security |
| `Strict-Transport-Security` | force future HTTPS | HTTPS hardening |

### 3.6 Response body explained

The body may contain:

- HTML
- JSON
- CSS
- JavaScript
- image bytes
- PDF data
- nothing at all for `204 No Content`

### 3.7 Example response from curl

Command:

```bash
curl -i https://www.example.com/
```

Representative output:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1582
Cache-Control: no-cache
Server: nginx

<!DOCTYPE html>
<html>
<head><title>Example</title></head>
<body>Hello</body>
</html>
```

### 3.8 Example JSON response

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 56
Cache-Control: no-store

{"id":123,"name":"John","email":"john@example.com"}
```

### 3.9 Response generation path

```mermaid
graph LR
    A["Handler returns data"] --> B["Framework serializes body"] --> C["Headers added"] --> D["Compression maybe applied"] --> E["Response written to socket"]
```

---

## 4. HTTP Methods — Visual with Use Cases

Methods communicate intent.

### 📸 REST API Methods
![REST Methods](https://upload.wikimedia.org/wikipedia/commons/5/57/Http_request_telnet_ubuntu.png)
> *Source: Wikimedia Commons — HTTP request example*

They do not magically enforce correctness.

But they are still central to good API design.

```mermaid
graph TD
    subgraph Safe["Safe Methods (No Side Effects)"]
        GET["GET<br/>Retrieve resource<br/>GET /users/123"]
        HEAD["HEAD<br/>Headers only<br/>HEAD /users/123"]
        OPTIONS["OPTIONS<br/>Allowed methods<br/>OPTIONS /api/users"]
    end
    
    subgraph Unsafe["Unsafe Methods (Side Effects)"]
        POST["POST<br/>Create resource<br/>POST /users"]
        PUT["PUT<br/>Replace resource<br/>PUT /users/123"]
        PATCH["PATCH<br/>Partial update<br/>PATCH /users/123"]
        DELETE["DELETE<br/>Remove resource<br/>DELETE /users/123"]
    end
```

### 4.1 Safe vs unsafe

Safe means the method should not change server state.

Common safe methods:

- `GET`
- `HEAD`
- `OPTIONS`

Unsafe means it can change server state.

Common unsafe methods:

- `POST`
- `PUT`
- `PATCH`
- `DELETE`

### 4.2 Idempotent vs non-idempotent

Idempotent means:

repeating the same request leaves the resource in the same final state.

Examples:

- `GET` is idempotent
- `PUT` is usually idempotent
- `DELETE` is usually idempotent
- `POST` is usually not idempotent

### 4.3 Method summary table

| Method | Purpose | Safe | Idempotent | Common success code |
|---|---|---:|---:|---|
| `GET` | read | yes | yes | `200` |
| `HEAD` | read headers only | yes | yes | `200` |
| `OPTIONS` | capability discovery | yes | yes | `204` |
| `POST` | create or submit | no | no | `201` or `202` |
| `PUT` | replace | no | yes | `200` or `204` |
| `PATCH` | partial update | no | usually no | `200` or `204` |
| `DELETE` | remove | no | yes | `204` |

### 4.4 GET

Use `GET` to retrieve a resource.

Example use cases:

- fetch an HTML page
- fetch a JSON object
- fetch an image
- fetch a list of users

curl example:

```bash
curl -i https://api.example.com/users/123
```

Representative request:

```http
GET /users/123 HTTP/1.1
Host: api.example.com
User-Agent: curl/8.7.1
Accept: */*
```

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 52

{"id":123,"name":"John","email":"john@example.com"}
```

Key points:

- should not mutate state
- may be cached
- may be retried more safely than `POST`

### 4.5 HEAD

Use `HEAD` to ask for the same headers as `GET`,

but without the body.

Typical use cases:

- health checks
- checking file size
- checking `ETag`
- checking `Last-Modified`

curl example:

```bash
curl -I https://api.example.com/users/123
```

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 52
ETag: "u123-v5"
Last-Modified: Tue, 14 Jan 2025 08:00:00 GMT
```

### 4.6 OPTIONS

Use `OPTIONS` to ask what is allowed.

Typical use cases:

- API discovery
- browser CORS preflight
- checking allowed methods

curl example:

```bash
curl -i -X OPTIONS https://api.example.com/users
```

Representative response:

```http
HTTP/1.1 204 No Content
Allow: GET, POST, OPTIONS
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
```

### 4.7 POST

Use `POST` to create a new resource,

or submit data for processing.

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John","email":"john@example.com"}'
```

Representative request:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 43

{"name":"John","email":"john@example.com"}
```

Representative response:

```http
HTTP/1.1 201 Created
Location: /users/123
Content-Type: application/json

{"id":123,"name":"John","email":"john@example.com"}
```

### 4.8 PUT

Use `PUT` to replace a resource representation.

If you send the same `PUT` again,

the final state should usually be the same.

curl example:

```bash
curl -i https://api.example.com/users/123 \
  -X PUT \
  -H 'Content-Type: application/json' \
  -d '{"name":"John Smith","email":"john@example.com","role":"editor"}'
```

Representative request:

```http
PUT /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 64

{"name":"John Smith","email":"john@example.com","role":"editor"}
```

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":123,"name":"John Smith","email":"john@example.com","role":"editor"}
```

### 4.9 PATCH

Use `PATCH` for partial updates.

You send only the changed fields.

curl example:

```bash
curl -i https://api.example.com/users/123 \
  -X PATCH \
  -H 'Content-Type: application/json' \
  -d '{"role":"admin"}'
```

Representative request:

```http
PATCH /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 16

{"role":"admin"}
```

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":123,"name":"John","email":"john@example.com","role":"admin"}
```

### 4.10 DELETE

Use `DELETE` to remove a resource.

curl example:

```bash
curl -i https://api.example.com/users/123 -X DELETE
```

Representative request:

```http
DELETE /users/123 HTTP/1.1
Host: api.example.com
User-Agent: curl/8.7.1
Accept: */*
```

Representative response:

```http
HTTP/1.1 204 No Content
```

### 4.11 TRACE and CONNECT

These are less common in everyday app APIs,

but still part of HTTP history and infrastructure.

#### TRACE

Purpose:

- diagnostic loopback
- server echoes the request

Why often disabled:

- security risk
- rarely needed in normal production apps

Example:

```bash
curl -i https://api.example.com/ -X TRACE
```

Possible response:

```http
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST
```

#### CONNECT

Purpose:

- ask a proxy to open a tunnel
- common with HTTPS through proxies

Conceptual example:

```http
CONNECT api.example.com:443 HTTP/1.1
Host: api.example.com:443
```

Possible response:

```http
HTTP/1.1 200 Connection Established
```

### 4.12 Method selection cheat sheet

| If you want to... | Usually use |
|---|---|
| read a page | `GET` |
| read an API record | `GET` |
| inspect headers only | `HEAD` |
| ask what is allowed | `OPTIONS` |
| create a record | `POST` |
| replace a record | `PUT` |
| update one field | `PATCH` |
| remove a record | `DELETE` |

### 4.13 Visual lifecycle of methods in a REST API

```mermaid
graph LR
    A["POST /users<br/>Create"] --> B["201 Created<br/>Location: /users/123"]
    B --> C["GET /users/123<br/>Read"]
    C --> D["PATCH /users/123<br/>Partial update"]
    D --> E["PUT /users/123<br/>Full replace"]
    E --> F["DELETE /users/123<br/>Remove"]
```

---

## 5. HTTP Status Codes — Visual Decision Tree

Status codes tell the client what happened.

They do not explain everything,

but they give the first big clue.

```mermaid
graph TD
    REQ["HTTP Request"] --> CODE{"Status Code?"}
    CODE -->|"1xx"| INFO["ℹ️ Informational<br/>100 Continue<br/>101 Switching Protocols"]
    CODE -->|"2xx"| SUCCESS["✅ Success<br/>200 OK<br/>201 Created<br/>204 No Content"]
    CODE -->|"3xx"| REDIRECT["↗️ Redirect<br/>301 Moved Permanently<br/>302 Found<br/>304 Not Modified"]
    CODE -->|"4xx"| CLIENT["❌ Client Error<br/>400 Bad Request<br/>401 Unauthorized<br/>403 Forbidden<br/>404 Not Found<br/>429 Too Many Requests"]
    CODE -->|"5xx"| SERVER["💥 Server Error<br/>500 Internal Server Error<br/>502 Bad Gateway<br/>503 Service Unavailable<br/>504 Gateway Timeout"]
```

### 5.1 Family overview

| Family | Meaning | Typical interpretation |
|---|---|---|
| `1xx` | informational | request is in progress |
| `2xx` | success | request worked |
| `3xx` | redirection | client needs another step or can reuse cache |
| `4xx` | client error | request is invalid or not allowed |
| `5xx` | server error | server or upstream failed |

### 5.2 How to read a status code during debugging

Ask:

- did the request reach the server?
- did the server understand it?
- did auth pass?
- did routing succeed?
- did the app or upstream crash?
- did a cache or proxy alter the result?

### 5.3 100 Continue

When it occurs:

- client wants to send a large body
- client uses `Expect: 100-continue`
- server says “okay, send the body”

What it means:

- initial headers are acceptable
- continue uploading

How to fix it if broken:

- verify proxy supports `Expect`
- verify body size limits
- disable `Expect` if infrastructure mishandles it

curl example:

```bash
curl -i https://upload.example.com/files \
  -H 'Expect: 100-continue' \
  -T ./large-file.iso
```

Representative exchange:

```http
HTTP/1.1 100 Continue

HTTP/1.1 201 Created
Content-Type: application/json

{"status":"uploaded"}
```

### 5.4 101 Switching Protocols

When it occurs:

- protocol upgrade requested
- common with WebSocket

What it means:

- server agrees to switch protocols
- HTTP handshake ends
- new protocol takes over

How to fix it if broken:

- check `Upgrade` and `Connection` headers
- ensure reverse proxy passes upgrade headers
- verify backend supports WebSocket

curl example:

```bash
curl -i https://chat.example.com/ \
  -H 'Connection: Upgrade' \
  -H 'Upgrade: websocket'
```

Representative response:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

### 5.5 200 OK

When it occurs:

- normal successful request
- page rendered
- API read succeeded
- update returned resource body

What it means:

- request succeeded
- body usually present

How to fix it if the wrong content is returned:

- verify route handler
- verify content negotiation
- inspect cache behavior
- inspect upstream response mapping

curl example:

```bash
curl -i https://api.example.com/users/123
```

Representative response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":123,"name":"John"}
```

### 5.6 201 Created

When it occurs:

- successful resource creation
- common with `POST`

What it means:

- new resource exists now
- `Location` header is often included

How to fix incorrect behavior:

- ensure creation really happened once
- add idempotency keys for duplicate-sensitive flows
- return `Location` header when useful

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John"}'
```

Representative response:

```http
HTTP/1.1 201 Created
Location: /users/123
Content-Type: application/json

{"id":123,"name":"John"}
```

### 5.7 204 No Content

When it occurs:

- successful delete
- successful update with no response body
- successful preflight or health endpoint

What it means:

- request succeeded
- there is no body to parse

How to fix client-side issues:

- do not try to parse JSON from a `204`
- ensure frontend handles empty body correctly
- ensure API docs say body is empty

curl example:

```bash
curl -i https://api.example.com/users/123 -X DELETE
```

Representative response:

```http
HTTP/1.1 204 No Content
```

### 5.8 206 Partial Content

When it occurs:

- client asks for a byte range
- video playback
- resume download

What it means:

- only part of the resource is returned

How to fix issues:

- verify `Range` header parsing
- verify `Content-Range` correctness
- ensure CDN or proxy preserves range support

curl example:

```bash
curl -i https://cdn.example.com/video.mp4 \
  -H 'Range: bytes=0-999'
```

Representative response:

```http
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-999/73400320
Content-Length: 1000
```

### 5.9 301 Moved Permanently

When it occurs:

- HTTP to HTTPS redirect
- non-canonical host redirect
- permanent URL migration

What it means:

- resource has a new permanent URL
- clients and search engines may cache the redirect

How to fix redirect problems:

- verify `Location` header
- avoid loops
- prefer permanent redirect only when stable
- test host and scheme rewrites carefully

curl example:

```bash
curl -i http://www.example.com/
```

Representative response:

```http
HTTP/1.1 301 Moved Permanently
Location: https://www.example.com/
```

### 5.10 302 Found

When it occurs:

- temporary redirect
- login flow redirect
- legacy framework redirect behavior

What it means:

- client should go somewhere else for now

How to fix bad behavior:

- use `307` or `308` if method preservation matters
- ensure temporary redirect is really temporary
- inspect repeated redirect chains

curl example:

```bash
curl -i https://app.example.com/private
```

Representative response:

```http
HTTP/1.1 302 Found
Location: /login
```

### 5.11 304 Not Modified

When it occurs:

- cache revalidation succeeds
- browser sends `If-None-Match` or `If-Modified-Since`

What it means:

- body not sent again
- client should use cached copy

How to fix cache issues:

- ensure validators are stable
- ensure `ETag` is correct
- ensure `Last-Modified` is meaningful
- add `Cache-Control` and `Vary` intentionally

curl example:

```bash
curl -i https://cdn.example.com/app.js \
  -H 'If-None-Match: "appjs-v17"'
```

Representative response:

```http
HTTP/1.1 304 Not Modified
ETag: "appjs-v17"
Cache-Control: public, max-age=31536000, immutable
```

### 5.12 307 Temporary Redirect

When it occurs:

- temporary redirect where method must stay the same
- upload moved temporarily

What it means:

- follow redirect
- preserve method and body

How to fix issues:

- choose `307` over `302` when preserving method matters
- verify clients follow redirects as expected

curl example:

```bash
curl -i https://upload.example.com/files \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"demo"}'
```

Representative response:

```http
HTTP/1.1 307 Temporary Redirect
Location: https://upload2.example.com/files
```

### 5.13 308 Permanent Redirect

When it occurs:

- permanent redirect where method must stay the same
- permanent API path migration

What it means:

- new canonical URL
- preserve method and body

How to fix issues:

- use only when the move is truly permanent
- verify clients and SDKs support `308`

curl example:

```bash
curl -i https://api.example.com/v1/orders \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"sku":"KB-100"}'
```

Representative response:

```http
HTTP/1.1 308 Permanent Redirect
Location: https://api.example.com/v2/orders
```

### 5.14 400 Bad Request

When it occurs:

- malformed JSON
- missing required parameter
- invalid syntax
- body does not match parser expectations

What it means:

- server cannot process the request as sent

How to fix it:

- validate request schema
- verify JSON encoding
- verify required fields
- inspect server parser logs

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John"'
```

Representative response:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{"error":"invalid JSON payload"}
```

### 5.15 401 Unauthorized

When it occurs:

- no auth token
- invalid token
- expired token
- wrong credentials

What it means:

- authentication is required or failed

How to fix it:

- send correct `Authorization` header
- refresh expired token
- verify secret or public key config
- verify clock skew for JWT validation

curl example:

```bash
curl -i https://api.example.com/me
```

Representative response:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
Content-Type: application/json

{"error":"missing bearer token"}
```

### 5.16 403 Forbidden

When it occurs:

- user is authenticated but not allowed
- WAF blocks request
- IP blocked
- file permissions deny access

What it means:

- server understood the request
- server refuses to authorize it

How to fix it:

- verify RBAC or ACL policy
- verify filesystem permissions
- inspect WAF rules
- verify upstream authorization middleware

curl example:

```bash
curl -i https://api.example.com/admin \
  -H 'Authorization: Bearer user-token'
```

Representative response:

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{"error":"insufficient permissions"}
```

### 5.17 404 Not Found

When it occurs:

- wrong URL
- route does not exist
- resource ID not present
- static file missing

What it means:

- server cannot find the requested resource

How to fix it:

- verify route mapping
- verify deploy artifact exists
- verify path rewriting rules
- verify identifier is correct

curl example:

```bash
curl -i https://api.example.com/users/999999
```

Representative response:

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{"error":"user not found"}
```

### 5.18 405 Method Not Allowed

When it occurs:

- endpoint exists
- method is not supported on that endpoint

What it means:

- path is valid
- method is wrong

How to fix it:

- switch to allowed method
- return `Allow` header from server
- update API docs if behavior is confusing

curl example:

```bash
curl -i https://api.example.com/users/123 -X POST
```

Representative response:

```http
HTTP/1.1 405 Method Not Allowed
Allow: GET, PUT, PATCH, DELETE
Content-Type: application/json

{"error":"method not allowed"}
```

### 5.19 409 Conflict

When it occurs:

- duplicate resource state
- version conflict
- optimistic lock failure
- business rule collision

What it means:

- request is understood
- current server state conflicts with requested action

How to fix it:

- retry with fresh version data
- use `If-Match` for concurrency control
- return conflict details in body

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"email":"john@example.com"}'
```

Representative response:

```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{"error":"email already exists"}
```

### 5.20 415 Unsupported Media Type

When it occurs:

- wrong `Content-Type`
- server expects JSON but client sends XML
- upload type not accepted

What it means:

- server refuses to parse body in that format

How to fix it:

- send correct `Content-Type`
- ensure parser is installed and enabled
- document supported media types clearly

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/xml' \
  -d '<user><name>John</name></user>'
```

Representative response:

```http
HTTP/1.1 415 Unsupported Media Type
Content-Type: application/json

{"error":"expected application/json"}
```

### 5.21 422 Unprocessable Content

When it occurs:

- JSON syntax is valid
- business or validation rules fail

What it means:

- server understood the body format
- data is semantically invalid

How to fix it:

- return field-level validation errors
- validate client-side before submit
- keep schema shared between client and server when possible

curl example:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"email":"not-an-email"}'
```

Representative response:

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/json

{"errors":{"email":"must be a valid email address"}}
```

### 5.22 429 Too Many Requests

When it occurs:

- rate limit exceeded
- abuse protection triggered
- burst traffic exceeds policy

What it means:

- client must slow down

How to fix it:

- respect `Retry-After`
- back off client retries
- tune rate-limit thresholds carefully
- use queues or batching for noisy clients

curl example:

```bash
curl -i https://api.example.com/search?q=linux
```

Representative response:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
Content-Type: application/json

{"error":"rate limit exceeded"}
```

### 5.23 500 Internal Server Error

When it occurs:

- unhandled exception
- template render failure
- code bug
- unexpected dependency failure

What it means:

- server failed unexpectedly

How to fix it:

- inspect application logs
- add structured error handling
- add tracing and request IDs
- test edge cases before deploy

curl example:

```bash
curl -i https://api.example.com/reports/123
```

Representative response:

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{"error":"unexpected server error"}
```

### 5.24 502 Bad Gateway

When it occurs:

- reverse proxy gets invalid upstream response
- backend port wrong
- backend process crashed
- upstream closed socket early

What it means:

- gateway or proxy could not get a valid response from upstream

How to fix it:

- verify backend is listening
- verify proxy upstream config
- inspect backend logs
- align timeout and keep-alive settings

curl example:

```bash
curl -i https://app.example.com/api/orders
```

Representative response:

```http
HTTP/1.1 502 Bad Gateway
Server: nginx
Content-Type: text/html

<html><body><h1>502 Bad Gateway</h1></body></html>
```

### 5.25 503 Service Unavailable

When it occurs:

- maintenance mode
- no healthy backend instances
- overload
- dependency outage

What it means:

- service is temporarily unavailable

How to fix it:

- check health checks and deployment state
- scale backends or reduce load
- show retry guidance
- set `Retry-After` when appropriate

curl example:

```bash
curl -i https://app.example.com/
```

Representative response:

```http
HTTP/1.1 503 Service Unavailable
Retry-After: 120
Content-Type: text/html

<html><body><h1>Service Unavailable</h1></body></html>
```

### 5.26 504 Gateway Timeout

When it occurs:

- upstream app is too slow
- database query is slow
- proxy timeout is shorter than backend work

What it means:

- gateway waited too long for upstream

How to fix it:

- profile slow queries
- align proxy and app timeouts
- add caching
- reduce request fan-out

curl example:

```bash
curl -i https://app.example.com/reports/monthly
```

Representative response:

```http
HTTP/1.1 504 Gateway Timeout
Server: nginx
Content-Type: text/html

<html><body><h1>504 Gateway Timeout</h1></body></html>
```

### 5.27 Fast decision map

| If you see... | First suspicion |
|---|---|
| `301` or `302` | redirect logic |
| `304` | caching validator worked |
| `400` | malformed request |
| `401` | bad or missing authentication |
| `403` | permission or WAF |
| `404` | wrong route or missing resource |
| `409` | state conflict |
| `415` | wrong content type |
| `422` | validation failed |
| `429` | rate limit |
| `500` | app bug |
| `502` | bad upstream response |
| `503` | service unavailable |
| `504` | slow upstream |

---

## 6. HTTP/1.0 vs HTTP/1.1 vs HTTP/2 vs HTTP/3 — Visual Comparison

Each new version tried to reduce latency and improve efficiency.

### 📸 HTTP/1.1 vs HTTP/2 Multiplexing
![HTTP2 Multiplexing](https://upload.wikimedia.org/wikipedia/commons/0/09/HTTP_1.1_vs._HTTP_2.svg)
> *Source: Wikimedia Commons — HTTP/1.1 sequential vs HTTP/2 multiplexed requests*

```mermaid
graph TD
    subgraph HTTP10["HTTP/1.0 (1996)"]
        H10_1["New TCP connection<br/>for EVERY request"] --> H10_2["Close connection<br/>after response"]
    end
    
    subgraph HTTP11["HTTP/1.1 (1997)"]
        H11_1["Keep-Alive:<br/>reuse TCP connection"] --> H11_2["But: Head-of-Line blocking<br/>requests must be sequential"]
    end
    
    subgraph HTTP2["HTTP/2 (2015)"]
        H2_1["Multiplexing:<br/>parallel requests<br/>on single connection"] --> H2_2["Header compression<br/>(HPACK)"] --> H2_3["Server Push"]
    end
    
    subgraph HTTP3["HTTP/3 (2022)"]
        H3_1["QUIC protocol<br/>(UDP-based)"] --> H3_2["No head-of-line<br/>blocking at transport"] --> H3_3["0-RTT connection<br/>resumption"]
    end
```

### 6.1 High-level comparison table

| Version | Transport | Connection behavior | Main gain | Main pain point |
|---|---|---|---|---|
| HTTP/1.0 | TCP | one request per connection | simple | many connections |
| HTTP/1.1 | TCP | persistent connections | fewer handshakes | request serialization |
| HTTP/2 | TCP | multiplexed streams | parallelism | TCP packet loss still hurts all streams |
| HTTP/3 | QUIC over UDP | multiplexed streams with QUIC | better loss handling and faster resume | newer tooling and infrastructure complexity |

### 6.2 Timing diagram: HTTP/1.0

Six assets.

Six TCP connections.

Slow.

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: TCP + GET /index.html
    S-->>B: HTML
    B->>S: TCP + GET /styles.css
    S-->>B: CSS
    B->>S: TCP + GET /app.js
    S-->>B: JS
    B->>S: TCP + GET /logo.png
    S-->>B: PNG
    B->>S: TCP + GET /font.woff2
    S-->>B: Font
    B->>S: TCP + GET /api/data
    S-->>B: JSON
```

### 6.3 Timing diagram: HTTP/1.1

Six assets.

One TCP connection.

Sequential requests on a reused connection.

Better,

but still blocked by order.

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: TCP connect
    B->>S: GET /index.html
    S-->>B: HTML
    B->>S: GET /styles.css
    S-->>B: CSS
    B->>S: GET /app.js
    S-->>B: JS
    B->>S: GET /logo.png
    S-->>B: PNG
    B->>S: GET /font.woff2
    S-->>B: Font
    B->>S: GET /api/data
    S-->>B: JSON
```

### 6.4 Timing diagram: HTTP/2

Six assets.

One TCP connection.

Parallel streams.

Fast.

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: TCP + TLS connect
    B->>S: Stream 1 GET /index.html
    B->>S: Stream 3 GET /styles.css
    B->>S: Stream 5 GET /app.js
    B->>S: Stream 7 GET /logo.png
    B->>S: Stream 9 GET /font.woff2
    B->>S: Stream 11 GET /api/data
    S-->>B: Stream 1 HTML
    S-->>B: Stream 3 CSS
    S-->>B: Stream 5 JS
    S-->>B: Stream 7 PNG
    S-->>B: Stream 9 Font
    S-->>B: Stream 11 JSON
```

### 6.5 Timing diagram: HTTP/3

Six assets.

One QUIC connection.

Parallel streams.

No transport-level head-of-line blocking across streams.

Fastest under loss.

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    B->>S: QUIC connect + TLS 1.3
    B->>S: Stream A GET /index.html
    B->>S: Stream B GET /styles.css
    B->>S: Stream C GET /app.js
    B->>S: Stream D GET /logo.png
    B->>S: Stream E GET /font.woff2
    B->>S: Stream F GET /api/data
    S-->>B: Stream A HTML
    S-->>B: Stream B CSS
    S-->>B: Stream C JS
    S-->>B: Stream D PNG
    S-->>B: Stream E Font
    S-->>B: Stream F JSON
```

### 6.6 ASCII view of the same idea

```text
HTTP/1.0
[conn1 HTML] [conn2 CSS] [conn3 JS] [conn4 PNG] [conn5 Font] [conn6 API]

HTTP/1.1
[one TCP connection] -> HTML -> CSS -> JS -> PNG -> Font -> API

HTTP/2
[one TCP connection] -> HTML | CSS | JS | PNG | Font | API

HTTP/3
[one QUIC connection] -> HTML | CSS | JS | PNG | Font | API
(with better loss isolation)
```

### 6.7 Why HTTP/1.1 felt slow on asset-heavy pages

Because browsers had to do more connection management,

or queue requests.

That meant:

- more handshake overhead
- more socket churn
- more latency before assets started

### 6.8 Why HTTP/2 mattered

HTTP/2 introduced:

- multiplexing
- header compression
- binary framing
- prioritization

That reduced:

- redundant connection setup
- request queueing pain
- header bloat across repeated requests

### 6.9 Why HTTP/3 matters

HTTP/3 uses QUIC.

QUIC runs over UDP.

The big win is not “UDP is magically faster”.

The big win is better connection behavior under packet loss,

plus faster secure setup and stream independence.

### 6.10 Version testing with curl

```bash
curl --http1.0 -I https://example.com
curl --http1.1 -I https://example.com
curl --http2 -I https://example.com
curl --http3 -I https://example.com
```

### 6.11 Practical rule of thumb

- understand HTTP/1.1 deeply because infrastructure still uses it everywhere
- prefer HTTP/2 or HTTP/3 at the public edge when available
- test real latency rather than assuming a protocol upgrade fixes everything

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

## 10. REST API Patterns — Visual

REST is about resources,

representations,

and standard HTTP semantics.

### 10.1 Resource map

```mermaid
graph TD
    A["/users"] --> B["GET list users"]
    A --> C["POST create user"]
    D["/users/{id}"] --> E["GET read one user"]
    D --> F["PUT replace user"]
    D --> G["PATCH update user"]
    D --> H["DELETE remove user"]
    I["/users/{id}/orders"] --> J["GET list user orders"]
    I --> K["POST create user order"]
```

### 10.2 Example API design

| Endpoint | Method | Meaning | Typical response |
|---|---|---|---|
| `/users` | `GET` | list users | `200` |
| `/users` | `POST` | create user | `201` |
| `/users/123` | `GET` | read user | `200` |
| `/users/123` | `PUT` | replace user | `200` or `204` |
| `/users/123` | `PATCH` | partial update | `200` or `204` |
| `/users/123` | `DELETE` | delete user | `204` |
| `/users/123/orders` | `GET` | list orders for user | `200` |
| `/users/123/orders` | `POST` | create order for user | `201` |

### 10.3 Example create flow

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API
    participant DB as Database
    C->>API: POST /users
    API->>DB: INSERT new user
    DB-->>API: id=123
    API-->>C: 201 Created + Location: /users/123
```

### 10.4 Good REST habits

- nouns for resources
- plural collections are common
- status codes match actual outcome
- use headers for metadata
- avoid verbs in paths when resource semantics fit better

---

## 11. Load Balancing HTTP — Visual

Load balancers spread requests across backend servers.

They may also:

- terminate TLS
- perform health checks
- add headers
- enforce rate limits
- retry idempotent requests

```mermaid
graph TD
    C["Clients"] --> LB["Load Balancer<br/>TLS termination<br/>health checks"]
    LB --> S1["Web Server 1"]
    LB --> S2["Web Server 2"]
    LB --> S3["Web Server 3"]
    S1 --> APP1["App Instance 1"]
    S2 --> APP2["App Instance 2"]
    S3 --> APP3["App Instance 3"]
    APP1 --> DB["Database"]
    APP2 --> DB
    APP3 --> DB
```

### 11.1 Common balancing strategies

- round robin
- least connections
- weighted distribution
- hash by IP or cookie

### 11.2 Request path through a load balancer

```mermaid
sequenceDiagram
    participant U as User
    participant LB as Load Balancer
    participant W as Web Server
    participant A as App
    U->>LB: HTTPS request
    LB->>LB: health check backend pool
    LB->>W: forward request
    W->>A: app processing
    A-->>W: response body
    W-->>LB: response
    LB-->>U: final HTTP response
```

### 11.3 Important forwarding headers

Examples:

```http
X-Forwarded-For: 203.0.113.10
X-Forwarded-Proto: https
X-Forwarded-Host: www.example.com
```

Why they matter:

- app sees real client IP
- app knows original scheme was HTTPS
- app can generate correct absolute URLs

---

## 12. HTTP Debugging with curl

`curl` is one of the best HTTP learning and debugging tools.

### 12.1 `curl -v` for verbose protocol details

Command:

```bash
curl -v https://www.example.com/
```

What it shows:

- DNS resolution
- TCP connect
- TLS handshake
- negotiated protocol
- raw request headers
- raw response headers

Representative output:

```text
* Host www.example.com:443 was resolved.
*   Trying 93.184.216.34:443...
* Connected to www.example.com (93.184.216.34) port 443
* ALPN: curl offers h2,http/1.1
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
> GET / HTTP/2
> Host: www.example.com
> User-Agent: curl/8.7.1
> Accept: */*
< HTTP/2 200
< content-type: text/html; charset=UTF-8
< content-length: 1582
```

### 12.2 `curl -H` for custom headers

Command:

```bash
curl -i https://api.example.com/me \
  -H 'Authorization: Bearer demo-token' \
  -H 'Accept: application/json'
```

### 12.3 `curl -X POST -d` for sending data

Command:

```bash
curl -i https://api.example.com/users \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"John","email":"john@example.com"}'
```

Representative response:

```http
HTTP/1.1 201 Created
Location: /users/123
Content-Type: application/json

{"id":123,"name":"John","email":"john@example.com"}
```

### 12.4 `curl -o` to write response body to a file

Command:

```bash
curl -o homepage.html https://www.example.com/
```

Useful when:

- saving HTML for offline inspection
- downloading binary files
- avoiding binary output in the terminal

### 12.5 `curl --resolve` to test without real DNS changes

Command:

```bash
curl -i https://www.example.com/ \
  --resolve www.example.com:443:203.0.113.50
```

What it does:

- uses the given IP for that host and port
- still sends correct Host/SNI info

Great for:

- testing new server before DNS cutover
- validating TLS on a future target

### 12.6 `curl -w` for timing metrics

Command:

```bash
curl -o /dev/null -s -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n' https://www.example.com/
```

Representative output:

```text
dns=0.004 connect=0.023 tls=0.071 ttfb=0.132 total=0.181
```

### 12.7 `curl -k` to skip TLS verification

Command:

```bash
curl -k -i https://staging.internal.example/
```

Warning:

Use only for debugging.

It disables certificate verification.

Do not normalize insecure production habits with `-k`.

### 12.8 Curl cheat sheet

| Goal | Command pattern |
|---|---|
| verbose details | `curl -v URL` |
| show headers and body | `curl -i URL` |
| HEAD request | `curl -I URL` |
| custom header | `curl -H 'Header: value' URL` |
| custom method | `curl -X METHOD URL` |
| send JSON body | `curl -H 'Content-Type: application/json' -d '{}' URL` |
| save output | `curl -o file URL` |
| test DNS override | `curl --resolve host:443:IP URL` |
| measure timings | `curl -w '...' URL` |
| skip TLS verify | `curl -k URL` |

## 13. WebSocket — HTTP Upgrade Visual

WebSocket starts as HTTP.

### 📸 WebSocket Protocol
![WebSocket](https://upload.wikimedia.org/wikipedia/commons/1/10/Websocket_connection.png)
> *Source: Wikimedia Commons — WebSocket connection upgrade*

Then it upgrades.

After upgrade,

it becomes a persistent full-duplex protocol.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    
    C->>S: GET /chat HTTP/1.1<br/>Upgrade: websocket<br/>Connection: Upgrade
    S-->>C: 101 Switching Protocols<br/>Upgrade: websocket
    Note over C,S: Full-duplex communication
    C->>S: Message frame
    S-->>C: Message frame
    S-->>C: Server-initiated message
    C->>S: Close frame
    S-->>C: Close frame
```

### 13.1 Why WebSocket exists

Normal HTTP is request/response.

That is fine for:

- pages
- normal APIs
- downloads

But for:

- chat
- live dashboards
- multiplayer state
- market feeds
- collaborative editing

it is often better to keep a connection open.

### 13.2 WebSocket handshake example

Request:

```http
GET /chat HTTP/1.1
Host: chat.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://app.example.com
```

Response:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### 13.3 Common proxy requirements for WebSocket

- forward `Upgrade`
- forward `Connection: Upgrade`
- keep connection timeouts long enough
- disable buffering when appropriate

### 13.4 Common WebSocket debugging questions

- did the server return `101`?
- did the proxy pass upgrade headers?
- is TLS terminating correctly?
- are idle timeouts too short?
- does load balancer need sticky sessions?

---

## 14. DNS, TCP, TLS, and HTTP Together

HTTP does not live alone.

It depends on lower layers.

### 14.1 Layered view

```mermaid
graph TD
    A["Application content<br/>HTML JSON CSS JS"] --> B["HTTP"]
    B --> C["TLS for HTTPS"]
    C --> D["TCP or QUIC"]
    D --> E["IP networking"]
    E --> F["DNS helps find the host"]
```

### 14.2 DNS refresher

DNS maps names to addresses.

Common record types:

- `A`
- `AAAA`
- `CNAME`
- `MX`
- `TXT`

Common commands:

```bash
dig example.com A
dig example.com AAAA
dig +short www.example.com
host www.example.com
```

### 14.3 TCP refresher

TCP gives:

- ordered delivery
- retransmission
- congestion control
- stream semantics

HTTP/1.0,
HTTP/1.1,
and HTTP/2

usually ride on TCP.

### 14.4 TLS refresher

TLS gives:

- encryption
- integrity
- authentication of the server

Common debugging command:

```bash
openssl s_client -connect www.example.com:443 -servername www.example.com
```

What you inspect:

- certificate subject
- SAN names
- issuer
- expiration
- negotiated TLS version
- cipher suite

### 14.5 Why HTTPS is not “application security” by itself

HTTPS protects data in transit.

It does not automatically fix:

- XSS
- CSRF
- SQL injection
- broken access control
- insecure session management

### 14.6 Seeing the layers in `curl -v`

Representative output sections:

- DNS resolved
- TCP connected
- TLS handshake succeeded
- HTTP request sent
- HTTP response received

That is why `curl -v` is so powerful.

---

## 15. Browser Rendering After the Response

Getting a `200 OK` is not the end.

It is the start of rendering work.

### 15.1 Rendering flow

```mermaid
graph TD
    A["HTML response received"] --> B["Parse HTML"]
    B --> C["Build DOM"]
    B --> D["Discover CSS JS images fonts"]
    D --> E["Fetch subresources via HTTP"]
    E --> F["Parse CSS"]
    F --> G["Build CSSOM"]
    C --> H["Combine DOM + CSSOM"]
    G --> H["Combine DOM + CSSOM"]
    H --> I["Layout"]
    I --> J["Paint"]
    J --> K["Composite on screen"]
```

### 15.2 Important consequence

One HTML response often causes many more HTTP requests.

Examples:

- CSS file
- JS bundle
- logo image
- font file
- API request for data

### 15.3 Why this matters for HTTP

Because page performance depends on:

- number of requests
- cacheability of subresources
- compression
- protocol version
- CDN placement
- render-blocking CSS and JS

### 15.4 Render-blocking resources

A CSS file in the `<head>` often blocks rendering.

A synchronous script can block parsing.

That is why:

- CSS delivery matters
- JS bundling and deferral matter
- caching static assets matters

---

## 16. Proxies, CDNs, and Reverse Proxies

Most production HTTP traffic touches intermediaries.

### 16.1 Forward proxy vs reverse proxy

| Type | Sits in front of | Used by |
|---|---|---|
| forward proxy | clients | enterprises, filtering, egress control |
| reverse proxy | servers | websites, APIs, edge control |

### 16.2 Reverse proxy jobs

- TLS termination
- static file serving
- routing to backends
- compression
- caching
- rate limiting
- header rewriting
- observability

### 16.3 CDN jobs

- cache static assets near users
- absorb traffic spikes
- terminate TLS globally
- provide edge security features
- lower latency worldwide

### 16.4 Reverse proxy visual

```mermaid
graph LR
    A["Browser"] --> B["CDN or edge proxy"] --> C["Reverse proxy"] --> D["App service"] --> E["Database"]
```

### 16.5 Why `Host` matters

A reverse proxy may host many domains on one IP.

The `Host` header tells it which site the client wants.

Example:

```http
Host: api.example.com
```

### 16.6 Why forwarding headers matter

Without forwarded headers,

the app may think:

- client IP is the proxy IP
- request scheme is HTTP not HTTPS
- original host is lost

That breaks:

- rate limiting
- audit logging
- redirect generation
- absolute URL generation
- secure cookie logic

---

## 17. Hands-On Debugging Scenarios

This section focuses on practical reasoning.

### 17.1 Scenario: site does not load at all

Start with:

```bash
curl -v https://www.example.com/
```

Look for:

- DNS failure
- TCP timeout
- TLS error
- HTTP error

Interpretation guide:

- no DNS answer -> DNS issue
- connect timeout -> network or firewall issue
- cert verify failure -> TLS issue
- `502` -> proxy/upstream issue
- `200` but blank page in browser -> likely frontend/render issue

### 17.2 Scenario: API returns 401 unexpectedly

Checklist:

- send token?
- token expired?
- wrong signing key?
- wrong audience or issuer?
- clock skew?

Command:

```bash
curl -i https://api.example.com/me \
  -H 'Authorization: Bearer demo-token'
```

### 17.3 Scenario: browser shows CORS error

Checklist:

- did response include `Access-Control-Allow-Origin`?
- is `Origin` exactly allowed?
- did preflight succeed?
- are credentials involved?
- is proxy stripping headers?

Command:

```bash
curl -i https://api.other.com/data \
  -X OPTIONS \
  -H 'Origin: https://app.example.com' \
  -H 'Access-Control-Request-Method: PUT' \
  -H 'Access-Control-Request-Headers: Authorization, Content-Type'
```

### 17.4 Scenario: assets are slow

Checklist:

- using HTTP/2 or HTTP/3?
- assets compressed?
- long-lived cache headers?
- CDN in front?
- asset count too high?

Commands:

```bash
curl --http2 -I https://www.example.com/app.js
curl -I https://www.example.com/app.js
curl -o /dev/null -s -w 'total=%{time_total}\n' https://www.example.com/app.js
```

### 17.5 Scenario: users randomly get logged out

Checklist:

- `Set-Cookie` attributes stable?
- domain or path mismatch?
- `Secure` cookie over HTTP?
- session store expiring too fast?
- load balancer stickiness required?

Inspect:

```bash
curl -i https://app.example.com/login
```

### 17.6 Scenario: intermittent 502

Checklist:

- upstream healthy?
- wrong port configured?
- keep-alive mismatch?
- backend restarting?
- app returning invalid headers?

Useful steps:

- test backend directly
- inspect proxy error logs
- compare timeout settings

### 17.7 Scenario: intermittent 504

Checklist:

- slow database?
- slow external API?
- too much request fan-out?
- timeout mismatch between LB, proxy, and app?

Useful commands:

```bash
curl -o /dev/null -s -w 'ttfb=%{time_starttransfer} total=%{time_total}\n' https://app.example.com/report
```

### 17.8 Scenario: downloads cannot resume

Checklist:

- does server support `Range`?
- does response use `206 Partial Content`?
- is CDN preserving range requests?

Command:

```bash
curl -i https://cdn.example.com/bigfile.iso \
  -H 'Range: bytes=1000-1999'
```

### 17.9 Scenario: browser still sees old JS after deploy

Checklist:

- long max-age on unversioned asset?
- old service worker?
- CDN cache not purged?
- asset fingerprint not changed?

Best fix:

- use content-hashed filenames
- keep long cache only for versioned assets

---

## 18. Glossary

### 18.1 A

- **Accept**: request header showing preferred response formats.
- **Access-Control-Allow-Origin**: response header controlling who may read a cross-origin response in browsers.
- **ALPN**: TLS negotiation mechanism for selecting protocols like HTTP/2.
- **API**: application programming interface.
- **ASCII**: plain-text character encoding family used for human-readable examples.
- **Authority**: host and optional port of a URL.

### 18.2 B

- **Backend**: service behind a proxy or frontend.
- **Bearer token**: token presented as proof of authorization.
- **Body**: payload part of request or response.
- **Browser cache**: client-side local storage of responses.
- **Byte range**: subset of a resource requested with `Range`.

### 18.3 C

- **Cache-Control**: header controlling caching behavior.
- **CDN**: content delivery network.
- **Chunked transfer**: HTTP/1.1 body framing when full size is not known ahead of time.
- **Client**: requester of a resource.
- **Compression**: reducing payload size with gzip or Brotli.
- **Connection reuse**: using the same connection for multiple requests.
- **Cookie**: small browser-stored value sent back to a server.
- **CORS**: cross-origin resource sharing.
- **CSP**: Content Security Policy.

### 18.4 D

- **DELETE**: HTTP method used to remove a resource.
- **DNS**: Domain Name System.
- **DOM**: Document Object Model.

### 18.5 E

- **ETag**: validator string representing a resource version.
- **Edge**: network location close to users, often CDN or proxy.
- **Expectation**: client behavior hinted through headers like `Expect: 100-continue`.

### 18.6 F

- **Forward proxy**: intermediary used by clients.
- **Fragment**: URL portion after `#`, handled client-side.
- **Full-duplex**: both sides can send independently.

### 18.7 G

- **GET**: HTTP method used to retrieve a representation.
- **Gateway**: intermediary server between client and origin.
- **gzip**: common compression format.

### 18.8 H

- **HEAD**: HTTP method returning headers without body.
- **Header**: metadata line in request or response.
- **HSTS**: Strict-Transport-Security policy.
- **HPACK**: HTTP/2 header compression.
- **HTTP**: Hypertext Transfer Protocol.
- **HTTPS**: HTTP over TLS.

### 18.9 I

- **Idempotent**: repeating the same request gives the same final state.
- **If-Modified-Since**: conditional request header using timestamps.
- **If-None-Match**: conditional request header using `ETag`.
- **Immutable**: cache directive saying a versioned resource will not change during freshness lifetime.

### 18.10 J

- **JSON**: JavaScript Object Notation.
- **JWT**: JSON Web Token.

### 18.11 K

- **Keep-Alive**: connection reuse behavior.

### 18.12 L

- **Latency**: time delay before or during transfer.
- **Load balancer**: component distributing requests to backends.
- **Location**: response header used for redirects and resource creation.

### 18.13 M

- **Method**: action requested by the client.
- **MIME type**: content type descriptor.
- **Multiplexing**: multiple streams on one connection.

### 18.14 N

- **No-cache**: may store but must revalidate before reuse.
- **No-store**: do not store at all.
- **Origin**: scheme + host + port.

### 18.15 O

- **OPTIONS**: method used for capability discovery and preflight.
- **Origin server**: the server that owns the resource.

### 18.16 P

- **PATCH**: partial update method.
- **Path**: resource location in a URL.
- **Persistent connection**: connection kept open for reuse.
- **POST**: method for create or submit semantics.
- **Preflight**: browser `OPTIONS` request before certain cross-origin requests.
- **Proxy**: intermediary between client and server.
- **PUT**: method for replacing a resource.

### 18.17 Q

- **QPACK**: HTTP/3 header compression scheme.
- **QUIC**: transport protocol used by HTTP/3.
- **Query string**: URL parameters after `?`.

### 18.18 R

- **Rate limit**: policy restricting request frequency.
- **Reason phrase**: human-readable text after status code.
- **Redirect**: server instruction to request a different URL.
- **Representation**: returned form of a resource such as HTML or JSON.
- **REST**: architectural style using resources and HTTP semantics.
- **Retry-After**: response header suggesting how long to wait before retrying.

### 18.19 S

- **Safe method**: method that should not change server state.
- **SameSite**: cookie attribute affecting cross-site sending.
- **SNI**: Server Name Indication in TLS.
- **Status code**: numeric outcome of an HTTP request.
- **Stream**: logical independent channel in HTTP/2 or HTTP/3.

### 18.20 T

- **TCP**: reliable transport protocol.
- **TLS**: Transport Layer Security.
- **TTFB**: time to first byte.

### 18.21 U

- **URI**: Uniform Resource Identifier.
- **URL**: Uniform Resource Locator.
- **Upgrade**: header used to switch protocols.
- **User-Agent**: request header identifying client software.

### 18.22 V

- **Validator**: value used to check whether cached content is still current.
- **Vary**: response header telling caches which request headers affect representation.

### 18.23 W

- **WebSocket**: upgraded protocol for persistent two-way communication.
- **WWW-Authenticate**: header telling client how to authenticate.

---

## 19. Rapid Review Checklists

### 19.1 When a page is down

- Can DNS resolve the host?
- Can TCP connect to the port?
- Does TLS validate?
- What status code comes back?
- Is the response from the edge or the app?
- Are subresources also failing?

### 19.2 When an API call fails

- Right method?
- Right path?
- Right query string?
- Right auth header?
- Right `Content-Type`?
- Valid JSON body?
- Reasonable timeout?

### 19.3 When caching looks wrong

- `Cache-Control` present?
- `ETag` present?
- `Vary` correct?
- resource versioned?
- CDN purged?
- browser hard refresh tested?

### 19.4 When cookies behave strangely

- correct domain?
- correct path?
- `Secure` needed?
- `HttpOnly` expected?
- `SameSite` suitable?
- HTTPS consistent?

### 19.5 When CORS fails

- exact origin allowed?
- preflight answered?
- headers allowed?
- methods allowed?
- credentials and wildcard mixed accidentally?

### 19.6 When redirects feel wrong

- loop present?
- wrong host in `Location`?
- HTTP to HTTPS configured correctly?
- method-preserving redirect needed?
- permanent redirect cached in browser?

### 19.7 When load balancing behaves badly

- any healthy backends?
- stickiness required?
- session stored centrally?
- timeout mismatch?
- forwarded headers correct?

---

## 20. Curl Practice Lab

### 20.1 Inspect only headers

```bash
curl -I https://www.example.com/
```

Look for:

- status code
- server
- content type
- cache-control
- location

### 20.2 Inspect full response

```bash
curl -i https://www.example.com/
```

Look for:

- headers first
- body second

### 20.3 Inspect verbose connection details

```bash
curl -v https://www.example.com/
```

Look for:

- resolved IP
- ALPN
- TLS version
- request line
- response headers

### 20.4 Force HTTP/1.1

```bash
curl --http1.1 -I https://www.example.com/
```

### 20.5 Force HTTP/2

```bash
curl --http2 -I https://www.example.com/
```

### 20.6 Send custom host testing traffic

```bash
curl -i https://www.example.com/ \
  --resolve www.example.com:443:203.0.113.50
```

### 20.7 Send JSON body

```bash
curl -i https://api.example.com/orders \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"sku":"KB-100","qty":1}'
```

### 20.8 Simulate cache revalidation

```bash
curl -i https://cdn.example.com/app.js \
  -H 'If-None-Match: "appjs-v17"'
```

### 20.9 Simulate CORS preflight

```bash
curl -i https://api.other.com/data \
  -X OPTIONS \
  -H 'Origin: https://app.example.com' \
  -H 'Access-Control-Request-Method: PATCH' \
  -H 'Access-Control-Request-Headers: Authorization, Content-Type'
```

### 20.10 Measure timings

```bash
curl -o /dev/null -s -w 'dns=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nttfb=%{time_starttransfer}\ntotal=%{time_total}\n' https://www.example.com/
```

### 20.11 Download a file

```bash
curl -o image.png https://www.example.com/logo.png
```

### 20.12 Test a self-signed staging cert

```bash
curl -k -i https://staging.internal.example/
```

### 20.13 Send bearer token

```bash
curl -i https://api.example.com/me \
  -H 'Authorization: Bearer demo-token'
```

### 20.14 Send basic auth

```bash
curl -i https://api.example.com/admin \
  -u admin:secret
```

### 20.15 Ask for compressed content

```bash
curl -i https://www.example.com/app.js \
  -H 'Accept-Encoding: gzip, br'
```

---

## 21. Quick Reference Tables

### 21.1 Frequently used request headers

| Header | Typical example | Purpose |
|---|---|---|
| `Host` | `Host: api.example.com` | virtual host routing |
| `Accept` | `Accept: application/json` | desired response format |
| `Authorization` | `Authorization: Bearer ...` | auth credentials |
| `Content-Type` | `Content-Type: application/json` | request body format |
| `Content-Length` | `Content-Length: 123` | body size |
| `Cookie` | `Cookie: session=abc` | browser state |
| `Origin` | `Origin: https://app.example.com` | browser origin info |
| `If-None-Match` | `If-None-Match: "v1"` | cache revalidation |
| `If-Modified-Since` | date value | timestamp revalidation |
| `Range` | `Range: bytes=0-999` | partial request |

### 21.2 Frequently used response headers

| Header | Typical example | Purpose |
|---|---|---|
| `Content-Type` | `application/json` | response format |
| `Content-Length` | `1234` | size |
| `Cache-Control` | `public, max-age=3600` | caching policy |
| `ETag` | `"asset-v17"` | validator |
| `Last-Modified` | date value | timestamp validator |
| `Set-Cookie` | `session=xyz; HttpOnly; Secure` | browser cookie creation |
| `Location` | `/login` | redirect or creation target |
| `Strict-Transport-Security` | long max-age | HTTPS enforcement |
| `Content-Security-Policy` | `default-src 'self'` | browser security policy |
| `Access-Control-Allow-Origin` | `https://app.example.com` | CORS access |

### 21.3 Common status codes at a glance

| Code | Meaning | Usual next step |
|---|---|---|
| `200` | success | use body |
| `201` | created | note `Location` |
| `204` | success, no body | do not parse body |
| `301` | permanent redirect | follow new URL |
| `304` | not modified | use cached copy |
| `400` | bad request | fix request syntax/data |
| `401` | unauthenticated | send valid credentials |
| `403` | forbidden | check permissions/policy |
| `404` | not found | check path/id |
| `409` | conflict | resolve state/version issue |
| `415` | wrong media type | fix `Content-Type` |
| `422` | validation error | fix field values |
| `429` | too many requests | back off |
| `500` | server bug | inspect server logs |
| `502` | bad upstream response | inspect proxy/backend |
| `503` | unavailable | check health/load |
| `504` | upstream timeout | profile slow dependency |

---

## 22. End-to-End Worked Example

Suppose a user opens:

```text
https://shop.example.com/products/42
```

### 22.1 URL parse

- scheme: `https`
- host: `shop.example.com`
- path: `/products/42`

### 22.2 DNS

Browser resolves host.

### 22.3 TCP and TLS

Browser connects to `443`.

TLS validates certificate.

### 22.4 Initial HTTP request

```http
GET /products/42 HTTP/1.1
Host: shop.example.com
Accept: text/html
Accept-Encoding: gzip, br
User-Agent: Mozilla/5.0
```

### 22.5 Server handling

- reverse proxy receives request
- route goes to product page handler
- app checks cache
- app queries product database
- template engine renders HTML

### 22.6 Response

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Cache-Control: no-cache
Content-Encoding: br

<!DOCTYPE html>
<html>...</html>
```

### 22.7 Browser discovers subresources

- `/assets/app.4f92ac7.js`
- `/assets/styles.2b8d1aa.css`
- `/images/products/42.webp`
- `/api/recommendations?product=42`

### 22.8 Asset caching example

Versioned JS response:

```http
HTTP/1.1 200 OK
Cache-Control: public, max-age=31536000, immutable
ETag: "app-4f92ac7"
Content-Type: application/javascript
```

### 22.9 API request example

```http
GET /api/recommendations?product=42 HTTP/1.1
Accept: application/json
```

### 22.10 API response example

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: private, no-cache

{"items":[{"id":87,"name":"Mouse"},{"id":91,"name":"Monitor"}]}
```

### 22.11 Final visible outcome

- HTML rendered
- CSS applied
- JS enhances interactions
- product image loaded
- recommendations appear

That is a realistic HTTP-driven page load.

---

## 23. Common Misconceptions

### 23.1 “HTTPS means the site is secure”

Not fully.

HTTPS protects transport.

Application bugs can still exist.

### 23.2 “GET can never change anything”

By convention it should not.

Poorly designed apps sometimes violate this.

That is bad design.

### 23.3 “401 means permission denied”

Not quite.

`401` means auth is missing or invalid.

`403` means auth may be present,

but access is denied.

### 23.4 “CORS protects my API from all misuse”

No.

CORS is enforced by browsers.

Non-browser clients can still call the API.

Use real authentication and authorization.

### 23.5 “304 means an error”

No.

It is a cache success signal.

### 23.6 “HTTP/3 is always faster in every situation”

Not always.

It often helps,

especially on lossy networks.

But application design,

payload size,

caching,

and backend latency still dominate many cases.

---

## 24. Practical Design Advice

### 24.1 For static assets

- fingerprint filenames
- compress with Brotli and gzip
- set long max-age
- use CDN where possible

### 24.2 For HTML pages

- keep cache policy deliberate
- avoid leaking private content into shared caches
- use CSP and HSTS

### 24.3 For JSON APIs

- use correct status codes
- document request and response schemas
- return machine-readable errors
- include request IDs for tracing

### 24.4 For auth

- use HTTPS only
- secure cookies with `HttpOnly` and `Secure`
- validate tokens carefully
- rotate secrets

### 24.5 For operations

- log status code and latency
- log upstream info at proxies
- expose health checks
- align timeouts across components

---

## 25. Final Summary

HTTP is simple at the surface.

But real web delivery includes:

- URL parsing
- DNS
- TCP or QUIC
- TLS
- HTTP request semantics
- headers
- caches
- proxies
- load balancers
- browsers
- rendering pipelines

If you remember only a few things,

remember these.

### 25.1 Core truths

- A browser does much more than “send GET”.
- `Content-Type` and `Accept` are different.
- `401` and `403` are different.
- `304` is a cache optimization, not a failure.
- HTTP/2 and HTTP/3 improve asset loading efficiency.
- CORS is a browser policy, not an API auth mechanism.
- Cache headers can make sites dramatically faster.
- Reverse proxies shape a huge amount of real-world HTTP behavior.
- `curl -v` is your friend.
- Always debug layer by layer.

### 25.2 Minimal troubleshooting sequence

1. Resolve DNS.
2. Test TCP reachability.
3. Verify TLS.
4. Inspect request and response headers.
5. Identify the exact status code.
6. Determine whether edge, proxy, app, or dependency failed.
7. Check cache, CORS, cookies, or redirects if behavior seems inconsistent.

### 25.3 Final mental model

HTTP is the language.

TCP or QUIC is the transport.

TLS is the secure wrapper.

Headers are the control plane.

Bodies carry the content.

Browsers, proxies, and caches all interpret the rules.

Once you can visualize the full path,

you can debug most web problems far faster.

---

## 26. Ultra-Short Review Prompts

- What part of a URL is not sent to the server?
- Why is `Host` required in HTTP/1.1?
- When should you use `201 Created`?
- What is the difference between `401` and `403`?
- Why does `304` reduce bandwidth?
- When is `PATCH` more appropriate than `PUT`?
- What does `Content-Encoding: br` mean?
- Why does `Vary` matter to caches?
- Why can CORS fail even if the backend returned `200`?
- Why do versioned assets often use `immutable`?
- What makes HTTP/2 faster than HTTP/1.1 on asset-heavy pages?
- What does QUIC improve for HTTP/3?
- Why is `curl --resolve` useful before DNS cutover?
- Why can a `204` response break a careless JSON parser?
- What headers must a WebSocket proxy pass?
- Why can big cookies hurt performance?
- Why should auth responses often use `Cache-Control: no-store`?
- Why can `502` and `504` point to very different root causes?
- Why does HSTS matter after a site first loads on HTTPS?
- Why should debugging begin with layers, not guesses?

---

## 27. Extended One-Line Reminders

- DNS failure is not an HTTP failure.
- A TLS failure happens before secure HTTP begins.
- The URL fragment stays in the browser.
- `GET` should be safe.
- `POST` is often non-idempotent.
- `PUT` usually replaces.
- `PATCH` usually modifies part of a resource.
- `DELETE` often returns `204`.
- `Content-Type` describes what the body is.
- `Accept` describes what the client wants back.
- `Content-Encoding` describes transfer encoding like gzip or Brotli.
- `Cache-Control` controls freshness and storage.
- `ETag` helps revalidation.
- `If-None-Match` asks “did this change?”.
- `Set-Cookie` changes browser state.
- `Cookie` sends browser state back.
- `Origin` matters for browser CORS checks.
- `Location` powers redirects and created-resource references.
- `WWW-Authenticate` tells a client how to authenticate.
- `Retry-After` helps clients back off.
- `301` is permanent.
- `302` is temporary but historically ambiguous.
- `307` preserves method temporarily.
- `308` preserves method permanently.
- `400` means the request is bad.
- `401` means auth is missing or invalid.
- `403` means access is refused.
- `404` means not found.
- `409` means conflict.
- `415` means wrong media type.
- `422` means validation failed.
- `429` means slow down.
- `500` means the app failed.
- `502` means the proxy got a bad upstream response.
- `503` means the service is unavailable.
- `504` means upstream took too long.
- HTTP/1.0 opened many connections.
- HTTP/1.1 reused connections.
- HTTP/2 multiplexes streams.
- HTTP/3 uses QUIC.
- CORS is enforced by browsers.
- CDNs lower latency and offload origins.
- Reverse proxies centralize edge behavior.
- Load balancers spread traffic.
- Caching improves speed when done carefully.
- Bad caching leaks bugs and stale content.
- Security headers reduce browser attack surface.
- `curl -v` exposes the wire-level story.
- `curl -w` exposes timing.
- `curl -k` is only for debugging.
- `curl --resolve` is ideal for pre-cutover testing.
- WebSocket starts as HTTP and upgrades.
- Real page loads generate many HTTP requests.
- Backend latency still dominates user experience.
- Protocol upgrades do not replace good design.
- Good headers make behavior explicit.
- Good status codes make debugging faster.
- Good caching makes websites feel instant.
- Good observability makes incidents shorter.
- Good mental models make HTTP feel much simpler.
