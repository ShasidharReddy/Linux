# Migration Operations and Checklists

← Back to [16-cloud-migration.md](./16-cloud-migration.md)

Reusable operational checklists, readiness packs, communications, and wave-management templates.

---

### Appendix K: Comprehensive pre-cutover checklist

#### Governance

- [ ] Change request approved.
- [ ] Business owner approval recorded.
- [ ] Support teams notified.
- [ ] War room bridge prepared.
- [ ] Status page draft prepared.
- [ ] Escalation matrix confirmed.
- [ ] Freeze window communicated.
- [ ] Vendor contacts on standby.
- [ ] Rollback authority defined.
- [ ] Evidence repository prepared.

#### Source environment

- [ ] Source backups successful.
- [ ] Source snapshots taken.
- [ ] Source monitoring healthy.
- [ ] No critical incidents open.
- [ ] Clock synchronization verified.
- [ ] Enough free storage for replication delta.
- [ ] Source antivirus exclusions reviewed.
- [ ] Replication agent healthy.
- [ ] Last configuration drift reviewed.
- [ ] Application writes freeze procedure tested.

#### Target environment

- [ ] Target subnet created.
- [ ] Route tables attached.
- [ ] Security rules approved.
- [ ] Target IAM roles assigned.
- [ ] Target logging enabled.
- [ ] Target backup policy attached.
- [ ] Jump host or bastion access tested.
- [ ] Load balancer configured.
- [ ] Secrets populated.
- [ ] Tagging standards applied.

#### Testing

- [ ] Smoke tests documented.
- [ ] Performance baseline available.
- [ ] Synthetic monitoring prepared.
- [ ] DB connectivity test ready.
- [ ] External dependency tests ready.
- [ ] User acceptance contacts available.
- [ ] Rollback tests documented.
- [ ] Serial console or boot diagnostics enabled.
- [ ] Post-cutover benchmark plan ready.
- [ ] DNS validation commands prepared.

### Appendix L: Workload archetypes and migration hints

- **Simple web server:** Rehost first, then place behind managed load balancer and WAF.
- **Three-tier business app:** Move tiers in dependency-aware waves and validate DB latency.
- **File server:** Review permissions, SMB or NFS semantics, and backup restoration process.
- **Directory-integrated app:** Validate LDAP, Kerberos, time sync, and group membership resolution.
- **License-bound app:** Check host binding, dongles, MAC locks, and vendor support statements.
- **Batch ETL worker:** Benchmark storage throughput and scheduling window changes.
- **Stateful database:** Use engine-native migration where possible, not only VM replication.
- **Low-latency plant system:** Consider retain or hybrid if circuit latency breaks requirements.
- **Legacy unsupported OS:** Upgrade first or isolate in tightly controlled migration pattern.
- **Internet-facing API:** Prioritize TLS, WAF, DDoS posture, and rate-limiting in target cloud.

### Appendix M: 90-day optimization backlog examples

1. Right-size oversized instances based on telemetry.
2. Move static assets to object storage and CDN.
3. Replace static admin keys with managed identities or IAM roles.
4. Centralize secrets in provider-native secret store.
5. Convert manual patching to automated maintenance windows.
6. Add autoscaling or scheduled scaling where safe.
7. Tune log retention for compliance and cost balance.
8. Remove temporary firewall exceptions from migration window.
9. Refine backup retention and copy policies.
10. Evaluate managed database adoption for next wave.
11. Automate infrastructure through Terraform, Bicep, CloudFormation, or Deployment Manager alternatives.
12. Implement policy-as-code guardrails.
13. Consolidate duplicate monitoring alerts.
14. Measure and reduce cross-zone or cross-region data transfer.
15. Document golden images and baseline hardening.

### Appendix N: Domain-by-domain migration questionnaire

#### Identity

- Which directory is authoritative for users?
- How are privileged accounts separated from daily-use accounts?
- Is MFA required for cloud admins?
- Which service accounts are embedded in config files?
- Are certificates tied to machine names or IP addresses?
- Is there a break-glass admin path?
- How often are groups synchronized?
- Are legacy NTLM dependencies present?
- Do workloads require Kerberos constrained delegation?
- How is password rotation automated?

#### Networking

- What are the source CIDR blocks?
- Do target CIDRs overlap?
- What DNS zones are required?
- Which egress destinations must remain reachable?
- Is asymmetric routing possible after migration?
- Will firewalls inspect east-west traffic?
- Are proxies required for package updates?
- What is the tolerated latency to retained systems?
- Which ports are opened only temporarily for migration?
- Is jumbo frame support required?

#### Storage

- What is the hot data set size?
- What is the cold data set size?
- Do applications rely on block, file, or object semantics?
- How long do restores currently take?
- Do snapshots need application quiescing?
- Is encryption key ownership regulated?
- What is the monthly growth rate?
- Are there archival obligations?
- What are peak write bursts?
- Are backup windows already constrained?

#### Operations

- Who owns patching?
- Who owns backup success review?
- Who approves emergency rollback?
- Are runbooks current?
- What is the change calendar conflict for the next quarter?
- Are NOC dashboards ready for the new cloud targets?
- Will ticket routing change?
- Is there a 24x7 support requirement?
- What evidence is required for closure?
- Are CMDB updates automated?

#### Security

- What logs must be centralized?
- How long must they be retained?
- Is endpoint protection provider-approved in cloud images?
- Which subnets may have internet access?
- Are public IPs prohibited?
- Is customer-managed encryption required?
- What vulnerability scan windows exist?
- What are the critical severity remediation SLAs?
- Is a cloud WAF mandated?
- Are admin sessions recorded?

