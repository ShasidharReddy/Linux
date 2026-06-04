# 09 Architecture Diagrams

Complete Visual Architecture Reference for Physical Ecommerce Setup

This guide is diagram-first by design. It complements the physical setup documents with detailed visual references for topology, request flow, monitoring, security, and recovery.
Each Mermaid diagram is intentionally large so teams can use it during planning, reviews, implementations, and operations runbooks without needing to redraw the system from scratch.

---

## Diagram 1 — Complete Physical Data Center Layout

This diagram shows a rack-aware physical ecommerce deployment with clear separation between the DMZ, web, application, database, storage, and management planes.
It maps VLANs, IP ranges, switch uplinks, and out-of-band access so operators can understand both service traffic and operational traffic in one view.

**When to use:** Use this for a production-ready bare-metal deployment that needs strong tier isolation, dual-homed switching, and explicit management network planning.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_clients[Internet Users] --> isp_edge_a[ISP Edge A 203.0.113.2/30]
    internet_clients --> isp_edge_b[ISP Edge B 198.51.100.2/30]
    isp_edge_a --> fw_pair[Perimeter Firewall Pair<br/>203.0.113.10 / 198.51.100.10]
    isp_edge_b --> fw_pair
    fw_pair --> core_pair[Core Switch Stack<br/>L3 Gateway 10.10.0.1]
    core_pair --> mgmt_switch[Mgmt Switch VLAN50 10.10.50.0/24]
    core_pair --> san_switch[Storage Fabric FC/iSCSI]

    subgraph dmz_zone[DMZ Rack - VLAN10 10.10.10.0/24]
        waf_a[WAF-A 10.10.10.11]
        waf_b[WAF-B 10.10.10.12]
        lb_a[LB-A HAProxy 10.10.10.21]
        lb_b[LB-B HAProxy 10.10.10.22]
        dmz_sw_a[Leaf-DMZ-A]
        dmz_sw_b[Leaf-DMZ-B]
    end

    subgraph web_zone[Web Rack - VLAN20 10.10.20.0/24]
        web_01[Web-01 Nginx 10.10.20.11]
        web_02[Web-02 Nginx 10.10.20.12]
        web_03[Web-03 Static/Canary 10.10.20.13]
        web_sw_a[Leaf-Web-A]
        web_sw_b[Leaf-Web-B]
    end

    subgraph app_zone[Application Rack - VLAN30 10.10.30.0/24]
        app_01[App-01 PHP-FPM 10.10.30.11]
        app_02[App-02 PHP-FPM 10.10.30.12]
        app_03[Worker-01 Queue 10.10.30.21]
        app_04[Worker-02 Queue 10.10.30.22]
        app_sw_a[Leaf-App-A]
        app_sw_b[Leaf-App-B]
    end

    subgraph db_zone[Database Rack - VLAN40 10.10.40.0/24]
        db_primary[MySQL Primary 10.10.40.11]
        db_replica[MySQL Replica 10.10.40.12]
        redis_cache[Redis Cache 10.10.40.21]
        proxysql_db[ProxySQL 10.10.40.31]
        db_sw_a[Leaf-DB-A]
        db_sw_b[Leaf-DB-B]
    end

    subgraph storage_zone[Storage Rack]
        nas_share[NAS Shared Uploads 10.10.60.20]
        san_array[SAN Array DB LUNs 10.10.60.30]
        backup_target[Backup Appliance 10.10.60.40]
        storage_sw_a[Leaf-Storage-A]
        storage_sw_b[Leaf-Storage-B]
    end

    subgraph mgmt_zone[Management VLAN50 10.10.50.0/24]
        bastion[Bastion Host 10.10.50.10]
        monitor[Prometheus/Grafana 10.10.50.20]
        ansible[Config Server 10.10.50.30]
        idrac_web01[iLO Web-01 10.10.50.111]
        idrac_web02[iLO Web-02 10.10.50.112]
        idrac_app01[iLO App-01 10.10.50.121]
        idrac_app02[iLO App-02 10.10.50.122]
        idrac_db01[iLO DB-01 10.10.50.131]
        idrac_db02[iLO DB-02 10.10.50.132]
    end

    core_pair --> dmz_sw_a
    core_pair --> dmz_sw_b
    core_pair --> web_sw_a
    core_pair --> web_sw_b
    core_pair --> app_sw_a
    core_pair --> app_sw_b
    core_pair --> db_sw_a
    core_pair --> db_sw_b
    core_pair --> storage_sw_a
    core_pair --> storage_sw_b

    fw_pair --> waf_a
    fw_pair --> waf_b
    waf_a --> lb_a
    waf_b --> lb_b
    lb_a --> web_01
    lb_a --> web_02
    lb_b --> web_02
    lb_b --> web_03
    web_01 --> app_01
    web_01 --> app_02
    web_02 --> app_01
    web_02 --> app_02
    web_03 --> app_02
    app_01 --> proxysql_db
    app_02 --> proxysql_db
    app_03 -.->|async jobs| redis_cache
    app_04 -.->|async jobs| redis_cache
    proxysql_db --> db_primary
    proxysql_db --> db_replica
    db_primary ==> san_array
    db_replica ==> san_array
    web_01 ==> nas_share
    web_02 ==> nas_share
    app_01 ==> nas_share
    app_02 ==> nas_share
    backup_target ==> nas_share
    backup_target ==> san_array
    mgmt_switch --> bastion
    mgmt_switch --> monitor
    mgmt_switch --> ansible
    mgmt_switch --> idrac_web01
    mgmt_switch --> idrac_web02
    mgmt_switch --> idrac_app01
    mgmt_switch --> idrac_app02
    mgmt_switch --> idrac_db01
    mgmt_switch --> idrac_db02
