# Build Systems & Application Deployment Guide for Linux

## Overview

This guide is a production-focused reference for building, packaging, deploying, operating, and troubleshooting applications on Linux.

It is intentionally practical.

It covers:

- Native compiled applications
- JVM applications
- Python services
- Node.js services
- Go binaries
- .NET services
- Web server integration
- Process management
- Deployment strategies
- CI/CD patterns
- Monitoring and troubleshooting
- Configuration and secrets management

It is written for:

- Linux administrators
- DevOps engineers
- Backend developers
- SRE teams
- Platform engineers
- Full-stack developers responsible for deployments

Assumptions:

- Target systems are Linux servers
- Bash is available
- You have sudo access when installing packages
- You are deploying server-side applications or services
- You want reliable, repeatable, and observable deployments

Conventions used in this guide:

- Commands prefixed with `$` are run as a regular user
- Commands prefixed with `#` are run as root or with sudo
- File paths are shown as Linux absolute paths where appropriate
- systemd is treated as the default init and service manager
- Nginx is the primary reverse proxy example

---

## Learning Path

```mermaid
graph TD
    N1["01 Build Systems"]
    N2["02 Java and JVM Applications"]
    N3["03 Python Applications"]
    N4["04 Node.js Applications"]
    N5["05 Go Applications"]
    N6["06 .NET Applications"]
    N7["07 Web Server Configuration"]
    N8["08 Process Management and Configuration"]
    N9["09 Deployment Strategies"]
    N10["10 CI/CD Integration"]
    N11["11 Application Monitoring"]
    N12["12 Troubleshooting Applications"]
    N1 --> N2
    N2 --> N3
    N3 --> N4
    N4 --> N5
    N5 --> N6
    N6 --> N7
    N7 --> N8
    N8 --> N9
    N9 --> N10
    N10 --> N11
    N11 --> N12
```

## Table of Contents

- [Build Systems](01-build-systems.md)
- [Java and JVM Applications](02-java-jvm.md)
- [Python Applications](03-python.md)
- [Node.js Applications](04-nodejs.md)
- [Go Applications](05-golang.md)
- [.NET Applications](06-dotnet.md)
- [Web Server Configuration](07-web-server-config.md)
- [Process Management and Configuration](08-process-management.md)
- [Deployment Strategies](09-deployment-strategies.md)
- [CI/CD Integration](10-cicd-integration.md)
- [Application Monitoring](11-monitoring.md)
- [Troubleshooting Applications](12-troubleshooting.md)
