# 02 - VM-Based E-Commerce Setup
<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                 VM-Based E-Commerce Setup                   │
└──────────────────────────────────────────────────────────────┘
</pre></div>
This guide deploys a complete e-commerce environment on KVM virtual machines. It assumes you already understand the basics from [01-virtualization-fundamentals.md](./01-virtualization-fundamentals.md). The design uses six VMs, three virtual networks, HAProxy load balancing, Nginx web nodes, application nodes, MySQL primary/replica replication, and Redis caching.
## Scenario
We will build the following platform:
- `web01` and `web02` for Nginx and static assets
- `app01` and `app02` for PHP-FPM and Node.js background tasks
- `db01` as MySQL primary
- `db02` as MySQL replica
- HAProxy on `web01` or a dedicated edge host for traffic distribution
- Redis on `app01` for session and cache acceleration
This layout is suitable for:
- a small to medium traffic e-commerce platform
- migration practice before adopting containers or Kubernetes
## Target Topology
### VM Provisioning Plan
| VM | Role | vCPU | RAM | Storage | Networks | Primary Software |
| --- | --- | --- | --- | --- | --- | --- |
| web01 | Web + HAProxy | 2 | 4 GB | 40 GB | frontend, backend | Nginx, HAProxy |
| web02 | Web | 2 | 4 GB | 40 GB | frontend, backend | Nginx |
| app01 | App + Redis | 4 | 8 GB | 60 GB | backend, database | PHP-FPM, Node.js, Redis |
| app02 | App | 4 | 8 GB | 60 GB | backend, database | PHP-FPM, Node.js |
| db01 | MySQL primary | 8 | 32 GB | 200 GB | database | MySQL Server |
| db02 | MySQL replica | 8 | 32 GB | 200 GB | database | MySQL Server |
### Network Segments
| Network | CIDR | Purpose |
| --- | --- | --- |
| frontend-net | 192.168.10.0/24 | Client-facing web ingress |
| backend-net | 192.168.20.0/24 | Web-to-app traffic |
| database-net | 192.168.30.0/24 | App-to-DB and DB replication |
### Network Topology
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    Clients[Users / Browsers] --> LB[HAProxy on web01]
    LB --> W1[web01 Nginx]
    LB --> W2[web02 Nginx]
    W1 --> A1[app01 PHP-FPM and Node]
    W1 --> A2[app02 PHP-FPM and Node]
    W2 --> A1
    W2 --> A2
    A1 --> R1[Redis on app01]
    A1 --> DB1[db01 MySQL Primary]
    A2 --> DB1
    DB1 --> DB2[db02 MySQL Replica]
~~~
## IP Address Plan
| Hostname | frontend-net | backend-net | database-net |
| --- | --- | --- | --- |
| web01 | 192.168.10.11 | 192.168.20.11 | - |
| web02 | 192.168.10.12 | 192.168.20.12 | - |
| app01 | - | 192.168.20.21 | 192.168.30.21 |
| app02 | - | 192.168.20.22 | 192.168.30.22 |
| db01 | - | - | 192.168.30.31 |
| db02 | - | - | 192.168.30.32 |
## Deployment Workflow
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Create virtual networks] --> B[Clone VMs from template]
    B --> C[Assign hostnames and static IPs]
    C --> D[Install Nginx, PHP-FPM, Node.js]
    D --> E[Install MySQL and replication]
    E --> F[Install Redis]
    F --> G[Configure HAProxy]
    G --> H[Deploy ecommerce app]
    H --> I[Enable TLS]
    I --> J[Snapshot and validate]
