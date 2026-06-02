# 7. Docker Compose

## 7.1 What Is Docker Compose?

Docker Compose lets you define and run multi-container applications using a YAML file.
It is ideal for:

- Local development
- Integration testing
- Small single-host deployments
- Demonstrations and reproducible environments

## 7.2 Compose File Purpose

A Compose file can define:

- Services
- Networks
- Volumes
- Environment variables
- Health checks
- Dependency relationships
- Profiles

## 7.3 Basic Compose Commands

```bash
docker compose up
```

```bash
docker compose up -d
```

```bash
docker compose down
```

```bash
docker compose ps
```

```bash
docker compose logs -f
```

## 7.4 Minimal Compose File Example

```yaml
services:
  web:
    image: nginx:stable
    ports:
      - "8080:80"
```

## 7.5 Compose File Structure Overview

| Key | Purpose |
|---|---|
| `services` | Containers that make up the app |
| `networks` | Custom network definitions |
| `volumes` | Persistent storage definitions |
| `configs` | Externalized config data in some environments |
| `secrets` | Managed secrets in supported contexts |

## 7.6 Service Example with Build

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
```

## 7.7 Environment Variables in Compose

```yaml
services:
  api:
    image: myapi:latest
    environment:
      APP_ENV: production
      LOG_LEVEL: info
```

Or via `.env` file and substitution:

```yaml
services:
  api:
    image: myapi:${APP_TAG}
```

## 7.8 `depends_on`

`depends_on` controls startup ordering, not full application readiness by itself.

```yaml
services:
  api:
    depends_on:
      - db
```

Modern Compose also supports health-based dependency conditions in some setups.

## 7.9 Health Checks in Compose

```yaml
services:
  api:
    image: myapi:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
```

## 7.10 Networks in Compose

```yaml
services:
  api:
    image: myapi:latest
    networks:
      - backend
  db:
    image: postgres:16
    networks:
      - backend

networks:
  backend:
```

## 7.11 Volumes in Compose

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## 7.12 Profiles

Profiles let you enable optional services.

```yaml
services:
  api:
    image: myapi:latest
  debug-ui:
    image: adminer
    profiles: ["debug"]
```

Run with:

```bash
docker compose --profile debug up
```

## 7.13 Full Multi-Container Example: Web + API + DB + Cache

```yaml
services:
  web:
    build:
      context: ./web
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy
    networks:
      - frontend
      - backend

  api:
    build:
      context: ./api
    environment:
      APP_ENV: development
      DATABASE_URL: postgres://app:secret@db:5432/appdb
      REDIS_URL: redis://cache:6379/0
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 15s
      timeout: 3s
      retries: 5
      start_period: 20s
    networks:
      - backend

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  cache:
    image: redis:7-alpine
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  pgdata:
```

## 7.14 Development Compose Example with Bind Mount

```yaml
services:
  app:
    image: node:22
    working_dir: /workspace
    command: sh -c "npm ci && npm run dev"
    ports:
      - "3000:3000"
    volumes:
      - ./:/workspace
```

## 7.15 Build Configuration Options

Common build keys:

| Key | Meaning |
|---|---|
| `context` | Build context path |
| `dockerfile` | Alternate Dockerfile path |
| `target` | Multi-stage target |
| `args` | Build arguments |
| `ssh` | SSH forwarding |
| `secrets` | Build secrets |

## 7.16 Example: Multi-Stage Target in Compose

```yaml
services:
  app:
    build:
      context: .
      target: runtime
```

## 7.17 `command` and `entrypoint` Overrides

```yaml
services:
  worker:
    image: myapp:latest
    command: ["python", "worker.py"]
```

```yaml
services:
  debug:
    image: myapp:latest
    entrypoint: ["sh", "-c"]
    command: ["sleep infinity"]
```

## 7.18 Compose Variable Substitution

Compose reads `.env` from the project directory by default.

Example `.env`:

```env
APP_TAG=1.2.3
HOST_PORT=8080
```

Compose usage:

```yaml
services:
  api:
    image: myorg/api:${APP_TAG}
    ports:
      - "${HOST_PORT}:8000"
