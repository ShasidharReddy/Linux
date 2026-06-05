# TLS Command and Troubleshooting Reference

← Back to [04-ssl-tls.md](./04-ssl-tls.md)

This focused reference keeps the unique cookbook material from the original appendix and removes the repeated operational-note boilerplate.

---

## Quick command cookbook

- Show certificate chain from live host: `openssl s_client -connect example.com:443 -servername example.com -showcerts`
- Show only certificate dates: `openssl x509 -in server.crt -noout -dates`
- Generate RSA key: `openssl genrsa -out server.key 4096`
- Generate CSR: `openssl req -new -key server.key -out server.csr`
- Check expiry from a live host: `echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates`
- Test nginx config: `sudo nginx -t`
- Reload nginx: `sudo systemctl reload nginx`
- Dry-run renewal: `sudo certbot renew --dry-run`
- Inspect SAN values: `openssl x509 -in server.crt -noout -text | grep -A2 "Subject Alternative Name"`
- Check HTTP headers: `curl -I https://example.com`

## What to check first when something breaks

| Symptom | Check first |
|---|---|
| Handshake fails immediately | Protocol compatibility, TCP reachability, SNI |
| Browser hostname warning | SAN values and virtual host routing |
| Only some clients fail | Intermediate chain and legacy compatibility |
| Renewed cert not visible | Service reload and edge cache layers |
| OCSP stapling missing | Resolver and outbound CA access |
| mTLS client rejected | Client cert chain, EKU, and trusted client CA |

## Consolidated operational reminders

The original appendix ended with 150 numbered notes that repeated the same two ideas with only the note number changed. They were collapsed into the deduplicated reminders below.

1. TLS incidents are rarely only cryptographic; verify DNS, TCP reachability, SNI, reverse proxies, service reload behavior, and certificate placement together.
2. Certificate lifecycle management matters as much as initial issuance; always track renewal paths, deploy hooks, trust-store assumptions, and monitoring alerts.
3. When troubleshooting clients, compare protocol versions, cipher support, chain completeness, hostname validation, and whether an intermediate device is modifying the connection.
4. Treat TLS as part of the full application path: edge configuration, logging, observability, automation, and rollback procedures all affect successful HTTPS delivery.