~~~
## Prerequisites
- a KVM host with libvirt enabled
- a clean golden image as described in [01](./01-virtualization-fundamentals.md)
- reachable package repositories from guests
- DNS or `/etc/hosts` entries for lab resolution
- SSH access to all VMs
Useful host-side commands:
~~~bash
virsh list --all
virsh net-list --all
virsh pool-list --all
~~~
## Step 1 - Create Virtual Networks with virsh
We will create three NAT-backed networks for isolation. You can switch to bridge-backed networks later if you need direct L2 reachability from your LAN.
### frontend-net definition
~~~bash
cat > frontend-net.xml <<'EOF'
<network>
  <name>frontend-net</name>
  <bridge name='virbr10' stp='on' delay='0'/>
  <forward mode='nat'/>
  <ip address='192.168.10.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.10.100' end='192.168.10.200'/>
    </dhcp>
  </ip>
</network>
EOF
virsh net-define frontend-net.xml
virsh net-start frontend-net
virsh net-autostart frontend-net
~~~
### backend-net definition
~~~bash
cat > backend-net.xml <<'EOF'
<network>
  <name>backend-net</name>
  <bridge name='virbr20' stp='on' delay='0'/>
  <forward mode='nat'/>
  <ip address='192.168.20.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.20.100' end='192.168.20.200'/>
    </dhcp>
  </ip>
</network>
EOF
virsh net-define backend-net.xml
virsh net-start backend-net
virsh net-autostart backend-net
~~~
### database-net definition
~~~bash
cat > database-net.xml <<'EOF'
<network>
  <name>database-net</name>
  <bridge name='virbr30' stp='on' delay='0'/>
  <forward mode='nat'/>
  <ip address='192.168.30.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.30.100' end='192.168.30.200'/>
    </dhcp>
  </ip>
</network>
EOF
virsh net-define database-net.xml
virsh net-start database-net
virsh net-autostart database-net
virsh net-list --all
~~~
Tip:
- for stricter environments, define isolated networks and route traffic explicitly through firewalls or reverse proxies.
## Step 2 - Provision VMs from a Template
Assume a base image exists at `/var/lib/libvirt/images/ubuntu-2404-template.qcow2`.
### Clone disks
~~~bash
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/web01.qcow2 40G
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/web02.qcow2 40G
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/app01.qcow2 60G
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/app02.qcow2 60G
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/db01.qcow2 200G
qemu-img create -f qcow2 -b /var/lib/libvirt/images/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/db02.qcow2 200G
~~~
### Create `web01`
~~~bash
virt-install \
  --name web01 \
  --memory 4096 \
  --vcpus 2 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/web01.qcow2,format=qcow2,bus=virtio \
  --network network=frontend-net,model=virtio \
  --network network=backend-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
~~~
### Create `web02`
~~~bash
virt-install \
  --name web02 \
  --memory 4096 \
  --vcpus 2 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/web02.qcow2,format=qcow2,bus=virtio \
  --network network=frontend-net,model=virtio \
  --network network=backend-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
~~~
### Create `app01` and `app02`
~~~bash
virt-install \
  --name app01 \
  --memory 8192 \
  --vcpus 4 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/app01.qcow2,format=qcow2,bus=virtio \
  --network network=backend-net,model=virtio \
  --network network=database-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
virt-install \
  --name app02 \
  --memory 8192 \
  --vcpus 4 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/app02.qcow2,format=qcow2,bus=virtio \
  --network network=backend-net,model=virtio \
  --network network=database-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
~~~
### Create `db01` and `db02`
~~~bash
virt-install \
  --name db01 \
  --memory 32768 \
  --vcpus 8 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/db01.qcow2,format=qcow2,bus=virtio \
  --network network=database-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
virt-install \
  --name db02 \
  --memory 32768 \
  --vcpus 8 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/db02.qcow2,format=qcow2,bus=virtio \
  --network network=database-net,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
~~~
## Step 3 - Configure Networking, Static IPs, and Hostnames
Use cloud-init if you baked it into the template, or configure Netplan manually.
### Example for `web01` `/etc/netplan/01-ecommerce.yaml`
~~~yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
      addresses:
        - 192.168.10.11/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
    ens4:
      dhcp4: false
      addresses:
        - 192.168.20.11/24