```

**Key takeaways**
- Traffic and operations are cleanly separated by VLANs so troubleshooting does not disrupt production data paths.
- Every critical tier is dual-homed to switching and has at least two compute nodes except for optional auxiliary services.
- The management network is isolated from customer traffic and terminates through bastion plus iLO/IPMI access.

## Diagram 2 — Network Architecture (Spine-Leaf)

This view focuses on switching, routing, and out-of-band control for a medium-sized ecommerce data center using a spine-leaf fabric.
It highlights redundant uplinks, ISP diversity, firewall insertion points, and the difference between in-band production traffic and OOB management traffic.

**When to use:** Use this when the infrastructure is beyond a single rack and you need predictable east-west performance, rapid failure domains, and simpler scale-out networking.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    isp1[ISP-1 ASN 64510] --> edge_rtr1[Edge Router-1 BGP]
    isp2[ISP-2 ASN 64520] --> edge_rtr2[Edge Router-2 BGP]
    edge_rtr1 --> fw_cluster[Firewall Cluster<br/>BGP northbound / OSPF southbound]
    edge_rtr2 --> fw_cluster
    fw_cluster --> spine_01[Spine-01<br/>OSPF Area 0]
    fw_cluster --> spine_02[Spine-02<br/>OSPF Area 0]

    subgraph fabric[Leaf / ToR Fabric]
        leaf_dmz_a[Leaf-DMZ-A]
        leaf_dmz_b[Leaf-DMZ-B]
        leaf_web_a[Leaf-Web-A]
        leaf_web_b[Leaf-Web-B]
        leaf_app_a[Leaf-App-A]
        leaf_app_b[Leaf-App-B]
        leaf_db_a[Leaf-DB-A]
        leaf_db_b[Leaf-DB-B]
        leaf_storage[Leaf-Storage]
        leaf_backup[Leaf-Backup]
    end

    subgraph oob[Out-of-Band Network]
        mgmt_gateway[Mgmt Gateway 10.10.50.1]
        oob_switch[OOB Management Switch]
        ilo_fw[FW iLO/IPMI]
        ilo_web[Web iLO/IPMI]
        ilo_app[App iLO/IPMI]
        ilo_db[DB iLO/IPMI]
        serial_console[Serial Console Server]
    end

    subgraph racks[Rack Endpoints]
        tor_dmz_1[ToR-DMZ-1]
        tor_web_1[ToR-WEB-1]
        tor_app_1[ToR-APP-1]
        tor_db_1[ToR-DB-1]
        tor_storage_1[ToR-STO-1]
        fw_nodes[Firewall Appliances]
        lb_nodes[HAProxy Appliances]
        web_srv_1[Web-01]
        web_srv_2[Web-02]
        app_srv_1[App-01]
        app_srv_2[App-02]
        app_srv_3[Worker-01]
        db_srv_1[DB-Primary]
        db_srv_2[DB-Replica]
        backup_srv[Backup Server]
        san_heads[SAN Heads]
    end

    spine_01 --> leaf_dmz_a
    spine_01 --> leaf_web_a
    spine_01 --> leaf_app_a
    spine_01 --> leaf_db_a
    spine_01 --> leaf_storage
    spine_01 --> leaf_backup
    spine_02 --> leaf_dmz_b
    spine_02 --> leaf_web_b
    spine_02 --> leaf_app_b
    spine_02 --> leaf_db_b
    spine_02 --> leaf_storage
    spine_02 --> leaf_backup

    leaf_dmz_a --> tor_dmz_1
    leaf_dmz_b --> tor_dmz_1
    leaf_web_a --> tor_web_1
    leaf_web_b --> tor_web_1
    leaf_app_a --> tor_app_1
    leaf_app_b --> tor_app_1
    leaf_db_a --> tor_db_1
    leaf_db_b --> tor_db_1
    leaf_storage --> tor_storage_1
    leaf_backup --> backup_srv

    tor_dmz_1 --> fw_nodes
    fw_nodes --> lb_nodes
    tor_web_1 --> web_srv_1
    tor_web_1 --> web_srv_2
    tor_app_1 --> app_srv_1
    tor_app_1 --> app_srv_2
    tor_app_1 --> app_srv_3
    tor_db_1 --> db_srv_1
    tor_db_1 --> db_srv_2
    tor_storage_1 --> san_heads

    mgmt_gateway --> oob_switch
    oob_switch --> ilo_fw
    oob_switch --> ilo_web
    oob_switch --> ilo_app
    oob_switch --> ilo_db
    oob_switch --> serial_console

    fw_nodes -.->|OOB| ilo_fw
    web_srv_1 -.->|iDRAC| ilo_web
    web_srv_2 -.->|iDRAC| ilo_web
    app_srv_1 -.->|iDRAC| ilo_app
    app_srv_2 -.->|iDRAC| ilo_app
    db_srv_1 -.->|iLO| ilo_db
    db_srv_2 -.->|iLO| ilo_db
    edge_rtr1 -.->|eBGP failover| edge_rtr2
    spine_01 -.->|ECMP fabric| spine_02
    db_srv_1 ==> san_heads
    db_srv_2 ==> san_heads
```

