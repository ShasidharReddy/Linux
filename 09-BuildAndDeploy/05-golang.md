# Go Applications

[Back to guide index](README.md)

## 5.1 Why Go Deployments Are Straightforward

Go compiles to native binaries.

This greatly simplifies deployment.

Benefits:

- Single binary distribution in many cases
- Simple process model
- Fast startup
- Strong cross-compilation support

## 5.2 Installing Go on Linux

Install from package manager or official tarball.

Package manager example:

```bash
# apt install -y golang-go
```

Official tarball pattern:

```bash
# tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

Set PATH:

```bash
export PATH=/usr/local/go/bin:$PATH
```

Verify:

```bash
$ go version
```

## 5.3 Workspace Basics

Modern Go uses modules.

Initialize a module:

```bash
$ go mod init example.com/myapp
```

Download dependencies:

```bash
$ go mod tidy
```

Important files:

- `go.mod`
- `go.sum`

## 5.4 Building a Go Binary

Simple build:

```bash
$ go build -o myapp ./cmd/myapp
```

Run:

```bash
$ ./myapp
```

## 5.5 Static Binaries

Many Go apps can be built statically.

Example:

```bash
$ CGO_ENABLED=0 go build -ldflags="-s -w" -o myapp ./cmd/myapp
```

Notes:

- `CGO_ENABLED=0` disables CGO
- `-s -w` reduces binary size by stripping symbol/debug information

Use static builds for:

- Simple host deployments
- Scratch or distroless containers
- Minimal runtime dependencies

## 5.6 Embedding Build Metadata

Example:

```bash
$ go build -ldflags="-X main.version=1.0.0 -X main.commit=$(git rev-parse --short HEAD)" -o myapp ./cmd/myapp
```

This is useful for:

- `/version` endpoints
- Incident diagnostics
- Deployment verification

## 5.7 Cross-Compilation

Go makes cross-compilation easy.

Examples:

```bash
$ GOOS=linux GOARCH=amd64 go build -o myapp-linux-amd64 ./cmd/myapp
$ GOOS=linux GOARCH=arm64 go build -o myapp-linux-arm64 ./cmd/myapp
$ GOOS=darwin GOARCH=arm64 go build -o myapp-darwin-arm64 ./cmd/myapp
```

Common pairs:

| GOOS | GOARCH | Target |
|---|---|---|
| `linux` | `amd64` | x86_64 Linux |
| `linux` | `arm64` | ARM64 Linux |
| `darwin` | `arm64` | Apple Silicon |
| `windows` | `amd64` | Windows x86_64 |

## 5.8 Go Test and Build Workflow

```bash
$ go fmt ./...
$ go test ./...
$ go build ./...
```

For CI:

```bash
$ go test -race ./...
$ go build -trimpath -o myapp ./cmd/myapp
```

## 5.9 Deploying a Go Binary with systemd

Example service:

```ini
[Unit]
Description=Go API Service
After=network.target

[Service]
Type=simple
User=goapp
Group=goapp
WorkingDirectory=/opt/goapp/current
EnvironmentFile=/etc/goapp/goapp.env
ExecStart=/opt/goapp/current/myapp
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

## 5.10 Nginx Reverse Proxy for Go HTTP Services

```nginx
server {
    listen 80;
    server_name go.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 5.11 Minimal Docker Images for Go

Multi-stage Dockerfile example:

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /out/myapp ./cmd/myapp

FROM gcr.io/distroless/static-debian12
COPY --from=build /out/myapp /myapp
USER nonroot:nonroot
ENTRYPOINT ["/myapp"]
```

Benefits:

- Small final image
- Reduced attack surface
- Fast startup

## 5.12 Release Layout for Go Apps

```text
/opt/goapp/
├── current -> /opt/goapp/releases/2025-01-15_120000/
├── releases/
└── shared/
    ├── logs/
    └── config/
```

## 5.13 Common Go Deployment Mistakes

- Forgetting CGO dependencies when moving binaries between hosts
- Not embedding version metadata
- Running without resource limits or visibility
- Binding publicly when a reverse proxy should front the service
- Shipping debug builds unintentionally

---