~~~
Apply it:
~~~bash
sudo hostnamectl set-hostname web01
sudo netplan apply
ip addr
~~~
### Example for `app01`
~~~yaml
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: false
      addresses:
        - 192.168.20.21/24
      routes:
        - to: default
          via: 192.168.20.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
    ens4:
      dhcp4: false
      addresses:
        - 192.168.30.21/24
~~~
### Example `/etc/hosts` entries on all nodes
~~~text
192.168.10.11 web01
192.168.10.12 web02
192.168.20.11 web01-back
192.168.20.12 web02-back
192.168.20.21 app01
192.168.20.22 app02
192.168.30.21 app01-db
192.168.30.22 app02-db
192.168.30.31 db01
192.168.30.32 db02
~~~
Verification:
~~~bash
ping -c 2 app01
ping -c 2 db01
ssh app01 hostname
~~~
## Step 4 - Install Nginx on Web VMs
Run on both `web01` and `web02`:
~~~bash
sudo apt-get update
sudo apt-get install -y nginx curl unzip
sudo systemctl enable --now nginx
~~~
### Nginx site configuration
Create `/etc/nginx/sites-available/ecommerce.conf`:
~~~nginx
upstream php_backend {
    server 192.168.20.21:9000 max_fails=3 fail_timeout=10s;
    server 192.168.20.22:9000 max_fails=3 fail_timeout=10s;
    keepalive 32;
}
server {
    listen 80;
    server_name shop.example.internal;
    root /var/www/ecommerce/public;
    index index.php index.html;
    access_log /var/log/nginx/ecommerce.access.log;
    error_log /var/log/nginx/ecommerce.error.log warn;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~* \.(css|js|jpg|jpeg|png|gif|svg|ico|woff2?)$ {
        expires 7d;
        add_header Cache-Control "public, no-transform";
        try_files $uri =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass php_backend;
        fastcgi_read_timeout 120s;
    }
    location /healthz {
        access_log off;
        return 200 'ok';
        add_header Content-Type text/plain;
    }
    client_max_body_size 32m;
}
~~~
Enable it:
~~~bash
sudo mkdir -p /var/www/ecommerce/public
sudo ln -s /etc/nginx/sites-available/ecommerce.conf /etc/nginx/sites-enabled/ecommerce.conf
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
~~~
## Step 5 - Install PHP-FPM and Node.js on App VMs
### Install packages on `app01` and `app02`
~~~bash
sudo apt-get update
sudo apt-get install -y php-fpm php-mysql php-xml php-mbstring php-curl php-zip php-bcmath composer git unzip redis-tools
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
php -v
node -v
npm -v
~~~
### PHP-FPM pool tuning
Edit `/etc/php/8.3/fpm/pool.d/www.conf`:
~~~ini
[www]
user = www-data
group = www-data
listen = 0.0.0.0:9000
listen.owner = www-data
listen.group = www-data
pm = dynamic
pm.max_children = 40
pm.start_servers = 8
pm.min_spare_servers = 4
pm.max_spare_servers = 12
pm.max_requests = 500
catch_workers_output = yes
php_admin_value[memory_limit] = 512M
~~~
Restart PHP-FPM:
~~~bash
sudo systemctl enable --now php8.3-fpm
sudo systemctl restart php8.3-fpm
sudo ss -lntp | grep 9000
~~~
### Optional Node.js worker service
Create `/opt/ecommerce/worker.js`:
~~~javascript
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.url === '/healthz') {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('ok');
    return;
  }
  res.writeHead(200, {'Content-Type': 'application/json'});
  res.end(JSON.stringify({service: 'worker', status: 'running'}));
});
server.listen(3000, '0.0.0.0');
~~~
Create `/etc/systemd/system/ecommerce-worker.service`:
~~~ini
[Unit]
Description=Ecommerce background worker
After=network.target
[Service]
WorkingDirectory=/opt/ecommerce
ExecStart=/usr/bin/node /opt/ecommerce/worker.js
Restart=always
RestartSec=5
User=www-data
Group=www-data
[Install]
WantedBy=multi-user.target
~~~
Enable it:
~~~bash
sudo mkdir -p /opt/ecommerce
sudo systemctl daemon-reload
sudo systemctl enable --now ecommerce-worker
systemctl status ecommerce-worker --no-pager
~~~
## Step 6 - Install MySQL and Set Up Replication
### Install MySQL on `db01` and `db02`
~~~bash
sudo apt-get update
sudo apt-get install -y mysql-server
sudo systemctl enable --now mysql
~~~
### Primary MySQL config on `db01`
Create `/etc/mysql/mysql.conf.d/mysqld.cnf` with key settings:
~~~ini
[mysqld]
bind-address = 0.0.0.0
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1
max_connections = 500
innodb_buffer_pool_size = 16G
innodb_log_file_size = 1G
relay_log_recovery = ON
~~~
Restart:
~~~bash
sudo systemctl restart mysql
~~~
Create database and replication user:
~~~bash
mysql -uroot <<'SQL'
CREATE DATABASE ecommerce CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'ecomuser'@'192.168.30.%' IDENTIFIED BY 'StrongAppPassword!';
GRANT ALL PRIVILEGES ON ecommerce.* TO 'ecomuser'@'192.168.30.%';
CREATE USER 'repl'@'192.168.30.%' IDENTIFIED BY 'StrongReplPassword!';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'192.168.30.%';
FLUSH PRIVILEGES;
SHOW MASTER STATUS;
SQL
~~~
### Replica MySQL config on `db02`
~~~ini
[mysqld]
bind-address = 0.0.0.0
server-id = 2
relay_log = /var/log/mysql/mysql-relay-bin
read_only = ON
super_read_only = ON
binlog_format = ROW
innodb_buffer_pool_size = 16G
~~~
Restart:
~~~bash
sudo systemctl restart mysql
~~~
### Seed and start replication
On `db01`:
~~~bash
mysqldump -uroot --single-transaction --routines --triggers ecommerce > ecommerce.sql
scp ecommerce.sql db02:/root/
~~~
On `db02`:
~~~bash
mysql -uroot ecommerce < /root/ecommerce.sql
mysql -uroot <<'SQL'
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='192.168.30.31',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='StrongReplPassword!',
  SOURCE_LOG_FILE='mysql-bin.000001',
  SOURCE_LOG_POS=157;
