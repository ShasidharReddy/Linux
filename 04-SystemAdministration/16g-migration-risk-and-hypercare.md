# Migration Risk, Hypercare, and Handoff

← Back to [16-cloud-migration.md](./16-cloud-migration.md)

Risk controls, anti-patterns, smoke tests, escalation thresholds, and post-hypercare handoff guidance.

---

### Appendix AA: Detailed smoke tests by workload type

#### Web applications

- [ ] Open landing page.
- [ ] Test login.
- [ ] Submit form or key transaction.
- [ ] Validate session persistence.
- [ ] Check file upload or download.
- [ ] Confirm TLS certificate and redirect behavior.
- [ ] Verify error page handling.
- [ ] Review application log for 5xx errors.

#### APIs

- [ ] Call health endpoint.
- [ ] Call authenticated endpoint.
- [ ] Validate latency target.
- [ ] Check rate limiting or auth headers.
- [ ] Confirm downstream dependency response.
- [ ] Inspect structured logs.
- [ ] Verify tracing IDs propagate.
- [ ] Review error budget after first hour.

#### Databases

- [ ] Test local listener.
- [ ] Run connection from app tier.
- [ ] Execute read query.
- [ ] Execute controlled write query.
- [ ] Check backup status.
- [ ] Validate replication or HA status.
- [ ] Confirm maintenance job schedule.
- [ ] Review storage latency metrics.

#### File services

- [ ] Mount share from approved client.
- [ ] Read test file.
- [ ] Write test file.
- [ ] Validate ACL inheritance.
- [ ] Check quota or capacity alert.
- [ ] Review snapshot policy.
- [ ] Confirm backup selection.
- [ ] Validate DNS alias if used.

### Appendix AB: Escalation thresholds during hypercare

#### Sev 1

- Production unavailable to majority of users.
- Rollback decision required immediately.
- Executive update every 15 minutes.

#### Sev 2

- Core business function degraded but workaround exists.
- War room remains active until stabilized.
- Status update every 30 minutes.

#### Sev 3

- Minor function degraded or non-critical error rate elevated.
- Track in stabilization backlog.
- Daily review until closed.

### Appendix AC: Per-provider cost watch items

#### Azure

- Unattached managed disks
- Public IPs left allocated
- Oversized VM families
- Log Analytics ingestion growth
- ExpressRoute circuit usage vs plan
- Snapshot sprawl

#### AWS

- Idle NAT gateways
- EBS gp3/io2 overprovisioning
- Cross-AZ data transfer
- Old snapshots and AMIs
- CloudWatch log retention defaults
- Unattached Elastic IPs

#### GCP

- Idle reserved IPs
- Persistent disks not deleted with instances
- Inter-zone traffic
- High snapshot retention
- Excessive logging volume
- Oversized machine types

### Appendix AD: Reusable migration patterns

- Pilot low-risk internal web app
- Database-adjacent app with retained DB
- VM-first then managed database modernization
- File server with staged cutover
- Hybrid identity dependent workload
- Internet-facing API behind managed WAF
- Batch worker wave using golden image
- DR-first replication before production move

### Appendix AE: Migration anti-patterns to avoid

- Do not migrate undocumented systems just because they are powered on; confirm business need first.
- Do not treat DNS as a last-step activity; resolution paths often determine whether cutover succeeds.
- Do not replicate poor security practices such as broad admin access or hard-coded secrets into the cloud.
- Do not size instances based only on allocated CPU and RAM; use measured utilization.
- Do not skip test failover because production windows are tight; that usually increases production risk.
- Do not assume cloud backup is automatic for every service; verify policy attachment explicitly.
- Do not forget egress cost when workloads keep talking to retained on-premises systems.
- Do not mix unrelated applications into one wave simply because they share an owner.
- Do not close a migration before hypercare confirms normal business usage patterns.
- Do not decommission the source until rollback windows and backup validation are complete.

### Appendix AF: Cloud-specific troubleshooting quick triage

#### Azure triage

- Check VM boot diagnostics and serial console output.
- Confirm NSG rules and effective routes.
- Validate Private DNS zone links and forwarders.
- Review Azure Monitor heartbeat and guest agent status.
- Check ASR recovery point state if rollback discussion begins.
- Validate managed identity token access if app authentication fails.

#### AWS triage

- Review EC2 console output and system status checks.
- Confirm security groups, NACLs, and route tables.
- Check target group health and listener rules.
- Validate IAM instance profile and Systems Manager connectivity.
- Review CloudWatch metrics and recent log streams.
- Confirm EBS attachment order and filesystem mounts.

#### GCP triage

- Inspect serial port output and operations logs.
- Validate firewall rules, routes, and load balancer health checks.
- Check service account scopes and IAM bindings.
- Review Cloud Logging for guest agent or startup-script failures.
- Confirm persistent disk attachment and fstab or mount logic.
- Validate Cloud DNS or forwarding policy behavior.

### Appendix AG: Validation query pack for stateful workloads

