# 16. Setting Up Essential Services
## 16.1 Service Setup Strategy
For foundational Linux services, follow the same operational pattern every time:
1. install the package
2. enable and start the service
3. open the correct firewall access
4. validate the configuration syntax
5. test locally and from another host
6. check logs for errors
7. document the change and rollback steps
## 16.2 Setting Up SSH Server
The pattern is the same on most distributions: install the package, enable the service, open the firewall, harden the config, and verify from another machine before signing out.
### Ubuntu and Debian
Install and start OpenSSH server:
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh --no-pager
```
Expected status excerpt:
```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
     Active: active (running)
```
Confirm the listening socket:
```bash
sudo ss -tulpn | grep :22
```
Allow SSH through UFW if enabled:
```bash
sudo ufw allow OpenSSH
sudo ufw status verbose
```
### RHEL and CentOS
```bash
sudo dnf install -y openssh-server
sudo systemctl enable --now sshd
sudo systemctl status sshd --no-pager
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
```
### Harden `sshd_config`
Example:
```sshconfig
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers admin deploy
LoginGraceTime 30
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```
Validate and restart:
```bash
sudo sshd -t
sudo systemctl restart sshd
sudo systemctl restart ssh
```
Check logs:
```bash
sudo tail -n 50 /var/log/auth.log
sudo tail -n 50 /var/log/secure
sudo journalctl -u sshd --no-pager -n 50
```
Always keep one working session open while testing changes.
## 16.3 Setting Up NFS Server
### Ubuntu NFS server
```bash
sudo apt update
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/share
sudo chown nobody:nogroup /srv/nfs/share
sudo chmod 0775 /srv/nfs/share
```
Add to `/etc/exports`:
```exports
/srv/nfs/share 192.168.1.0/24(rw,sync,no_subtree_check)
```
Apply and verify:
```bash
sudo exportfs -ra
sudo exportfs -v
sudo systemctl enable --now nfs-kernel-server
sudo systemctl status nfs-kernel-server --no-pager
```
### RHEL/CentOS NFS server
```bash
sudo dnf install -y nfs-utils
sudo mkdir -p /srv/nfs/share
sudo chown nfsnobody:nfsnobody /srv/nfs/share
sudo chmod 0775 /srv/nfs/share
```
Add the same export definition to `/etc/exports`, then run:
```bash
sudo exportfs -ra
sudo systemctl enable --now nfs-server
sudo systemctl status nfs-server --no-pager
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --reload
```
If SELinux is enforcing, validate contexts and any required booleans for the design you chose.
### NFS client setup
Ubuntu or Debian client:
```bash
sudo apt update
sudo apt install -y nfs-common
```
RHEL or CentOS client:
```bash
sudo dnf install -y nfs-utils
```
Mount the share:
```bash
sudo mkdir -p /mnt/nfs
sudo mount -t nfs nfs01.example.com:/srv/nfs/share /mnt/nfs
```
Verify:
```bash
df -hT /mnt/nfs
ls -la /mnt/nfs
```
Example output:
```text
Filesystem                       Type  Size  Used Avail Use% Mounted on
nfs01.example.com:/srv/nfs/share nfs4   50G  2.0G   48G   4% /mnt/nfs
```
Persistent mount:
```fstab
nfs01.example.com:/srv/nfs/share  /mnt/nfs  nfs  defaults,_netdev  0  0
```
Test without rebooting:
```bash
sudo umount /mnt/nfs
sudo mount -a
```
`autofs` is useful when you want mounts to appear only when accessed.
## 16.4 Setting Up DNS Server (BIND9)
BIND is a classic authoritative DNS server package used for internal zones, forward lookups, reverse zones, and lab environments.
### Ubuntu or Debian installation
```bash
sudo apt update
sudo apt install -y bind9 bind9-utils bind9-dnsutils
```
Common files:
- `/etc/bind/named.conf`
- `/etc/bind/named.conf.options`
- `/etc/bind/named.conf.local`
- `/etc/bind/db.*`
### RHEL or CentOS installation
```bash
sudo dnf install -y bind bind-utils
```
Common files:
- `/etc/named.conf`
- `/var/named/`
### Forward and reverse zone definitions
Ubuntu or Debian in `/etc/bind/named.conf.local`:
```conf
zone "example.internal" {
    type master;
    file "/etc/bind/db.example.internal";
};
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
};
```
RHEL or CentOS in `/etc/named.conf` or an included file:
```conf
zone "example.internal" IN {
    type master;
    file "/var/named/db.example.internal";
    allow-update { none; };
};
zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/db.192.168.1";
    allow-update { none; };
};
```
### Example forward zone file
```dns
$TTL 3600
@   IN  SOA ns1.example.internal. admin.example.internal. (
        2025060101
        3600
        900
        604800
        86400 )
    IN  NS  ns1.example.internal.
