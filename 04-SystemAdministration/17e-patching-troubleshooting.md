# Patching Troubleshooting

← Back to [17-patching-and-vulnerabilities.md](./17-patching-and-vulnerabilities.md)

Operational scenarios and troubleshooting FAQ material for patch windows and remediation work.

---

## 🌍 Appendix C: Real-World Patching Scenarios

These scenarios show how theory becomes operations. Each scenario includes context, approach, commands, verification, and rollback thinking.

### 🌐 Scenario 1: Monthly patching of an internet-facing RHEL web cluster

**Environment:** Three RHEL web servers behind a load balancer serving customer traffic.

**Risk profile:** Security exposure is high because the service is internet-facing. Availability must remain above 99.9 percent.

**Execution strategy:**
- Drain one node at a time from the load balancer.
- Apply security updates only during the first wave.
- Reboot the node if required, validate HTTPS health, then return it to service.
- Repeat sequentially until all nodes pass validation.

```bash
# Example on each node
sudo dnf updateinfo list security
sudo dnf upgrade --security -y
sudo needs-restarting -r || echo "No reboot required"
sudo reboot
```

**Validation checklist:**
- Check `/health` endpoint returns HTTP 200.
- Confirm node is back in load balancer rotation.
- Review `journalctl -b` for startup failures.
- Confirm customer synthetic monitoring remains green.

**Rollback and containment notes:**
- Keep prior VM snapshot for the first node.
- If the node fails health checks, remove it from rotation and revert snapshot or downgrade packages.
- Pause the campaign until root cause is understood.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🛠️ Scenario 2: Ubuntu application fleet with unattended-upgrades for security patches

**Environment:** Ten Ubuntu app servers running internal business APIs.

**Risk profile:** Patch consistency is poor when teams rely on manual SSH sessions.

**Execution strategy:**
- Enable unattended-upgrades for security repositories.
- Use a maintenance window for reboots rather than unscheduled daytime reboots.
- Collect package logs centrally.

```bash
sudo apt install -y unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades
sudo unattended-upgrade --dry-run --debug
```

**Validation checklist:**
- Check `/var/log/unattended-upgrades/` for successful runs.
- Verify application services remain active after package changes.
- Review `/var/run/reboot-required` and coordinate reboot windows.

**Rollback and containment notes:**
- Restore from snapshot if app stack breaks after library upgrade.
- Temporarily hold affected packages while root cause is investigated.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🗄️ Scenario 3: Patching a two-node database cluster with minimal downtime

**Environment:** Two Linux VMs running a replicated database cluster.

**Risk profile:** Simultaneous reboot could create service outage or split-brain risk.

**Execution strategy:**
- Verify replication health before changes.
- Fail traffic to node B while patching node A.
- Rejoin node A, validate replication, then patch node B.
- Keep DBA on bridge during the change.

```bash
# OS patching example
sudo dnf upgrade --security -y
sudo reboot

# Example database health checks are product-specific
```

**Validation checklist:**
- Replication healthy before and after each node reboot.
- Database accepts writes after final failback.
- Latency and error metrics remain within baseline.

**Rollback and containment notes:**
- If a patched node fails to rejoin, fail all traffic to the healthy node and restore the broken node from snapshot if needed.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🚑 Scenario 4: Emergency kernel CVE remediation with kpatch

**Environment:** Critical RHEL systems supporting 24x7 transactions.

**Risk profile:** Immediate reboot during business hours is unacceptable, but exploit risk is high.

**Execution strategy:**
- Deploy vendor-supported live patch using kpatch.
- Schedule standard reboot and full kernel alignment in the next maintenance window.
- Track the temporary risk acceptance window explicitly.

```bash
sudo dnf install -y kpatch kpatch-dnf
sudo systemctl enable --now kpatch.service
kpatch list
```

**Validation checklist:**
- Confirm live patch is loaded.
- Check application latency and kernel logs.
- Verify emergency change record documents later reboot requirement.

**Rollback and containment notes:**
- Unload or revert live patch only under vendor guidance and only if the patch itself causes instability.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🛰️ Scenario 5: Satellite-managed RHEL production patch cycle

**Environment:** Hundreds of RHEL servers registered to Red Hat Satellite across Dev, Test, and Prod.

**Risk profile:** Uncontrolled repository exposure could lead to inconsistent package states.

**Execution strategy:**
- Sync repos to Satellite.
- Publish content view after testing.
- Promote content through lifecycle environments.
- Patch production only after approval.

```bash
hammer repository synchronize --name "Red Hat Enterprise Linux 8 for x86_64 - BaseOS RPMs" --product "Red Hat Enterprise Linux for x86_64" --organization "ExampleCorp"
hammer content-view publish --name RHEL8-Base --organization "ExampleCorp"
hammer content-view version promote --content-view RHEL8-Base --to-lifecycle-environment Prod --organization "ExampleCorp"
```

**Validation checklist:**
- Confirm production hosts are attached to the intended content view version.
- Review applicable errata counts after patching.
- Export compliance report for CAB records.