```

## 7.19 Secrets in Compose

Avoid plain environment variables for highly sensitive data where possible.
Depending on environment and tooling support, Compose can define secrets more safely.

Example:

```yaml
services:
  app:
    image: myapp:latest
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## 7.20 Readiness vs Ordering

A classic mistake is assuming `depends_on` means “database is ready.”
It usually only helps with start order unless combined with health checks and proper app retry behavior.

Best practice:

- Add service health checks
- Make clients retry connections sensibly
- Design startup to tolerate dependency delays

## 7.21 Scaling Services in Compose

You can scale some stateless services:

```bash
docker compose up -d --scale api=3
```

Be mindful of:

- Port conflicts
- Session affinity
- Shared state issues

## 7.22 Compose Logs and Lifecycle

```bash
docker compose logs -f api
```

```bash
docker compose restart api
```

```bash
docker compose down -v
```

`-v` removes named volumes created by the stack.
Use carefully.

## 7.23 Example: Observability Sidecar in Compose

```yaml
services:
  app:
    image: myapp:latest
    ports:
      - "8000:8000"
    networks:
      - appnet

  log-shipper:
    image: fluent/fluent-bit:latest
    volumes:
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
    networks:
      - appnet

networks:
  appnet:
```

## 7.24 Compose Best Practices

- Use user-defined networks
- Keep service names stable
- Use health checks for critical dependencies
- Prefer named volumes for persistent data
- Separate dev and prod concerns
- Parameterize versions and ports with environment variables where useful

## 7.25 Compose for Production?

Compose can be acceptable for small single-host production environments, but for larger or more dynamic systems, orchestration platforms like Kubernetes or Swarm usually provide stronger scheduling and resilience features.

## 7.26 Summary

Docker Compose is excellent for defining reproducible, multi-container environments with clean service wiring and manageable configuration.

---

## B.7 Compose Q&A

### Q121. What is Compose mainly used for?
A121. Defining and running multi-container applications.

### Q122. What command starts a Compose app in background?
A122. `docker compose up -d`.

### Q123. What command stops and removes Compose resources?
A123. `docker compose down`.

### Q124. What top-level key defines services?
A124. `services`.

### Q125. What top-level key defines reusable volumes?
A125. `volumes`.

### Q126. What top-level key defines custom networks?
A126. `networks`.

### Q127. Does `depends_on` guarantee full readiness?
A127. No.

### Q128. Why combine `depends_on` with health checks?
A128. To improve dependency startup sequencing.

### Q129. What are profiles used for?
A129. Enabling optional services by scenario.

### Q130. Can Compose build images as part of startup?
A130. Yes.

### Q131. Why is `.env` useful with Compose?
A131. For variable substitution and environment-specific parameters.

### Q132. What command tails logs for a Compose app?
A132. `docker compose logs -f`.

### Q133. Can Compose scale services?
A133. Yes, for suitable stateless services.

### Q134. Why is service naming important in Compose?
A134. It affects discoverability and clarity.

### Q135. Why prefer named volumes in Compose for databases?
A135. For durable and manageable data storage.

### Q136. What is a health check in Compose?
A136. A container health evaluation command with timing controls.

### Q137. What is `build.context`?
A137. The directory sent as the build context.

### Q138. What is `target` in build config?
A138. A selected stage in a multi-stage Dockerfile.

### Q139. Why keep dev and prod Compose concerns separate?
A139. Different workflows, mounts, performance, and security requirements.

### Q140. Is Compose a full cluster orchestrator?
A140. No.

---

## F.4 Compose Keys Summary

| Key | Role |
|---|---|
| `services` | Define application containers |
| `volumes` | Define named volumes |
| `networks` | Define custom networks |
| `environment` | Define environment variables |
| `depends_on` | Startup dependency ordering |
| `healthcheck` | Define health evaluation |
| `profiles` | Optional service grouping |