ns1 IN  A   192.168.1.10
web1 IN  A   192.168.1.20
app1 IN  A   192.168.1.30
db1  IN  A   192.168.1.40
```
Increment the serial whenever you modify the zone; a `YYYYMMDDNN` scheme works well.
### Example reverse zone file
```dns
$TTL 3600
@   IN  SOA ns1.example.internal. admin.example.internal. (
        2025060101
        3600
        900
        604800
        86400 )
    IN  NS  ns1.example.internal.
10  IN  PTR ns1.example.internal.
20  IN  PTR web1.example.internal.
30  IN  PTR app1.example.internal.
40  IN  PTR db1.example.internal.
```
### Validate the configuration
```bash
sudo named-checkconf
sudo named-checkconf /etc/named.conf
sudo named-checkzone example.internal /etc/bind/db.example.internal
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1
```
Healthy output looks like:
```text
zone example.internal/IN: loaded serial 2025060101
OK
```
### Start the service and test
Ubuntu or Debian:
```bash
sudo systemctl enable --now bind9
sudo systemctl status bind9 --no-pager
```
RHEL or CentOS:
```bash
sudo systemctl enable --now named
sudo systemctl status named --no-pager
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
```
Query the server directly:
```bash
dig @192.168.1.10 web1.example.internal A
dig @192.168.1.10 -x 192.168.1.20
```
Expected answer excerpt:
```text
;; ANSWER SECTION:
web1.example.internal. 3600 IN A 192.168.1.20
```
## 16.5 Setting Up HTTP Server
### Quick Nginx setup
Ubuntu or Debian:
```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
sudo ufw allow 'Nginx Full'
```
RHEL or CentOS:
```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
Create test content:
```bash
echo '<h1>Nginx is serving this site</h1>' | sudo tee /var/www/html/index.html
curl -I http://127.0.0.1/
```
Example response:
```text
HTTP/1.1 200 OK
Server: nginx/1.24.0
Content-Type: text/html
```
If SELinux is enforcing and you changed document roots, restore contexts:
```bash
sudo restorecon -Rv /var/www/html
```
Example Nginx server block in `/etc/nginx/conf.d/example.conf`:
```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example;
    index index.html;
    access_log /var/log/nginx/example.access.log;
    error_log /var/log/nginx/example.error.log;
    location / {
        try_files $uri $uri/ =404;
    }
}
```
Create content and reload:
```bash
sudo mkdir -p /var/www/example
printf '%s\n' '<h1>Hello from Nginx</h1>' | sudo tee /var/www/example/index.html
sudo nginx -t
sudo systemctl reload nginx
```
### Quick Apache setup
Ubuntu or Debian:
```bash
sudo apt update
sudo apt install -y apache2
sudo systemctl enable --now apache2
sudo systemctl status apache2 --no-pager
sudo ufw allow 'Apache Full'
```
RHEL or CentOS:
```bash
sudo dnf install -y httpd
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
Create test content:
```bash
echo '<h1>Apache is serving this site</h1>' | sudo tee /var/www/html/index.html
curl -I http://127.0.0.1/
```
Example response:
```text
HTTP/1.1 200 OK
Server: Apache/2.4.58
Content-Type: text/html; charset=UTF-8
```
Example Apache virtual host:
```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
    ErrorLog ${APACHE_LOG_DIR}/example-error.log
    CustomLog ${APACHE_LOG_DIR}/example-access.log combined
    <Directory /var/www/example>
        Require all granted
        AllowOverride None
    </Directory>
</VirtualHost>
```
Create content and enable it:
```bash
sudo mkdir -p /var/www/example
printf '%s\n' '<h1>Hello from Apache</h1>' | sudo tee /var/www/example/index.html
sudo a2ensite example.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo httpd -t
sudo systemctl reload httpd
```
## 16.6 Service Verification Checklist
For any service you set up, verify the same basics every time:
1. package installed successfully
2. service enabled and running
3. port or socket is listening
4. firewall allows the intended traffic
5. config syntax checks pass
6. logs show a clean startup
7. client access works from another machine
Useful commands:
```bash
systemctl status sshd --no-pager
systemctl status nfs-server --no-pager
systemctl status named --no-pager
systemctl status nginx --no-pager
systemctl status httpd --no-pager
ss -tulpn
journalctl -u sshd --no-pager -n 50
journalctl -u nginx --no-pager -n 50
journalctl -u httpd --no-pager -n 50
```
## 16.7 Production Notes
- Prefer automation or configuration management for repeatable builds.
- Keep configuration files in version control when possible.
- Back up configs before major changes.
- Test syntax before restart or reload.
- Restrict exposure with firewalls and security groups.
- Use TLS for public-facing HTTP services.
- Monitor health checks and logs after deployment.
- Document custom ports, ownership, and rollback steps.