START REPLICA;
SHOW REPLICA STATUS\G
SQL
~~~
Replication checklist:
- `Replica_IO_Running: Yes`
- `Replica_SQL_Running: Yes`
- `Seconds_Behind_Source` near zero
## Step 7 - Install Redis on `app01`
~~~bash
sudo apt-get update
sudo apt-get install -y redis-server
~~~
Edit `/etc/redis/redis.conf`:
~~~ini
bind 0.0.0.0
protected-mode yes
port 6379
timeout 0
tcp-keepalive 300
supervised systemd
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
maxmemory 2gb
maxmemory-policy allkeys-lru
~~~
Restart and test:
~~~bash
sudo systemctl enable --now redis-server
redis-cli -h 127.0.0.1 ping
~~~
Application tip:
- use Redis for sessions, carts, rate limits, cache tags, and queue state, but not as the only persistent system of record.
## Step 8 - Configure HAProxy for Load Balancing
Install on `web01`:
~~~bash
sudo apt-get install -y haproxy
~~~
Create `/etc/haproxy/haproxy.cfg`:
~~~haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    user haproxy
    group haproxy
    daemon
    maxconn 5000
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5s
    timeout client  30s
    timeout server  30s
    retries 3
frontend fe_http
    bind *:80
    http-request set-header X-Forwarded-Proto http
    default_backend be_web
