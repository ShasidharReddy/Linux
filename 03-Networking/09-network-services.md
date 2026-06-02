# Network Services

This section covers common Linux network services and the basics of configuring them.

## 7.1 HTTP and HTTPS overview

Common Linux web servers:

- Apache HTTP Server
- Nginx

## 7.2 Apache basics

Common package names:

- Debian/Ubuntu: `apache2`
- RHEL/CentOS: `httpd`

### 7.2.1 Start and enable Apache

```bash
sudo systemctl enable --now apache2
```

or on RHEL:

```bash
sudo systemctl enable --now httpd
```

### 7.2.2 Check listening ports

```bash
ss -tulpn | grep -E ':80|:443'
```

### 7.2.3 Common config locations

| Distro | Path |
|---|---|
| Debian/Ubuntu | `/etc/apache2/` |
| RHEL/CentOS | `/etc/httpd/` |

### 7.2.4 Basic virtual host example

```apache
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot /var/www/example
    ErrorLog ${APACHE_LOG_DIR}/example-error.log
    CustomLog ${APACHE_LOG_DIR}/example-access.log combined
</VirtualHost>
```

## 7.3 Nginx basics

### 7.3.1 Start and enable Nginx

```bash
sudo systemctl enable --now nginx
```

### 7.3.2 Basic server block example

```nginx
server {
    listen 80;
    server_name www.example.com;
    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 7.3.3 Test config

```bash
sudo nginx -t
```

## 7.4 TLS basics for HTTPS

Key elements:

- Certificate
- Private key
- CA chain
- Cipher suites
- Protocol versions
- SNI

Example Nginx TLS snippet:

```nginx
server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate /etc/ssl/certs/www.example.com.crt;
    ssl_certificate_key /etc/ssl/private/www.example.com.key;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

## 7.5 FTP and vsftpd

FTP is legacy and often replaced by SFTP or HTTPS-based transfers.

If FTP is required, `vsftpd` is common.

### 7.5.1 Start and enable `vsftpd`

```bash
sudo systemctl enable --now vsftpd
```

### 7.5.2 Minimal config concepts

- Local user access
- Chroot jails
- Passive ports
- TLS for FTPS

### 7.5.3 Passive mode note

FTP passive mode requires a port range to be allowed in the firewall.

## 7.6 NFS overview

NFS is common for Linux-to-Linux file sharing.

Packages and services may include:

- `nfs-kernel-server`
- `nfs-utils`
- `rpcbind`

### 7.6.1 Export example

File:

```text
/etc/exports
```

Example:

```exports
/srv/nfs/share 10.10.20.0/24(rw,sync,no_subtree_check)
```

Apply exports:

```bash
sudo exportfs -rav
```

### 7.6.2 Mount example

```bash
sudo mount -t nfs server:/srv/nfs/share /mnt/share
```

### 7.6.3 Persistent mount via `/etc/fstab`

```fstab
server:/srv/nfs/share  /mnt/share  nfs  defaults,_netdev  0  0
```

## 7.7 Samba/CIFS overview

Samba provides SMB/CIFS file sharing for Windows and mixed environments.

### 7.7.1 Basic share example

File:

```text
/etc/samba/smb.conf
```

Example share:

```ini
[shared]
    path = /srv/samba/shared
    browseable = yes
    read only = no
    guest ok = no
```

### 7.7.2 Manage Samba users

```bash
sudo smbpasswd -a alice
```

### 7.7.3 Test config

```bash
testparm
```

## 7.8 DHCP server overview

Common Linux DHCP service:

- ISC DHCP server

### 7.8.1 Example DHCP scope

```conf
subnet 10.10.20.0 netmask 255.255.255.0 {
    range 10.10.20.100 10.10.20.200;
    option routers 10.10.20.1;
    option domain-name-servers 10.10.1.53, 1.1.1.1;
    option domain-name "example.com";
    default-lease-time 600;
    max-lease-time 7200;
}
```

### 7.8.2 Static reservation example

```conf
host web01 {
    hardware ethernet 52:54:00:12:34:56;
    fixed-address 10.10.20.15;
}
```