- [ ] Verify database listener is up and bound to the correct interface.
- [ ] Confirm application can open new connections from the target subnet.
- [ ] Run row-count or checksum validation on representative tables.
- [ ] Confirm scheduled maintenance jobs are enabled in the target.
- [ ] Review replication lag or high availability status.
- [ ] Validate backup catalog sees the migrated instance.
- [ ] Check storage latency and queue depth during smoke tests.
- [ ] Validate disk free space against growth forecast.
- [ ] Confirm temp storage and transaction log placement.
- [ ] Run a controlled write and verify downstream read path.
- [ ] Check reporting jobs, ETL pipelines, and message consumers.
- [ ] Verify time zone and locale settings if business logic depends on them.
- [ ] Validate certificate store or trust chain for encrypted connections.
- [ ] Check application connection pools for stale endpoints.
- [ ] Confirm secrets rotation path works after migration.
- [ ] Review error logs for retry storms or auth failures.
- [ ] Confirm monitoring dashboards reflect new hostnames or resource IDs.
- [ ] Validate restore point creation after cutover.
- [ ] Document final database version, patch level, and parameter differences.
- [ ] Capture evidence for data integrity sign-off.

### Appendix AH: DNS and traffic cutover checklist pack

#### Before change

- [ ] TTL reduced if policy allows.
- [ ] Current records exported or documented.
- [ ] Health checks validated against cloud targets.
- [ ] Split-horizon zones reviewed.
- [ ] Reverse DNS requirements reviewed.
- [ ] Certificate SANs match target hostnames.

#### During change

- [ ] Authoritative record updated.
- [ ] Load balancer target or backend pool updated.
- [ ] Resolver caches sampled from multiple networks.
- [ ] Synthetic transaction started.
- [ ] Error rate watched during first traffic shift.
- [ ] Rollback timer visible to all teams.

#### After change

- [ ] Client traffic reaches target.
- [ ] Old endpoint traffic drops as expected.
- [ ] No unexpected source geographies fail.
- [ ] Support desk has no surge of auth or timeout tickets.
- [ ] Monitoring labels and dashboards reflect new endpoint.
- [ ] TTL reset if desired after stabilization.

### Appendix AI: Backup and restore verification pack

- [ ] Policy attached on day zero.
- [ ] First scheduled backup completed.
- [ ] Manual on-demand backup tested.
- [ ] Restore initiated to alternate location or isolated network.
- [ ] Restore duration measured.
- [ ] Encryption key access validated.
- [ ] Retention matches policy.
- [ ] Cross-region or vault copy verified if required.
- [ ] Application-consistent backup behavior confirmed.
- [ ] Database transaction log or binlog handling documented.
- [ ] Backup alerts wired to on-call.
- [ ] Expired snapshots cleanup policy reviewed.
- [ ] Recovery documentation updated.
- [ ] Ownership for restore testing assigned.
- [ ] Quarterly recovery test date scheduled.

### Appendix AJ: Security evidence checklist for go-live

#### Identity evidence

- [ ] Privileged groups reviewed.
- [ ] Break-glass account vaulted.
- [ ] MFA control documented.
- [ ] Role assignments exported.
- [ ] Service account ownership recorded.

#### Network evidence

- [ ] Firewall rules approved.
- [ ] Public exposure reviewed.
- [ ] Route changes documented.
- [ ] VPN or circuit health captured.
- [ ] DNS forwarding behavior validated.

#### Logging evidence

- [ ] Audit logs retained.
- [ ] Cloud activity logs enabled.
- [ ] Application logs centralized.
- [ ] Alert routing tested.
- [ ] SIEM ingestion confirmed.

#### Data protection evidence

- [ ] Encryption at rest confirmed.
- [ ] Encryption in transit confirmed.
- [ ] Backup encryption verified.
- [ ] Residency requirement mapped to region.
- [ ] Secrets stored in approved service.

### Appendix AK: Wave closure report template

| Field | Suggested content |
| --- | --- |
| Wave ID | Wave-03-api-services |
| Date and window | 2026-03-14 22:00 to 2026-03-15 02:30 |
| Applications included | Customer API, auth API, scheduler |
| Cloud target | AWS us-east-1 |
| Outcome | Successful with minor post-cutover tuning |
| Incidents | Two Sev3 issues resolved in hypercare |
| Rollback invoked? | No |
| Performance result | Median latency 8 percent better than baseline |
| Cost note | Initial sizing reduced after 7-day telemetry review |
| Lessons learned | Pre-stage security group rules and synthetic tests earlier |

### Appendix AL: Provider selection heuristics

- If the estate is heavily Microsoft-centric with AD, Windows Server, SQL Server, and existing Microsoft licensing strategy, Azure often reduces friction.
- If the program needs the broadest service catalog, deep marketplace ecosystem, and mature multi-account patterns, AWS often fits well.
- If the roadmap strongly emphasizes analytics, Kubernetes, and straightforward VPC constructs, GCP can be compelling.
- If latency to retained on-premises dependencies is dominant, choose the provider region and connectivity option that best matches network reality, not brand preference.
- If skills are scarce, bias toward the cloud where the operating team already has the strongest automation and security competency.

### Appendix AM: Post-hypercare handoff checklist

- [ ] Incident queue returns to normal ownership.
- [ ] Dashboards moved from migration war room to steady-state operations.
- [ ] Runbooks published in the operational knowledge base.
- [ ] Known issues backlog assigned to product or platform team.
- [ ] Cost optimization tasks logged for follow-up.
- [ ] Source decommission plan scheduled.
- [ ] Final sign-off from application owner captured.
- [ ] Compliance evidence archived.
- [ ] Lessons learned shared with next migration wave.
- [ ] Project tracker updated to closed or stabilized state.
