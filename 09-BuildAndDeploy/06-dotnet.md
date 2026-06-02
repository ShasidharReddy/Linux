# .NET Applications

[Back to guide index](README.md)

## 6.1 .NET on Linux

Modern .NET runs well on Linux.

Common app types:

- Console apps
- ASP.NET Core APIs
- Background workers
- Minimal APIs
- MVC applications

## 6.2 Installing the .NET SDK

Install from Microsoft package repositories.

Ubuntu example outline:

```bash
# apt update
# apt install -y dotnet-sdk-8.0
```

Verify:

```bash
$ dotnet --info
$ dotnet --list-sdks
$ dotnet --list-runtimes
```

## 6.3 Basic Build Workflow

Restore dependencies:

```bash
$ dotnet restore
```

Build:

```bash
$ dotnet build -c Release
```

Test:

```bash
$ dotnet test -c Release --no-build
```

Publish:

```bash
$ dotnet publish -c Release -o publish
```

## 6.4 Publish Modes

Common publish modes:

- Framework-dependent deployment
- Self-contained deployment

Framework-dependent means the target host has the required runtime installed.

Self-contained bundles the runtime.

Framework-dependent example:

```bash
$ dotnet publish -c Release -o publish
```

Self-contained example:

```bash
$ dotnet publish -c Release -r linux-x64 --self-contained true -o publish
```

Trade-offs:

| Mode | Pros | Cons |
|---|---|---|
| Framework-dependent | Smaller artifacts | Runtime must exist on host |
| Self-contained | Simpler host runtime requirements | Larger artifacts |

## 6.5 ASP.NET Core and Kestrel

Kestrel is the built-in web server for ASP.NET Core.

Typical production pattern:

- Kestrel listens on localhost
- Nginx reverse proxies external traffic to Kestrel

Example environment:

```bash
ASPNETCORE_URLS=http://127.0.0.1:5000
ASPNETCORE_ENVIRONMENT=Production
```

## 6.6 systemd Service for ASP.NET Core

Example service file:

```ini
[Unit]
Description=ASP.NET Core Application
After=network.target

[Service]
WorkingDirectory=/opt/dotnetapp/current
ExecStart=/usr/bin/dotnet /opt/dotnetapp/current/MyApp.dll
Restart=always
RestartSec=5
KillSignal=SIGINT
SyslogIdentifier=dotnetapp
User=dotnetapp
Group=dotnetapp
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=ASPNETCORE_URLS=http://127.0.0.1:5000

[Install]
WantedBy=multi-user.target
```

## 6.7 Nginx Reverse Proxy for ASP.NET Core

```nginx
server {
    listen 80;
    server_name dotnet.example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Add helper mapping in the `http` block if needed:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

## 6.8 Publish Directory Contents

Typical output:

- Application DLLs
- Dependency DLLs
- Config files
- Static assets
- Runtime files in self-contained publishes

## 6.9 Deploying ASP.NET Core

Typical process:

1. `dotnet publish -c Release`
2. Copy `publish/` contents to the target release directory
3. Update symlink or deployment path
4. Restart or reload systemd service
5. Validate health endpoint

## 6.10 Single-File Publish

Example:

```bash
$ dotnet publish -c Release -r linux-x64 -p:PublishSingleFile=true -o publish
```

Use when operational simplicity is more important than extraction/startup considerations.

## 6.11 Common .NET Deployment Mistakes

- Publishing framework-dependent apps without installing the runtime
- Forgetting `ASPNETCORE_ENVIRONMENT`
- Binding Kestrel directly to public interfaces unintentionally
- Not forwarding proxy headers correctly
- Rebuilding on the production host instead of deploying publish output

---
