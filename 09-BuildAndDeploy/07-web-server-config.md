# Web Server Configuration

[Back to guide index](README.md)

## 7.1 Why Use a Reverse Proxy

Reverse proxies provide:

- TLS termination
- Request routing
- Compression
- Static file serving
- Connection buffering
- Security controls
- Load balancing
- WebSocket support

Common choices:

- Nginx
- Apache HTTP Server
- HAProxy

This section emphasizes Nginx and Apache.

## 7.2 Nginx Reverse Proxy Essentials

Typical proxy headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

These headers are critical for:

- Accurate client IP logging
- Redirect generation
- Scheme-aware apps
- Audit trails

## 7.3 Complete Nginx Config for a Java App

```nginx
server {
    listen 80;
    server_name java.example.com;

    access_log /var/log/nginx/java-access.log;
    error_log /var/log/nginx/java-error.log warn;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}
```

## 7.4 Complete Nginx Config for Python Gunicorn

```nginx
server {
    listen 80;
    server_name python.example.com;

    client_max_body_size 20m;

    location /static/ {
        alias /opt/pythonapp/shared/static/;
        expires 7d;
        add_header Cache-Control "public";
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}
```

## 7.5 Complete Nginx Config for Node.js

```nginx
server {
    listen 80;
    server_name node.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 7.6 Complete Nginx Config for Go

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

## 7.7 Complete Nginx Config for ASP.NET Core

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

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

## 7.8 Static File Serving with Nginx

Serving static assets directly is efficient.

Example:

```nginx
location /assets/ {
    alias /opt/webapp/current/assets/;
    expires 30d;
    add_header Cache-Control "public, immutable";
}
```

Benefits:

- Lower application CPU usage
- Better caching behavior
- Reduced latency

## 7.9 Upload Size Controls

Example:

```nginx
client_max_body_size 50m;
```

Set appropriately for:

- File uploads
- API payloads
- Administrative imports

## 7.10 WebSocket Proxying

WebSockets require HTTP/1.1 and upgrade headers.

Example:

```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

## 7.11 Apache with mod_proxy

Enable modules:

```bash
# a2enmod proxy proxy_http headers ssl rewrite
```

Basic virtual host:

```apache
<VirtualHost *:80>
    ServerName app.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Forwarded-Port "80"
</VirtualHost>
```

## 7.12 Apache WebSocket Proxy Example

```apache
<VirtualHost *:80>
    ServerName socket.example.com

    ProxyPreserveHost On
    ProxyPass /ws/ ws://127.0.0.1:3000/ws/
    ProxyPassReverse /ws/ ws://127.0.0.1:3000/ws/
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

## 7.13 SSL/TLS with Let's Encrypt

Using Certbot with Nginx on Ubuntu:

```bash
# apt install -y certbot python3-certbot-nginx
# certbot --nginx -d app.example.com
```

Using Certbot with Apache:

```bash
# apt install -y certbot python3-certbot-apache
# certbot --apache -d app.example.com
```

Key benefits:

- Free certificates
- Automated renewal
- Easy integration

## 7.14 HTTPS Redirect

Nginx example:

```nginx
server {
    listen 80;
    server_name app.example.com;
    return 301 https://$host$request_uri;
}
```

## 7.15 Hardened TLS Basics

Minimum recommendations:

- Disable obsolete protocols
- Prefer TLS 1.2 and TLS 1.3
- Use strong ciphers where relevant
- Enable HSTS after validation

Nginx snippet:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

## 7.16 Testing Web Server Configurations

Nginx:

```bash
# nginx -t
# systemctl reload nginx
```

Apache:

```bash
# apachectl configtest
# systemctl reload apache2
```

## 7.17 Common Reverse Proxy Mistakes

- Missing `Host` and forwarding headers
- Not enabling upgrade headers for WebSockets
- Serving large uploads without adjusting body size limits
- Proxying to public interfaces instead of localhost
- Forgetting to validate config before reload

---