**Key takeaways**
- BGP is used at the ISP edge while OSPF or ECMP inside the fabric keeps internal routing deterministic.
- Each rack tier has redundant uplinks to both spines, eliminating single-switch dependency for east-west traffic.
- The OOB network remains separate so failed production switching does not block recovery operations.

## Diagram 3 — Single Server Architecture (Basic)

This is the smallest realistic ecommerce footprint: one physical server running the full stack with local redundancy where possible.
It shows the request path, local service boundaries, port usage, and how storage is carved for OS, application data, and backups.

**When to use:** Use this for labs, pilots, or very small stores where budget matters more than horizontal scale and planned downtime is acceptable.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph LR
    shopper[Client Browser / Mobile App] --> dns_resolver[Authoritative DNS 53]
    dns_resolver --> host_public[Single Host 203.0.113.50]
    host_public --> host_fw[Host Firewall nftables]

    subgraph nginx_stack[Nginx Tier]
        nginx_listener[Nginx :443 SSL Termination]
        nginx_static[Static Files /assets]
        nginx_gzip[Gzip + Brotli Compression]
        nginx_cache[Microcache 1-5s]
        nginx_logs[Access/Error Logs]
    end

    subgraph app_stack[PHP Runtime]
        php_fpm[PHP-FPM :9000]
        php_pool[www pool pm.dynamic]
        php_opcache[OPcache shared memory]
        composer_vendor[Application Code /var/www/app]
        cron_jobs[Cron + Queue Workers]
    end

    subgraph data_stack[Local Data Services]
        mysql_local[MySQL InnoDB :3306]
        mysql_redo[Redo Logs / ib_logfile]
        redis_local[Redis :6379<br/>sessions + cache]
        backup_agent[Backup Agent rsync + mysqldump]
    end

    subgraph storage_stack[Local Disk Layout]
        os_raid[RAID-1 SSDs /]
        data_raid[RAID-10 NVMe /srv/data]
        upload_fs[Uploads /srv/data/uploads]
        mysql_fs[MySQL datadir /srv/data/mysql]
        backup_fs[Backups /srv/backup]
    end

    host_fw --> nginx_listener
    nginx_listener -->|443/HTTPS| nginx_static
    nginx_listener --> nginx_gzip
    nginx_listener --> nginx_cache
    nginx_listener -->|FastCGI 127.0.0.1:9000| php_fpm
    php_fpm --> php_pool
    php_pool --> php_opcache
    php_pool --> composer_vendor
    php_pool -->|TCP 127.0.0.1:3306| mysql_local
    php_pool -->|TCP 127.0.0.1:6379| redis_local
    cron_jobs -.->|async queue| php_pool
    mysql_local --> mysql_redo
    mysql_local ==> mysql_fs
    redis_local ==> data_raid
    nginx_static ==> upload_fs
    composer_vendor ==> os_raid
    mysql_fs ==> data_raid
    upload_fs ==> data_raid
    backup_agent ==> backup_fs
    backup_agent -.->|off-host rsync 22| remote_target[Remote Backup Host]
    nginx_logs ==> os_raid
