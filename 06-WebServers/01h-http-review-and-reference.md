# HTTP Review and Reference

← Back to [01-http-fundamentals.md](./01-http-fundamentals.md)

Glossary, recap material, short prompts, and compact reminders for interview and incident review.

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