backend be_web
    balance roundrobin
    option httpchk GET /healthz
    http-check expect status 200
    server web01 192.168.10.11:80 check
    server web02 192.168.10.12:80 check
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
~~~
Enable it:
~~~bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl enable --now haproxy
curl http://127.0.0.1:8404/stats
~~~
## Step 9 - Deploy the E-Commerce Application
This example uses a PHP-style application layout, but the same VM pattern works for Magento, Laravel, Shopware, or a custom PHP storefront backed by Node workers.
### Prepare code on app nodes
~~~bash
sudo mkdir -p /var/www/ecommerce
sudo chown -R $USER:www-data /var/www/ecommerce
cd /var/www/ecommerce
git clone https://github.com/laravel/laravel.git .
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate
~~~
### Example application environment file
~~~dotenv
APP_NAME=Ecommerce
APP_ENV=production
APP_DEBUG=false
APP_URL=https://shop.example.internal
DB_CONNECTION=mysql
DB_HOST=192.168.30.31
DB_PORT=3306
DB_DATABASE=ecommerce
DB_USERNAME=ecomuser
DB_PASSWORD=StrongAppPassword!
CACHE_STORE=redis
REDIS_HOST=192.168.30.21
REDIS_PASSWORD=null
REDIS_PORT=6379
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
~~~
### Run migrations and seed sample data
~~~bash
php artisan migrate --force
php artisan db:seed --force
php artisan config:cache
php artisan route:cache
~~~
### Share code to web nodes
If web nodes serve the public directory directly, use `rsync`:
~~~bash
rsync -avz --delete /var/www/ecommerce/ web01:/var/www/ecommerce/
rsync -avz --delete /var/www/ecommerce/ web02:/var/www/ecommerce/
~~~
## Step 10 - SSL/TLS Setup
For an internet-facing test system with public DNS:
~~~bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d shop.example.internal --redirect --agree-tos -m admin@example.internal
~~~
For an internal-only lab, use a self-signed certificate:
~~~bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout /etc/ssl/private/ecommerce.key \
  -out /etc/ssl/certs/ecommerce.crt \
  -subj "/CN=shop.example.internal"
