# 4. Dockerfile

## 4.1 What Is a Dockerfile?

A Dockerfile is a text file containing instructions to build a container image.
Each instruction creates metadata and often a new layer.

## 4.2 Simple Dockerfile Example

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

## 4.3 Dockerfile Instruction Overview

| Instruction | Purpose |
|---|---|
| `FROM` | Select base image |
| `RUN` | Execute build-time commands |
| `COPY` | Copy files from build context |
| `ADD` | Copy plus archive/URL extras |
| `CMD` | Default runtime command/args |
| `ENTRYPOINT` | Main executable |
| `ENV` | Set environment variables |
| `ARG` | Build-time variable |
| `EXPOSE` | Document listening ports |
| `WORKDIR` | Set working directory |
| `USER` | Set default user |
| `HEALTHCHECK` | Define health command |

## 4.4 `FROM`

`FROM` sets the base image.

Examples:

```dockerfile
FROM ubuntu:24.04
```

```dockerfile
FROM node:22-alpine
```

Best practices:

- Pin versions
- Prefer trusted sources
- Prefer minimal images when appropriate
- Consider digest pinning for critical builds

## 4.5 `RUN`

`RUN` executes commands during image build.

Example:

```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

Guidelines:

- Combine related commands to reduce layers and keep cleanup in same layer
- Remove package manager caches when useful
- Use shell options for safer builds when appropriate

## 4.6 `COPY`

`COPY` copies files from the build context into the image.

Example:

```dockerfile
COPY . /app
```

Prefer `COPY` over `ADD` unless you need `ADD` features.

## 4.7 `ADD`

`ADD` can:

- Copy local files
- Auto-extract certain local tar archives
- Fetch remote URLs in some implementations/workflows

Because `ADD` has extra behavior, prefer `COPY` for predictability.

## 4.8 `CMD`

`CMD` provides default runtime command or arguments.

Example:

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Only one `CMD` is effective; the last one wins.

## 4.9 `ENTRYPOINT`

`ENTRYPOINT` defines the main executable.

Example:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Use `CMD` with `ENTRYPOINT` to provide default arguments.

## 4.10 `CMD` vs `ENTRYPOINT`

| Pattern | Behavior |
|---|---|
| `CMD ["python", "app.py"]` | Default command, easily overridden |
| `ENTRYPOINT ["python", "app.py"]` | Fixed executable |
| `ENTRYPOINT` + `CMD` | Fixed executable with default arguments |

Example:

```dockerfile
ENTRYPOINT ["gunicorn"]
CMD ["-b", "0.0.0.0:8000", "app:wsgi"]
```

## 4.11 `ENV`

`ENV` sets environment variables in the image/runtime environment.

```dockerfile
ENV APP_ENV=production
ENV PYTHONUNBUFFERED=1
```

Use for:

- Runtime defaults
- Tool behavior

Avoid baking sensitive values into images.

## 4.12 `ARG`

`ARG` defines build-time variables.

```dockerfile
ARG APP_VERSION=dev
RUN echo "$APP_VERSION"
```

Important:

- `ARG` is available only during build unless promoted to `ENV`
- Build args may still appear in metadata/history
- Do not pass secrets with plain `ARG`

## 4.13 `EXPOSE`

`EXPOSE` documents intended listening ports.
It does **not** publish ports by itself.

```dockerfile
EXPOSE 8080
```

## 4.14 `WORKDIR`

`WORKDIR` sets the working directory for following instructions.

```dockerfile
WORKDIR /app
```

Prefer this instead of repeated `cd` in `RUN` commands.

## 4.15 `USER`

`USER` sets the default user for later instructions and runtime.

```dockerfile
RUN useradd -r -u 10001 appuser
USER appuser
```

Running as non-root is one of the simplest and most valuable image hardening steps.

## 4.16 `HEALTHCHECK`

`HEALTHCHECK` defines how the runtime checks container health.

Example:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

Cautions:

- Keep checks lightweight
- Avoid false positives
- Do not hide real failures with overly permissive checks

## 4.17 Multi-Stage Builds Overview

Multi-stage builds let you separate build and runtime environments.
This reduces final image size and attack surface.

## 4.18 Multi-Stage Build Example: Go

```dockerfile
FROM golang:1.23 AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /out/app ./cmd/app

FROM gcr.io/distroless/static-debian12
COPY --from=builder /out/app /app
USER 65532:65532
ENTRYPOINT ["/app"]
```

## 4.19 Multi-Stage Build Example: Node.js

```dockerfile
FROM node:22 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22 AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

## 4.20 Why Multi-Stage Builds Help

- Keep compilers out of runtime image
- Reduce image size
- Reduce vulnerability surface
- Separate concerns cleanly

## 4.21 Docker Layer Caching

Docker reuses previous layers if the instruction and inputs have not changed.

Example optimization:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

If only application code changes, the dependency install layer can remain cached.

## 4.22 Bad Layer Ordering Example

```dockerfile
COPY . .
RUN npm ci
```

This invalidates the dependency layer every time any source file changes.

## 4.23 Good Layer Ordering Example

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

## 4.24 Minimal Base Images

Popular choices:

| Base Type | Pros | Cons |
|---|---|---|
| Alpine | Small | musl differences may cause surprises |
| Debian slim | Good compatibility | Larger than Alpine |
| Distroless | Minimal runtime attack surface | Harder debugging |
| Scratch | Smallest possible | Only for static binaries |

## 4.25 Choosing a Base Image

Choose based on:

- Runtime compatibility
- Security patch cadence
- Debuggability needs
- Team familiarity
- Library requirements

## 4.26 `.dockerignore`

`.dockerignore` reduces build context size and prevents unnecessary files from being sent into builds.

Example:

```gitignore
.git
node_modules
venv
__pycache__
.env
coverage
Dockerfile*
*.log
```

