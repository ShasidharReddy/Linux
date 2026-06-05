# HTTP Status Codes Reference II

← Back to [01-http-fundamentals.md](./01-http-fundamentals.md)

Reference guide for common error responses, gateway failures, and fast decision mapping.

---

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