```

**Key takeaways**
- Nginx terminates TLS, serves static assets, and forwards only dynamic traffic to PHP-FPM on port 9000.
- MySQL and Redis remain local, so latency is low but maintenance or host failure impacts the entire platform.
- RAID-1 protects the OS while RAID-10 is reserved for transactional data and uploads.

## Diagram 4 — Multi-Tier Architecture (Intermediate)

This diagram represents the common intermediate step where load balancing, web, application, database, cache, and storage roles are separated across hosts.
It shows active/standby ingress, health checks, shared file storage, read/write database routing, and centralized backups.

**When to use:** Use this when a single host is no longer enough but the environment is still small enough to manage with simple failover instead of full multi-site clustering.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_users[Internet Users] --> keepalived_vip[Keepalived VIP 203.0.113.100]

    subgraph ingress_pair[Load Balancer Pair]
        haproxy_active[HAProxy-A Active]
        haproxy_standby[HAProxy-B Standby]
        healthd[VRRP + TCP Health Checks]
    end

    subgraph web_cluster[Web Tier VLAN20]
        web01[Nginx-Web-01]
        web02[Nginx-Web-02]
        web_health[HTTP /healthz checks]
        upload_mount[Shared Upload Mount /mnt/uploads]
    end

    subgraph app_cluster[App Tier VLAN30]
        app01[PHP-FPM-App-01]
        app02[PHP-FPM-App-02]
        worker01[Queue-Worker-01]
        worker02[Queue-Worker-02]
        session_mode[Session Handler Redis]
    end

    subgraph data_services[Data Tier VLAN40]
        proxysql_rw[ProxySQL RW Endpoint]
        proxysql_ro[ProxySQL RO Endpoint]
        mysql_pri[MySQL Primary]
        mysql_rep[MySQL Replica]
        redis_master[Redis Master]
        redis_replica[Redis Replica]
    end

    subgraph shared_services[Shared Infrastructure]
        nfs_server[NFS Shared Storage]
        backup_server[Backup Server rsync]
        monitor_stack[Monitoring Host]
        deploy_host[Ansible Deploy Host]
    end

    keepalived_vip --> haproxy_active
    keepalived_vip -.-> haproxy_standby
    haproxy_active -->|HTTP health checks| web_health
    haproxy_active --> web01
    haproxy_active --> web02
    haproxy_standby -.->|state sync| haproxy_active
    healthd --> haproxy_active
    healthd --> haproxy_standby
    web01 -->|FastCGI 9000| app01
    web01 -->|FastCGI 9000| app02
    web02 -->|FastCGI 9000| app01
    web02 -->|FastCGI 9000| app02
    app01 -->|sessions 6379| redis_master
    app02 -->|sessions 6379| redis_master
    worker01 -.->|jobs| redis_master
    worker02 -.->|jobs| redis_master
    app01 -->|writes 6033| proxysql_rw
    app02 -->|reads 6032| proxysql_ro
    proxysql_rw --> mysql_pri
    proxysql_ro --> mysql_rep
    mysql_pri ==> mysql_rep
    redis_master ==> redis_replica
    web01 ==> upload_mount
    web02 ==> upload_mount
    app01 ==> upload_mount
    app02 ==> upload_mount
    upload_mount ==> nfs_server
    backup_server ==> nfs_server
    backup_server ==> mysql_pri
    backup_server ==> mysql_rep
    monitor_stack -.->|scrape/exporters| web01
    monitor_stack -.->|scrape/exporters| web02
    monitor_stack -.->|scrape/exporters| app01
    monitor_stack -.->|scrape/exporters| app02
    deploy_host -.->|rolling deploy| web01
    deploy_host -.->|rolling deploy| web02
    deploy_host -.->|release| app01
    deploy_host -.->|release| app02
```

**Key takeaways**
- Keepalived provides a stable VIP while HAProxy actively checks web node health before routing traffic.
- Redis centralizes sessions so users can move between web and app nodes without sticky-session dependence.
- ProxySQL splits read and write paths, keeping replicas useful without changing application connection logic.

## Diagram 5 — Enterprise Production Architecture (Advanced)

This is a full production reference for a larger ecommerce platform spanning two data centers with global steering and disaster recovery pathways.
The primary site contains clustered caching, search, queueing, and database layers, while the secondary site receives asynchronous replicas and backup streams.