**Rollback and containment notes:**
- Republish or repromote previous content view version if a bad package set was approved.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🏝️ Scenario 6: Air-gapped patching for a regulated site

**Environment:** A disconnected environment with strict import controls.

**Risk profile:** Patches arrive less frequently and require careful repository curation.

**Execution strategy:**
- Mirror approved packages to a staging repo outside the air gap.
- Transfer signed content through approved media handling.
- Test in a representative disconnected environment before production deployment.

```bash
# Example concepts
reposync --download-metadata --repoid=rhel-8-baseos-rpms
createrepo /srv/mirror/rhel8-baseos
```

**Validation checklist:**
- Verify GPG signatures and repository metadata.
- Confirm disconnected hosts resolve the internal mirror only.
- Document chain of custody for imported content.

**Rollback and containment notes:**
- Restore prior mirror snapshot or remove newly published repository version.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 🔐 Scenario 7: Remediating a Nessus finding for vulnerable OpenSSL

**Environment:** Mixed RHEL and Ubuntu web servers flagged by a scheduled scan.

**Risk profile:** TLS-related issues are high visibility and often externally reachable.

**Execution strategy:**
- Validate installed OpenSSL package versions and linked service impact.
- Patch in staging first.
- Restart or reload dependent services such as NGINX, Apache, HAProxy, or custom daemons.

```bash
# RHEL
rpm -q openssl
sudo dnf upgrade openssl -y

# Ubuntu
dpkg -l | grep openssl
sudo apt install --only-upgrade openssl -y
```

**Validation checklist:**
- Check scanner re-run clears the finding.
- Verify TLS handshakes and certificate presentation after service restart.
- Confirm no legacy ciphers were re-enabled accidentally.

**Rollback and containment notes:**
- Downgrade package only if service incompatibility occurs and no safer mitigation exists; otherwise revert VM snapshot.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 📐 Scenario 8: CIS hardening before an external audit

**Environment:** A group of Linux jump hosts in audit scope.

**Risk profile:** Last-minute hardening can break admin workflows if not tested.

**Execution strategy:**
- Run CIS assessment first.
- Remediate Level 1 controls in lower environment.
- Create exception list for controls that conflict with operational requirements.

```bash
oscap info /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis --report cis-report.html /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
```

**Validation checklist:**
- Review score improvement after remediation.
- Test SSH, sudo, logging, and break-glass access.
- Attach assessment reports to audit evidence set.

**Rollback and containment notes:**
- Revert individual controls rather than backing out all hardening if only a small subset causes issues.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 💳 Scenario 9: PCI weekend patch window for payment-processing systems

**Environment:** Linux hosts handling payment-related workloads in a segmented environment.

**Risk profile:** Regulated scope and high sensitivity require tight evidence collection.

**Execution strategy:**
- Perform pre-window authenticated scan.
- Patch systems in service order.
- Capture command outputs, screenshots, and ticket updates during the change.
- Run validation plus follow-up scan.

```bash
# Example evidence commands
sudo dnf history list
sudo dnf updateinfo summary installed
journalctl -b --no-pager | tail -50
```

**Validation checklist:**
- Compliance dashboard updated within the same reporting cycle.
- Remaining exceptions approved with expiration dates.
- Payment application transactions validated end-to-end.

**Rollback and containment notes:**
- Use tier-specific rollback plans, especially for databases and transaction brokers.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### ↩️ Scenario 10: Recovering from a failed patch after reboot

**Environment:** A single business-critical VM fails application startup after patching.

**Risk profile:** Extended outage while engineers troubleshoot under pressure.

**Execution strategy:**
- Use console access immediately.
- Capture logs before making more changes.
- Determine whether the issue is package regression, configuration drift, or service dependency failure.
- Decide between rollback and forward-fix.

```bash
# Useful triage commands
systemctl --failed
journalctl -xe --no-pager | tail -100
rpm -qa --last | head -50
uname -r
```

**Validation checklist:**
- Application owner confirms restored service.
- Monitoring and backups are healthy.
- Root cause analysis is opened before next cycle.

**Rollback and containment notes:**
- Boot prior kernel, undo DNF transaction, or restore VM snapshot depending on fastest safe recovery path.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### ☁️ Scenario 11: Golden image patching for autoscaled cloud nodes

**Environment:** Cloud worker nodes are replaced frequently from machine images.

**Risk profile:** In-place patching can drift from image pipeline standards.

**Execution strategy:**
- Patch the image pipeline first.
- Bake new approved image.
- Roll out by replacing instances instead of long-lived in-place updates where possible.

```bash
# Example build step inside image pipeline
sudo dnf upgrade -y
sudo systemctl disable --now unnecessary-service || true
```

**Validation checklist:**
- New instances launch healthy and register to monitoring.
- Old instances are drained and terminated.
- Image metadata records patch date and package baseline.

**Rollback and containment notes:**
- Revert autoscaling group launch template to previous image version.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