## 4.27 Why `.dockerignore` Matters

It helps:

- Speed up builds
- Avoid leaking secrets or local artifacts
- Improve cache behavior
- Reduce disk/network usage

## 4.28 Shell Form vs Exec Form

Shell form:

```dockerfile
CMD python app.py
```

Exec form:

```dockerfile
CMD ["python", "app.py"]
```

Exec form is preferred because:

- Better signal handling
- No implicit shell wrapping
- More predictable argument parsing

## 4.29 Apt Best Practices

For Debian/Ubuntu-based images:

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
```

Guidelines:

- Combine update/install/cleanup in one layer
- Use `--no-install-recommends`
- Clean package lists in the same layer

## 4.30 Package Manager Cache Considerations

For Python:

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

For npm:

- Consider `npm ci`
- Clean caches if necessary
- Keep lockfile consistent

## 4.31 Example Production Dockerfile: Python API

```dockerfile
FROM python:3.12-slim AS base
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /app

RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN useradd -r -u 10001 appuser
COPY . .
USER appuser
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --retries=3 CMD curl -f http://localhost:8000/health || exit 1
CMD ["gunicorn", "-b", "0.0.0.0:8000", "app:app"]
```

## 4.32 Example Production Dockerfile: Java

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /src
COPY pom.xml .
RUN mvn -q -DskipTests dependency:go-offline
COPY . .
RUN mvn -q -DskipTests package

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /src/target/app.jar app.jar
RUN useradd -r -u 10001 appuser
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 4.33 Example Production Dockerfile: Static Nginx Site

```dockerfile
FROM node:22 AS build
WORKDIR /src
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.27-alpine
COPY --from=build /src/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 4.34 Build Arguments Example

```dockerfile
ARG VERSION=dev
LABEL org.opencontainers.image.version=$VERSION
```

Build with:

```bash
docker build --build-arg VERSION=1.0.0 -t myapp:1.0.0 .
```

## 4.35 Labels

OCI labels help add metadata.

Example:

```dockerfile
LABEL org.opencontainers.image.title="myapp"
LABEL org.opencontainers.image.source="https://github.com/example/myapp"
LABEL org.opencontainers.image.description="Example service"
```

## 4.36 Stop Signal

You can define a stop signal:

```dockerfile
STOPSIGNAL SIGTERM
```

This can improve graceful shutdown behavior for some workloads.

## 4.37 Build Secrets with BuildKit

Use BuildKit secret mounts instead of baking secrets into layers.

Example command:

```bash
DOCKER_BUILDKIT=1 docker build --secret id=npmrc,src=$HOME/.npmrc -t myapp:dev .
```

Dockerfile example:

```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci
```

## 4.38 SSH Forwarding in Builds

For private Git dependencies:

```bash
DOCKER_BUILDKIT=1 docker build --ssh default -t myapp:dev .
```

Dockerfile example:

```dockerfile
RUN --mount=type=ssh git clone git@github.com:org/private-repo.git
```

## 4.39 Common Dockerfile Anti-Patterns

- Using `latest`
- Installing unnecessary packages
- Running as root by default
- Copying the entire repo too early
- Storing secrets in image layers
- Using shell form when exec form is better
- Huge monolithic Dockerfiles with no stage separation

## 4.40 Dockerfile Best Practices Checklist

- Pin base image versions
- Use multi-stage builds
- Order layers for caching
- Minimize installed packages
- Use non-root users
- Keep runtime image small
- Add only necessary files
- Use `.dockerignore`
- Document ports with `EXPOSE`
- Use health checks judiciously

## 4.41 Summary

A well-designed Dockerfile produces smaller, safer, faster, and more reproducible images.
Dockerfile quality directly impacts build speed, security posture, and operational reliability.

---

## Appendix A.4 Dockerfile Quick Reference

- Prefer exec form
- Use multi-stage builds
- Order layers for caching
- Use `.dockerignore`
- Run as non-root

---

## B.4 Dockerfile Q&A

### Q61. What does `FROM` do?
A61. It sets the base image.

### Q62. What does `RUN` do?
A62. It executes build-time commands.

### Q63. What does `COPY` do?
A63. It copies files from build context into the image.

### Q64. Why prefer `COPY` over `ADD`?
A64. `COPY` is simpler and more predictable.

### Q65. What does `CMD` define?
A65. The default runtime command or arguments.

### Q66. What does `ENTRYPOINT` define?
A66. The main executable for the container.

### Q67. What does `ENV` do?
A67. Sets environment variables in the image/runtime environment.

### Q68. What does `ARG` do?
A68. Defines build-time variables.

### Q69. Does `EXPOSE` publish a port?
A69. No; it only documents intended ports.

### Q70. What does `WORKDIR` do?
A70. Sets the working directory for subsequent instructions.

### Q71. Why use `USER`?
A71. To avoid running as root by default.

### Q72. What does `HEALTHCHECK` provide?
A72. A command to assess container health.

### Q73. Why use multi-stage builds?
A73. To keep build tools out of the final runtime image.

### Q74. Why does layer order matter?
A74. It affects cache reuse and build speed.

### Q75. What is `.dockerignore` for?
A75. Excluding files from the build context.

### Q76. Why is exec form preferred for `CMD` and `ENTRYPOINT`?
A76. Better signal handling and predictable argument parsing.

### Q77. Why should cleanup happen in the same `RUN` layer?
A77. To avoid leaving unnecessary data in earlier layers.

### Q78. Why is `npm ci` often preferred in images?
A78. It is more reproducible for lockfile-based installs.

### Q79. Why avoid secrets in `ARG`?
A79. They can leak via build history and metadata.

### Q80. What is a distroless image?
A80. A minimal runtime image without package managers and shells.