**When to use:** Use this when downtime costs are high, search and queue workloads are material, and the business needs both local high availability and remote DR coverage.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    gslb[GSLB / GeoDNS] --> dc1_ingress[DC1 Active Site]
    gslb -.->|failover| dc2_ingress[DC2 DR Site]

    subgraph dc1[Data Center 1 - Primary]
        isp_dc1[ISP Edge DC1] --> fw_dc1[Firewall Pair DC1]
        fw_dc1 --> lb_dc1[HAProxy + Keepalived VIP]
        lb_dc1 --> varnish_a[Varnish-01]
        lb_dc1 --> varnish_b[Varnish-02]
        varnish_a --> web_a1[Web-01]
        varnish_a --> web_a2[Web-02]
        varnish_b --> web_a3[Web-03]
        web_a1 --> app_a1[App-01]
        web_a1 --> app_a2[App-02]
        web_a2 --> app_a2
        web_a2 --> app_a3[App-03]
        web_a3 --> app_a1
        web_a3 --> app_a3
        app_a1 --> galera1[Galera-01]
        app_a2 --> galera2[Galera-02]
        app_a3 --> galera3[Galera-03]
        app_a1 --> redis_sentinel1[Redis-Sentinel-01]
        app_a2 --> redis_sentinel2[Redis-Sentinel-02]
        app_a3 --> redis_sentinel3[Redis-Sentinel-03]
        redis_sentinel1 --> redis_master_a[Redis Master]
        redis_sentinel2 --> redis_replica_a1[Redis Replica-1]
        redis_sentinel3 --> redis_replica_a2[Redis Replica-2]
        app_a1 -.->|catalog sync| es1[Elasticsearch-01]
        app_a2 -.->|catalog sync| es2[Elasticsearch-02]
        app_a3 -.->|catalog sync| es3[Elasticsearch-03]
        app_a1 -.->|events| mq1[RabbitMQ-01]
        app_a2 -.->|events| mq2[RabbitMQ-02]
        app_a3 -.->|events| mq3[RabbitMQ-03]
        backup_dc1[Backup Orchestrator]
    end

    subgraph dc2[Data Center 2 - DR]
        isp_dc2[ISP Edge DC2] --> fw_dc2[Firewall Pair DC2]
        fw_dc2 --> lb_dc2[Standby LB VIP]
        lb_dc2 --> web_b1[DR Web-01]
        lb_dc2 --> web_b2[DR Web-02]
        web_b1 --> app_b1[DR App-01]
        web_b2 --> app_b2[DR App-02]
        app_b1 --> mysql_dr[MySQL DR Replica]
        app_b2 --> redis_dr[Redis DR Replica]
        app_b1 -.->|warm index| es_dr1[ES-DR-01]
        app_b2 -.->|warm queue| mq_dr1[RabbitMQ-DR-01]
        dr_backup[DR Backup Vault]
    end

    dc1_ingress --> isp_dc1
    dc2_ingress --> isp_dc2
    galera1 ==> galera2
    galera2 ==> galera3
    redis_master_a ==> redis_replica_a1
    redis_master_a ==> redis_replica_a2
    es1 ==> es2
    es2 ==> es3
    mq1 ==> mq2
    mq2 ==> mq3
    galera3 -.->|async DR replication| mysql_dr
    redis_replica_a2 -.->|replica stream| redis_dr
    es3 -.->|snapshot replication| es_dr1
    mq3 -.->|federation| mq_dr1
    backup_dc1 ==> dr_backup
    backup_dc1 -.->|config sync| web_b1
    backup_dc1 -.->|artifact sync| app_b1
```

**Key takeaways**
- The primary site uses local clustering for availability, while DC2 stays warm with asynchronous copies to avoid cross-site write latency.
- Search, cache, and messaging are first-class tiers because enterprise ecommerce traffic is rarely database-only.
- Global load balancing keeps customer entry simple while enabling DR redirection when site health drops.

## Diagram 6 — Request Flow: User Places an Order

This sequence follows a shopper from the first HTTPS request through order creation, payment authorization, event emission, and customer confirmation.
It makes the synchronous path and the asynchronous side effects visible, which is useful for latency budgeting and troubleshooting checkout defects.

**When to use:** Use this when teams need a transaction-by-transaction view of checkout behavior, especially during performance tuning or failure analysis.

```mermaid
%%{init:{"theme":"neutral"}}%%
sequenceDiagram
    autonumber
    participant user as User Browser
    participant cdn as CDN / Edge Cache
    participant lb as HAProxy VIP
    participant nginx as Nginx Web
    participant php as PHP-FPM App
    participant redis as Redis Session Cache
    participant mysql as MySQL Primary
    participant mq as RabbitMQ
    participant pay as Payment Gateway
    participant email as Email Service
    user->>cdn: GET /checkout + cart cookie
    Note right of cdn: ~10 ms edge lookup
    cdn->>lb: HTTPS origin request
    Note right of lb: ~2-5 ms route decision
    lb->>nginx: Forward request to healthy web node
    nginx->>php: FastCGI /checkout
    Note right of php: app bootstrap + auth
    php->>redis: GET session:{user_id}
    redis-->>php: cart, CSRF token, auth context
    php->>mysql: SELECT inventory, pricing, shipping rules
    Note right of mysql: ~8-15 ms indexed read
    mysql-->>php: inventory available
    php->>mysql: INSERT order + order_items status=pending
    mysql-->>php: order_id=ORD-10452
    php-.->>mq: publish order.created event
    Note right of mq: async fan-out for downstream jobs
    php->>pay: POST authorize payment
    Note right of pay: external API 200-600 ms
    alt payment approved
        pay-->>php: auth_success txn_id=TXN8891
        php->>mysql: UPDATE order status=paid
        php-.->>mq: publish payment.captured
        mq-.->>email: consume order confirmation event
        email-->>user: send email + SMS confirmation
        php-->>nginx: 200 checkout success page
        nginx-->>lb: response with order summary
        lb-->>cdn: cache-bypass dynamic response
        cdn-->>user: Order placed confirmation
    else payment failed
        pay-->>php: decline / timeout
        php->>mysql: UPDATE order status=payment_failed
        php-.->>mq: publish payment.failed
        php-->>nginx: 402/500 retry payment page
        nginx-->>user: display error and retry path
    end