### 📉 Scenario 12: Qualys-driven remediation sprint for legacy Linux servers

**Environment:** Older Linux servers with many medium and high findings.

**Risk profile:** Some findings may be false positives due to backported vendor fixes.

**Execution strategy:**
- Sort findings by exploitable exposure and patchability.
- Cross-check vendor advisories for backported fixes.
- Retire unsupported systems where patching is no longer sufficient.

```bash
# Backport verification examples
rpm -q --changelog glibc | grep -i CVE | tail -20
apt changelog sudo | grep -i CVE
```

**Validation checklist:**
- False positives documented with vendor references.
- Patchable items reduced first.
- Unsupported assets flagged for refresh or isolation.

**Rollback and containment notes:**
- Legacy rollback often depends on VM snapshots because package version retention may be limited.

**Operational lesson:**
Patching succeeds when technical steps, service context, and recovery planning are treated as one workflow rather than separate tasks.

## ❓ Appendix D: Troubleshooting FAQ

### ❔ Why does a scanner still show a CVE after patching?

- The scanner may use version comparison logic that does not understand vendor backports.
- The service may still be running an old process image and need restart or reboot.
- A secondary package or bundled library may remain vulnerable.
- Rescan timing may be delayed or cached.

### ❔ Why did the host not reboot even though a kernel package updated?

- The package manager does not automatically reboot by default in most enterprise workflows.
- A running kernel remains active until reboot.
- Use reboot-required indicators or tools such as `needs-restarting -r` to detect this state.

### ❔ Why does `dnf check-update` return exit code 100?

- On RHEL-family systems, exit code 100 commonly means updates are available.
- Treat it as expected behavior in automation rather than a failure.

### ❔ Why did patching fill `/boot`?

- Old kernel packages may not have been cleaned up.
- Systems with small boot partitions are especially sensitive.
- Always check space before large kernel update campaigns.

### ❔ Why is rollback harder than expected?

- Dependencies may have changed during the transaction.
- Prior package versions may no longer exist in enabled repos.
- Snapshot-based rollback is usually faster and safer for full-host recovery.

### ❔ Why do CIS controls break SSH access?

- Settings such as root login disablement, stricter PAM, or network restrictions may affect remote admin workflows.
- Test hardening in lower environments and preserve emergency access paths.

### ❔ Why does a package look old but still contain the fix?

- Enterprise vendors frequently backport security fixes without rebasing to the newest upstream version.
- Read vendor advisories and changelogs instead of assuming upstream version logic.

### ❔ Why does a patch window overrun?

- Insufficient pre-checks, slow reboots, application validation delays, or repo/network issues are common causes.
- Measure actual durations and improve the runbook each cycle.

### ❔ Why do load-balanced nodes fail health checks after patching?

- Application dependencies may have restarted in the wrong order.
- New libraries may require service reload or configuration adjustment.
- Kernel or firewall changes may affect local connectivity.

### ❔ Why does a vulnerability remain open when no patch exists?

- Not all vulnerabilities have immediate vendor fixes.
- Use mitigations such as disabling a feature, firewall rules, config changes, or service isolation while tracking the exception.

### ❔ Why should I patch staging before production if the vulnerability is critical?

- Staging validates package compatibility and helps prevent self-inflicted outages.
- For truly urgent cases, use abbreviated but still deliberate testing on a representative canary host.

### ❔ Why do package-manager transactions fail on one host only?

- Repo configuration drift, broken dependencies, stale metadata, or local filesystem issues may exist.
- Compare repository files and package history with a healthy peer.

### ❔ Why does `unattended-upgrades` not patch everything?

- Its scope depends on configured origins and policies.
- It is often aimed at security updates rather than arbitrary full distribution upgrades.

### ❔ Why is live patching not enough forever?

- Live patching covers only selected kernel issues and supported kernels.
- Periodic full kernel alignment and reboot remain necessary for long-term support posture.

### ❔ Why do auditors ask for exception registers?

- No environment patches 100 percent of findings immediately.
- Auditors want to see that unresolved risk is known, owned, time-bound, and approved.

### ❔ Why is vulnerability management broader than patching?

- Some issues are configuration weaknesses or design flaws, not missing packages.
- Mitigation may require network controls, account restrictions, or service redesign.

### ❔ Why do I need application-level testing after OS patching?

- Package updates can affect runtime libraries, ciphers, kernel behavior, and startup ordering.
- Service health is not the same as business functionality.

### ❔ Why do I need a change ticket for routine monthly patching?

- The ticket provides authorization, accountability, communication, and retained evidence.
- Even routine work benefits from formal tracking.

### ❔ Why do reboot flags differ across distros?

- Each distro exposes reboot-required state differently.
- Build automation that is distro-aware rather than assuming one method works everywhere.

### ❔ Why should dashboards track missing reboot counts?

- Many hosts appear patched on disk but still run vulnerable code in memory until reboot.
- Missing reboot counts expose this hidden risk.
