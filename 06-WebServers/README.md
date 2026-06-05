# Linux Web Servers & Services Guide

This directory is now a lightweight index instead of a monolithic handbook.

Use it as an overview, learning path, and directory map. Detailed content lives in focused topic files so each guide stays readable and easier to maintain.

## Learning path

1. Start with [01-http-fundamentals.md](./01-http-fundamentals.md) for request flow, protocol behavior, headers, caching, security, and debugging.
2. Continue with [02-apache.md](./02-apache.md) and [03-nginx.md](./03-nginx.md) for server-specific administration.
3. Use [04-ssl-tls.md](./04-ssl-tls.md) for certificate, TLS, and HTTPS deployment guidance.
4. Move to [05-reverse-proxy-lb.md](./05-reverse-proxy-lb.md) and [06-caching.md](./06-caching.md) for edge design and performance.
5. Round out the stack with the service-specific guides for databases, mail, DNS, monitoring, and high availability.

## Directory map

| File | Focus |
| --- | --- |
| [01-http-fundamentals.md](./01-http-fundamentals.md) | HTTP request flow, headers, caching, protocol versions, security, troubleshooting |
| [02-apache.md](./02-apache.md) | Apache architecture, configuration, modules, logging, and operations |
| [03-nginx.md](./03-nginx.md) | Nginx architecture, reverse proxying, tuning, and troubleshooting |
| [04-ssl-tls.md](./04-ssl-tls.md) | TLS fundamentals, certificates, server configuration, mTLS, and reference material |
| [05-reverse-proxy-lb.md](./05-reverse-proxy-lb.md) | Reverse proxy design, load balancing, session handling, and routing patterns |
| [06-caching.md](./06-caching.md) | HTTP caching, proxy caching, CDN patterns, and invalidation strategies |
| [07-database-servers.md](./07-database-servers.md) | Database server concepts for common web stacks |
| [08-mail-servers.md](./08-mail-servers.md) | Postfix, Dovecot, TLS mail flow, and mail security |
| [09-dns-servers.md](./09-dns-servers.md) | DNS operations, BIND basics, validation, and troubleshooting |
| [10-monitoring-services.md](./10-monitoring-services.md) | Monitoring stacks, metrics, alerting, and logging |
| [11-high-availability.md](./11-high-availability.md) | HA concepts, failover design, and operational testing |

## Quick reference

- HTTP and application behavior: [01-http-fundamentals.md](./01-http-fundamentals.md)
- Apache builds and vhosts: [02-apache.md](./02-apache.md)
- Nginx reverse proxy patterns: [03-nginx.md](./03-nginx.md)
- TLS and certificate lifecycle: [04-ssl-tls.md](./04-ssl-tls.md)
- Reverse proxy and edge routing: [05-reverse-proxy-lb.md](./05-reverse-proxy-lb.md)
- Cache behavior and CDN topics: [06-caching.md](./06-caching.md)
- Service-specific operations: [07-database-servers.md](./07-database-servers.md), [08-mail-servers.md](./08-mail-servers.md), [09-dns-servers.md](./09-dns-servers.md)
- Observability and HA: [10-monitoring-services.md](./10-monitoring-services.md), [11-high-availability.md](./11-high-availability.md)

## Notes on the reorganization

- The old README duplicated large parts of the numbered guides.
- Detailed sections were intentionally moved back into the focused files above.
- Existing numbered topic files remain the source of truth for deep-dive content.