```

**Key takeaways**
- Redis removes repeated session reconstruction from the critical path, while MySQL remains the system of record for inventory and order state.
- RabbitMQ decouples notifications and side effects so the customer does not wait on non-critical post-checkout processing.
- Payment handling dominates checkout latency, so upstream stack tuning helps less than external gateway reliability and timeouts.

## Diagram 7 — Backup and Disaster Recovery Architecture

This backup map shows which data types use logical dumps, hot backups, or file-level syncs and how they move from production to local and remote retention tiers.
RPO and RTO goals are attached to the backup chain so infrastructure choices stay tied to recovery outcomes rather than copy mechanics alone.

**When to use:** Use this when building restore runbooks, proving compliance, or validating whether the backup design actually supports the business recovery objectives.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph production[Production Estate]
        prod_mysql[MySQL Primary + Replica]
        prod_uploads[Uploads / Media]
        prod_configs[Configs / Playbooks]
        prod_search[Elasticsearch Indexes]
        prod_logs[Audit Logs]
    end

    subgraph methods[Backup Methods]
        mysqldump_job[mysqldump every 4h<br/>RPO 4h]
        xtrabackup_job[XtraBackup nightly<br/>RPO 24h baseline + binlogs]
        binlog_ship[Binlog Shipping every 5m<br/>RPO 5m]
        rsync_job[rsync uploads nightly<br/>RPO 24h]
        config_git[Git + etckeeper on change<br/>RPO <1h]
        es_snapshot[ES snapshot nightly<br/>RPO 24h]
    end

    subgraph local_backup[Local Backup Server]
        backup_repo[Backup Repository RAID-6]
        verify_host[Restore Verify Host]
        retention_short[7-30 day retention]
    end

    subgraph remote_copy[Remote Backup at DC2]
        remote_vault[Remote Vault Immutable]
        retention_long[30-180 day retention]
        dr_restore[DR Restore Staging]
    end

    subgraph cold_storage[Cold Archive]
        tape_lib[Tape Library / WORM]
        object_archive[Offline Object Archive]
        legal_hold[Quarterly Legal Hold]
    end

    prod_mysql --> mysqldump_job
    prod_mysql --> xtrabackup_job
    prod_mysql -.->|continuous| binlog_ship
    prod_uploads --> rsync_job
    prod_configs --> config_git
    prod_search --> es_snapshot
    prod_logs --> rsync_job
    mysqldump_job ==> backup_repo
    xtrabackup_job ==> backup_repo
    binlog_ship ==> backup_repo
    rsync_job ==> backup_repo
    config_git ==> backup_repo
    es_snapshot ==> backup_repo
    backup_repo --> verify_host
    backup_repo --> retention_short
    verify_host -.->|weekly restore drill / RTO target 2h| backup_repo
    backup_repo ==> remote_vault
    remote_vault --> retention_long
    remote_vault --> dr_restore
    dr_restore -.->|monthly failover test / RTO target 4h| remote_vault
    remote_vault ==> tape_lib
    remote_vault ==> object_archive
    tape_lib --> legal_hold
    object_archive --> legal_hold
```

**Key takeaways**
- Hot database backups and binlog shipping are what shorten RPO; nightly file copies alone do not support tight recovery targets.
- Restore verification hosts are mandatory because backup success without restore success is not meaningful protection.
- Cold archive tiers protect against ransomware, operator error, and long-tail compliance retrieval needs.

## Diagram 8 — Monitoring Stack

This observability diagram ties infrastructure metrics, application metrics, centralized logging, and alert routing into one operating picture.
It also shows which exporters and collectors sit close to workloads versus which systems aggregate data for dashboards and incidents.