## 7.9 DNS server service overview

A DNS server may be:

- Recursive resolver
- Authoritative server
- Both, in smaller environments

Common implementations:

- BIND
- Unbound
- dnsmasq
- PowerDNS

## 7.10 `dnsmasq` for small environments

`dnsmasq` is lightweight and excellent for labs and edge systems.

Example config snippet:

```conf
interface=eth0
dhcp-range=10.10.20.100,10.10.20.200,12h
dhcp-option=option:router,10.10.20.1
dhcp-option=option:dns-server,10.10.20.1
address=/lab.local/10.10.20.10
```

## 7.11 Service binding and exposure

When deploying services, always check:

- Bind address
- Port
- Firewall rules
- SELinux contexts
- TLS configuration
- Reverse proxy behavior

## 7.12 Common service ports

| Service | Port | Protocol |
|---|---:|---|
| HTTP | 80 | TCP |
| HTTPS | 443 | TCP |
| FTP | 21 | TCP |
| SSH | 22 | TCP |
| DNS | 53 | UDP/TCP |
| DHCP server | 67 | UDP |
| DHCP client | 68 | UDP |
| NFS | 2049 | TCP/UDP |
| SMB | 445 | TCP |

## 7.13 Reverse proxy pattern

A common design is:

- Nginx or Apache on ports 80 and 443
- Application bound to localhost on 8080 or 3000
- Reverse proxy handles TLS and client connections

## 7.14 Health checks and monitoring

Monitor:

- Port availability
- HTTP status code
- TLS certificate expiry
- DNS response time
- NFS mount health
- Samba authentication failures

## 7.15 Logging locations

| Service | Common Logs |
|---|---|
| Apache | `/var/log/apache2/` or `/var/log/httpd/` |
| Nginx | `/var/log/nginx/` |
| SSH | `/var/log/auth.log` or journal |
| BIND | journal or named logs |
| Samba | `/var/log/samba/` |
| DHCP | journal or syslog |

## 7.16 SELinux notes for network services

Examples:

```bash
sudo semanage port -l | grep http
sudo setsebool -P httpd_can_network_connect 1
```

## 7.17 Quick deployment checklist

- Assign static IP or DHCP reservation
- Confirm DNS record
- Open firewall port
- Validate service config syntax
- Start and enable service
- Test locally and remotely
- Add monitoring

## 7.18 Summary

Linux can host nearly every common network service. Reliability depends on clean configuration, correct exposure, and good observability.

---

## 12.13 Build an internal reverse proxy for private applications

Objective:

- Expose one HTTPS endpoint such as `portal.corp.example.internal`
- Route by hostname or path to internal services

Example Nginx config:

```nginx
server {
    listen 443 ssl http2;
    server_name portal.corp.example.internal;

    ssl_certificate /etc/nginx/tls/portal.crt;
    ssl_certificate_key /etc/nginx/tls/portal.key;

    location /grafana/ {
        proxy_pass http://10.20.30.51:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /argocd/ {
        proxy_pass https://10.20.30.52:443/;
        proxy_ssl_server_name on;
        proxy_set_header Host $host;
    }
}
```

Checks:

```bash
sudo nginx -t
curl -vk https://portal.corp.example.internal/grafana/
```

---

# Service Deployment Checklists

## D.5 Service checklist

- [ ] Validate config syntax before restart.
- [ ] Confirm service binds to intended address only.
- [ ] Open firewall ports explicitly.
- [ ] Add monitoring and health checks.
- [ ] Check SELinux or AppArmor if service fails unexpectedly.

---

## E.11 Service exposure models

| Model | Description |
|---|---|
| Public direct | Service exposed directly on public IP |
| Reverse proxy | Frontend proxy terminates HTTP or TLS |
| Load balanced | Multiple backends behind VIP or LB |
| Private only | Service reachable only over VPN or internal network |

---

## X.1 Minimal web server exposure checklist

1. Install Nginx.
2. Confirm local listener.
3. Open port 80 or 443.
4. Confirm DNS points to host.
5. Test from local host.
6. Test from remote client.
7. Add TLS and redirect HTTP to HTTPS.
