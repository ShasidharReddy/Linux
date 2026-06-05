# HTTP/2, HTTP/3, and Request Lifecycle Context

← Back to [01-http-fundamentals.md](./01-http-fundamentals.md)

How a request reaches the server and how newer protocol versions improve transport behavior.

---

## 1. What Happens When You Type a URL — Complete Visual

A browser does much more than “send a GET request”.

### 📸 HTTP Request-Response Cycle
```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: HTTP Request (GET /index.html)
    S-->>C: HTTP Response (200 OK + HTML)
```
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

## 6. HTTP/1.0 vs HTTP/1.1 vs HTTP/2 vs HTTP/3 — Visual Comparison

Each new version tried to reduce latency and improve efficiency.

### 📸 HTTP/1.1 vs HTTP/2 Multiplexing
```mermaid
graph LR
    subgraph "HTTP/1.1 — Sequential"
        A1[Request 1] --> A2[Response 1]
        A2 --> A3[Request 2]
        A3 --> A4[Response 2]
    end
    subgraph "HTTP/2 — Multiplexed"
        B1[Stream 1 + Stream 2 + Stream 3] --> B2[All responses interleaved]
    end
```
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