**When to use:** Use this for any environment where operations need fast correlation between node health, application latency, queue depth, logs, and business alerts.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph estate[Observed Systems]
        web_nodes[Web Servers]
        app_nodes[App Servers]
        db_nodes[MySQL Nodes]
        cache_nodes[Redis Nodes]
        mq_nodes[RabbitMQ Nodes]
        search_nodes[Elasticsearch Nodes]
        fw_nodes_obs[Firewall / LB Appliances]
    end

    subgraph metrics[Metrics Collection]
        node_exporter[node_exporter CPU RAM disk NIC]
        mysql_exporter[mysqld_exporter queries lag locks]
        redis_exporter[redis_exporter hits memory evictions]
        rabbit_exporter[rabbitmq_exporter queue depth consumers]
        nginx_exporter[nginx exporter requests latency]
        blackbox_exporter[blackbox_exporter HTTPS /login /checkout]
        prometheus[Prometheus]
        grafana[Grafana Dashboards]
    end

    subgraph logging[Log Pipeline]
        filebeat[Filebeat on servers]
        logstash[Logstash parse + enrich]
        elastic[Elasticsearch logging cluster]
        kibana[Kibana]
    end

    subgraph alerting[Alerting]
        alertmanager[Alertmanager]
        pagerduty[PagerDuty]
        slack[Slack]
        email_ops[Email Ops Inbox]
        sms_oncall[SMS Gateway]
    end

    web_nodes --> node_exporter
    web_nodes --> nginx_exporter
    app_nodes --> node_exporter
    db_nodes --> node_exporter
    db_nodes --> mysql_exporter
    cache_nodes --> redis_exporter
    mq_nodes --> rabbit_exporter
    search_nodes --> node_exporter
    fw_nodes_obs --> blackbox_exporter
    node_exporter --> prometheus
    mysql_exporter --> prometheus
    redis_exporter --> prometheus
    rabbit_exporter --> prometheus
    nginx_exporter --> prometheus
    blackbox_exporter --> prometheus
    prometheus --> grafana
    prometheus --> alertmanager
    alertmanager --> pagerduty
    alertmanager --> slack
    alertmanager --> email_ops
    alertmanager --> sms_oncall
    web_nodes --> filebeat
    app_nodes --> filebeat
    db_nodes --> filebeat
    cache_nodes --> filebeat
    mq_nodes --> filebeat
    search_nodes --> filebeat
    fw_nodes_obs --> filebeat
    filebeat --> logstash
    logstash --> elastic
    elastic --> kibana
```

**Key takeaways**
- Metrics, logs, and synthetic checks cover different failure modes and must be reviewed together during incidents.
- Prometheus and Alertmanager handle thresholding, while ELK answers the detailed why behind those thresholds.
- Exporter choice should match the tier so alerts carry real operational context such as replication lag or queue backlog.

## Diagram 9 — Security Zones and Firewall Rules

This security model divides the platform into explicit trust zones and annotates the only traffic paths that should exist between them.
It is useful for turning architecture intent into concrete firewall objects, iptables chains, and audit conversations.

**When to use:** Use this when you need defendable segmentation boundaries, especially for PCI-sensitive checkout traffic and restricted management access.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    internet_zone[Internet Zone 0.0.0.0/0] -->|allow 80,443| dmz_zone_sec[DMZ Zone VLAN10]
    internet_zone -.->|deny all other inbound| deny_internet[DROP by fw_internet]

    subgraph dmz_block[DMZ Zone]
        waf_rules[WAF OWASP CRS]
        lb_vip_sec[LB VIP 443]
        fw_dmz[iptables chain DMZ_IN]
    end

    subgraph web_block[Web Zone VLAN20]
        web_nodes_sec[Nginx Web Servers :8080]
        fw_web[iptables chain WEB_IN]
    end

    subgraph app_block[App Zone VLAN30]
        app_nodes_sec[PHP-FPM App Servers :9000]
        worker_nodes_sec[Queue Workers]
        fw_app[iptables chain APP_IN]
    end

    subgraph db_block[DB Zone VLAN40]
        mysql_zone[MySQL :3306]
        redis_zone[Redis :6379]
        mq_zone[RabbitMQ :5672]
        fw_db[iptables chain DB_IN]
    end

    subgraph mgmt_block[Management Zone VLAN50]
        bastion_zone[Bastion :22]
        monitor_zone[Monitoring :9090/:3000]
        ipmi_zone[IPMI / iLO :443 / 623]
        fw_mgmt[iptables chain MGMT_IN]
    end

    dmz_zone_sec --> waf_rules
    waf_rules --> lb_vip_sec
    lb_vip_sec -->|allow 8080 only| web_nodes_sec
    lb_vip_sec -.->|deny 22,3306,6379| deny_web_misc[DROP by fw_dmz]
    web_nodes_sec -->|allow 9000 FastCGI| app_nodes_sec
    web_nodes_sec -.->|deny direct DB access| deny_web_db[DROP by fw_web]
    app_nodes_sec -->|allow 3306| mysql_zone
    app_nodes_sec -->|allow 6379| redis_zone
    worker_nodes_sec -->|allow 5672| mq_zone
    app_nodes_sec -.->|deny internet egress except payment API| deny_app_egress[REJECT by fw_app]
    bastion_zone -->|allow 22 SSH admin| web_nodes_sec
    bastion_zone -->|allow 22 SSH admin| app_nodes_sec
    bastion_zone -->|allow 22 SSH admin| mysql_zone
    monitor_zone -->|allow scrape 9100/9113/9121| web_nodes_sec
    monitor_zone -->|allow scrape 9100/9187| mysql_zone
    ipmi_zone -->|allow OOB 443/623| web_nodes_sec
    ipmi_zone -->|allow OOB 443/623| app_nodes_sec
    ipmi_zone -->|allow OOB 443/623| mysql_zone
    fw_dmz -.-> fw_web
    fw_web -.-> fw_app
    fw_app -.-> fw_db
    fw_mgmt -.-> fw_web
    fw_mgmt -.-> fw_app
    fw_mgmt -.-> fw_db
```