### Appendix O: Example migration communications timeline

- T-14 days: lower TTL where allowed and confirm stakeholder contacts.
- T-7 days: finalize test evidence, rollback documents, and go/no-go criteria.
- T-3 days: confirm backups, replication health, and network changes.
- T-1 day: freeze non-essential changes and send final migration notice.
- T-2 hours: start war room, confirm all teams present, verify observability dashboards.
- T-0: stop writes as needed, execute cutover, and begin smoke testing.
- T+30 minutes: send first status update.
- T+2 hours: confirm application owner validation and watch error budgets.
- T+24 hours: end hypercare if stable.
- T+7 days: close wave and update lessons learned.

### Appendix P: Lessons learned prompts

- Which dependencies were discovered too late?
- Which access requests slowed execution?
- Did the selected instance sizes match observed usage?
- Which alerts were noisy or missing?
- Was rollback realistically achievable within the window?
- Did DNS behave as expected in hybrid mode?
- Did application owners have sufficient test coverage?
- Which automation should be built before the next wave?
- Were cost estimates accurate enough for executive reporting?
- What should be standardized as a reusable migration pattern?

#### Azure advanced command set

```bash
az monitor action-group create --name ag-prod --resource-group rg-migrate-prod --short-name agprod

az monitor metrics alert create --name cpu-high --resource-group rg-migrate-prod --scopes /subscriptions/<sub>/resourceGroups/rg-migrate-prod/providers/Microsoft.Compute/virtualMachines/app01 --condition "avg Percentage CPU > 80" --description "CPU high"

az network lb create --resource-group rg-migrate-prod --name lb-web --sku Standard --vnet-name vnet-prod-eastus --subnet snet-app

az vm extension set --publisher Microsoft.Azure.Monitor --name AzureMonitorLinuxAgent --resource-group rg-migrate-prod --vm-name app01

az disk list -g rg-migrate-prod -o table

az network private-dns zone create --resource-group rg-migrate-prod --name corp.internal

```

#### AWS advanced command set

```bash
aws ssm describe-instance-information --output table

aws backup list-backup-plans --output table

aws ec2 describe-volumes --filters Name=attachment.instance-id,Values=i-xxxxxxxx --output table

aws ec2 create-snapshot --volume-id vol-xxxxxxxx --description "precutover snapshot"

aws logs describe-log-groups --output table

aws iam list-roles --query "Roles[].RoleName" --output table

```

#### GCP advanced command set

```bash
gcloud compute routes list

gcloud compute networks peerings list --network=vpc-prod

gcloud logging sinks list

gcloud monitoring uptime list-configs

gcloud compute snapshots list

gcloud iam service-accounts list

```

### Appendix Q: Platform readiness matrices

#### Azure readiness matrix

- [ ] Landing zone approved
- [ ] Connectivity tested
- [ ] Identity integration validated
- [ ] Monitoring baseline created
- [ ] Backup policy attached
- [ ] Golden image or base hardening applied
- [ ] Cost tags or labels enforced
- [ ] Firewall rules reviewed
- [ ] Quota confirmed
- [ ] Runbook signed off
- [ ] Rollback checkpoints taken
- [ ] Application owner available for testing

#### AWS readiness matrix

- [ ] Landing zone approved
- [ ] Connectivity tested
- [ ] Identity integration validated
- [ ] Monitoring baseline created
- [ ] Backup policy attached
- [ ] Golden image or base hardening applied
- [ ] Cost tags or labels enforced
- [ ] Firewall rules reviewed
- [ ] Quota confirmed
- [ ] Runbook signed off
- [ ] Rollback checkpoints taken
- [ ] Application owner available for testing

#### GCP readiness matrix

- [ ] Landing zone approved
- [ ] Connectivity tested
- [ ] Identity integration validated
- [ ] Monitoring baseline created
- [ ] Backup policy attached
- [ ] Golden image or base hardening applied
- [ ] Cost tags or labels enforced
- [ ] Firewall rules reviewed
- [ ] Quota confirmed
- [ ] Runbook signed off
- [ ] Rollback checkpoints taken
- [ ] Application owner available for testing

### Appendix R: Sample migration wave backlog

#### Wave 1

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 2

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 3

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 4

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 5

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 6

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 7

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 8

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 9

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

#### Wave 10

- Review assessment output and dependency map.
- Confirm cloud account, project, or subscription placement.
- Validate CIDR allocation and DNS requirements.
- Confirm IAM group assignments and admin access.
- Create or verify backup and monitoring configuration.
- Run test migration and capture results.
- Resolve defects discovered in testing.
- Obtain go-live approval.
- Execute cutover and monitor hypercare.
- Document lessons learned.

### Appendix S: Migration glossary

- **Landing zone:** A pre-built cloud foundation with network, IAM, logging, policy, and management patterns.
- **Cutover:** The point when production traffic or write activity is moved to the target environment.
- **Hypercare:** An intensified support period immediately after cutover.
- **RPO:** Recovery Point Objective: acceptable data loss measured in time.
- **RTO:** Recovery Time Objective: acceptable time to restore service.
- **Right-sizing:** Adjusting instance sizes to fit measured workload demand.
- **Pilot wave:** An early, limited migration used to validate pattern and governance.
- **Shared responsibility model:** The division of security and operations responsibilities between cloud provider and customer.
- **Drift:** Configuration changes that deviate from the approved or automated baseline.
- **FinOps:** Operational discipline that aligns technology, finance, and business teams on cloud cost management.
