# Linux Troubleshooting Guide

This guide is now split into focused troubleshooting runbooks so you can jump directly to the subsystem or recovery workflow you need.

## Overview

- Start with methodology before making changes on a production system.
- Use the subsystem guides for boot, storage, memory, CPU, network, service, permission, package, and log issues.
- Use the recovery and scenario guides for high-pressure incidents and end-to-end practice.

## Learning Path

```mermaid
graph LR
A["Methodology"] --> B["Boot"]
A --> C["Disk"]
A --> D["Memory"]
A --> E["CPU and Performance"]
A --> F["Network"]
A --> G["Services"]
A --> H["Permissions"]
A --> I["Packages"]
A --> J["Logs"]
J --> K["Recovery"]
K --> L["Scenarios"]
```

## Table of Contents

1. [Troubleshooting Methodology](01-methodology.md)
2. [Boot Issues](02-boot-issues.md)
3. [Disk Issues](03-disk-issues.md)
4. [Memory Issues](04-memory-issues.md)
5. [CPU Issues](05-cpu-issues.md)
6. [Network Issues](06-network-issues.md)
7. [Service Issues](07-service-issues.md)
8. [Permission Issues](08-permission-issues.md)
9. [Package Issues](09-package-issues.md)
10. [Log Analysis](10-log-analysis.md)
11. [Recovery](11-recovery.md)
12. [Real-World Scenarios](12-real-world-scenarios.md)