**Key takeaways**
- Only the DMZ should face the internet, and each deeper tier should expose fewer protocols than the one before it.
- Bastion-based administration plus explicit monitoring paths reduces the temptation to leave broad SSH access open.
- Firewall chains should mirror architectural zones so audits and rule reviews remain understandable.

## Diagram 10 — Storage Architecture

This storage view shows how ecommerce workloads map different durability and performance needs onto local RAID, NAS, SAN, and distributed storage platforms.
It also annotates rough IOPS and throughput expectations so storage design aligns with application behavior rather than generic capacity planning.

**When to use:** Use this when sizing disks for transactional databases, shared media, backups, and future distributed storage expansion.

```mermaid
%%{init:{"theme":"neutral"}}%%
graph TB
    subgraph compute_storage[Per-Server Local Storage]
        web_os[Web OS RAID-1 SSD<br/>80k read IOPS]
        web_data[Web Cache RAID-10 SSD<br/>120k read IOPS]
        app_os[App OS RAID-1 SSD<br/>60k read IOPS]
        app_data[App Temp RAID-10 NVMe<br/>150k mixed IOPS]
        db_os[DB OS RAID-1 SSD<br/>40k read IOPS]
        db_data[DB Data RAID-10 NVMe<br/>250k write-optimized IOPS]
        backup_local[Backup Server RAID-6 NL-SAS<br/>1.5 GB/s sequential]
    end

    subgraph shared_file_storage[Shared File Services]
        nas_head[NAS Head Pair<br/>NFS/SMB 5-8 GB/s]
        uploads_share[Product Images + Uploads]
        release_share[Release Artifacts]
        report_share[Exports / Reports]
    end

    subgraph block_storage[Database Block Storage]
        san_fc[SAN FC Fabric 32Gb]
        san_iscsi[iSCSI VLAN 60 25GbE]
        db_lun_primary[Primary DB LUNs]
        db_lun_replica[Replica DB LUNs]
        journal_lun[Low-latency Journal LUN]
    end

    subgraph distributed_storage[Distributed / Scale-Out]
        ceph_mon[Ceph MON x3]
        ceph_mgr[Ceph MGR x2]
        ceph_osd1[Ceph OSD-01]
        ceph_osd2[Ceph OSD-02]
        ceph_osd3[Ceph OSD-03]
        ceph_rgw[Ceph RGW Object API]
    end

    web_os --> web_data
    app_os --> app_data
    db_os --> db_data
    web_data ==> nas_head
    app_data ==> nas_head
    nas_head --> uploads_share
    nas_head --> release_share
    nas_head --> report_share
    db_data ==> san_fc
    db_data ==> san_iscsi
    san_fc --> db_lun_primary
    san_iscsi --> db_lun_replica
    san_fc --> journal_lun
    backup_local ==> uploads_share
    backup_local ==> db_lun_primary
    backup_local ==> db_lun_replica
    backup_local ==> ceph_rgw
    ceph_mon --> ceph_mgr
    ceph_mgr --> ceph_osd1
    ceph_mgr --> ceph_osd2
    ceph_mgr --> ceph_osd3
    ceph_osd1 ==> ceph_rgw
    ceph_osd2 ==> ceph_rgw
    ceph_osd3 ==> ceph_rgw
    uploads_share ==> ceph_rgw
```

**Key takeaways**
- Operating system volumes should prioritize simple redundancy, while database and cache volumes should prioritize low latency and write durability.
- NAS suits shared file workloads, SAN suits transactional block storage, and Ceph suits scale-out object or future cloud-like patterns.
- Storage throughput and IOPS assumptions should be written into the architecture so application growth has an explicit capacity baseline.