~~~
Add HTTPS server block to Nginx:
~~~nginx
server {
    listen 443 ssl http2;
    server_name shop.example.internal;
    ssl_certificate /etc/ssl/certs/ecommerce.crt;
    ssl_certificate_key /etc/ssl/private/ecommerce.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    root /var/www/ecommerce/public;
    index index.php index.html;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass php_backend;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
~~~
Validate:
~~~bash
sudo nginx -t
sudo systemctl reload nginx
curl -k https://shop.example.internal/healthz
~~~
## Verification Checklist
Run these checks after deployment:
~~~bash
curl -I http://192.168.10.11/healthz
curl -I http://192.168.10.12/healthz
curl -I http://shop.example.internal/healthz
mysql -h 192.168.30.31 -u ecomuser -p'StrongAppPassword!' -e 'SHOW DATABASES;'
redis-cli -h 192.168.30.21 ping
~~~
Application-level checks:
- create a test product
- browse the catalog through HAProxy
- add an item to cart
- verify Redis keys are created
- check a database write on `db01`
- confirm replicated data appears on `db02`
## Automation with Vagrant
Vagrant is useful for repeatable local labs. The following example assumes the `libvirt` provider.
### `Vagrantfile`
~~~ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2404"
  config.vm.synced_folder ".", "/vagrant", disabled: false
  networks = {
    "frontend" => {ip_base: "192.168.10."},
    "backend" => {ip_base: "192.168.20."},
    "database" => {ip_base: "192.168.30."}
  }
  {
    "web01" => {memory: 4096, cpus: 2, nics: [["frontend", 11], ["backend", 11]]},
    "web02" => {memory: 4096, cpus: 2, nics: [["frontend", 12], ["backend", 12]]},
    "app01" => {memory: 8192, cpus: 4, nics: [["backend", 21], ["database", 21]]},
    "app02" => {memory: 8192, cpus: 4, nics: [["backend", 22], ["database", 22]]},
    "db01"  => {memory: 32768, cpus: 8, nics: [["database", 31]]},
    "db02"  => {memory: 32768, cpus: 8, nics: [["database", 32]]}
  }.each do |name, settings|
    config.vm.define name do |node|
      node.vm.hostname = name
      settings[:nics].each do |network_name, host_octet|
        node.vm.network "private_network", ip: "#{networks[network_name][:ip_base]}#{host_octet}"
      end
      node.vm.provider :libvirt do |lv|
        lv.memory = settings[:memory]
        lv.cpus = settings[:cpus]
      end
      node.vm.provision "shell", path: "scripts/common.sh"
      node.vm.provision "shell", path: "scripts/#{name}.sh", privileged: true
    end
  end
end
~~~
### `scripts/common.sh`
~~~bash
#!/usr/bin/env bash
set -euo pipefail
apt-get update
apt-get install -y curl vim git unzip ca-certificates qemu-guest-agent
systemctl enable --now qemu-guest-agent
~~~
### `scripts/web01.sh`
~~~bash
#!/usr/bin/env bash
set -euo pipefail
apt-get install -y nginx haproxy
systemctl enable --now nginx haproxy
~~~
### `scripts/app01.sh`
~~~bash
#!/usr/bin/env bash
set -euo pipefail
apt-get install -y php-fpm php-mysql redis-server
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt-get install -y nodejs
systemctl enable --now php8.3-fpm redis-server
~~~
### `scripts/db01.sh`
~~~bash
#!/usr/bin/env bash
set -euo pipefail
apt-get install -y mysql-server
systemctl enable --now mysql
~~~
Bring it up:
~~~bash
vagrant up --provider=libvirt
vagrant status
vagrant ssh web01
~~~
## VM Snapshots and Rollback
Take pre-deployment snapshots before major changes.
~~~bash
virsh snapshot-create-as web01 pre-app-release "Before release 2025-01"
virsh snapshot-create-as app01 pre-app-release "Before release 2025-01"
virsh snapshot-create-as db01 pre-schema-change "Before migration"
virsh snapshot-list db01
~~~
Rollback procedure:
1. put HAProxy into maintenance or drain the affected node
2. stop application writes if rolling back the database tier
3. revert the VM snapshot
4. validate service health
5. re-enable traffic only after app and DB consistency checks
Example rollback:
~~~bash
virsh shutdown app01
virsh snapshot-revert app01 pre-app-release
virsh start app01
~~~
Warning:
- never revert a database snapshot casually when newer application writes already exist elsewhere
- for MySQL, coordinated backup and restore is safer than ad hoc snapshot reversal
## Live Migration
Live migration requires shared storage or block migration and compatible CPU models.
Example migration from `kvm-host1` to `kvm-host2`:
~~~bash
virsh migrate --live --persistent --undefinesource web02 qemu+ssh://kvm-host2/system
~~~
With non-shared storage block migration:
~~~bash
virsh migrate --live --persistent --undefinesource --copy-storage-all web02 qemu+ssh://kvm-host2/system
~~~
Validation:
~~~bash
virsh dominfo web02
ssh kvm-host2 virsh list --all
~~~
Common pitfalls:
- CPU incompatibility between hosts
- missing bridge names or network definitions on the destination host
- inadequate bandwidth during block migration
- firewalls blocking migration traffic
## Common Pitfalls
- using one NIC/network for all traffic classes
- exposing MySQL directly to the frontend network
- letting web nodes talk directly to the database when app nodes should mediate writes
- forgetting TLS between users and the edge proxy
- assuming Redis persistence is a substitute for database durability
- ignoring MySQL binary log retention and disk growth
[← Back to Virtual Setup](./README.md)
