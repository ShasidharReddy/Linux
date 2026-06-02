# Intrusion Detection

Intrusion detection helps identify malicious activity, policy violations, and attacker persistence.

No single tool is sufficient.

Use host-based and network-based detection together.

### 10.1 Host-Based Intrusion Detection

Host-based tools observe events on the server itself.

Examples:

- OSSEC or Wazuh
- AIDE
- rkhunter
- chkrootkit
- auditd-based detections

### 10.2 OSSEC and Wazuh

OSSEC is a host-based intrusion detection system.

Wazuh extends the OSSEC ecosystem with broader analytics and management features.

Capabilities typically include:

- log analysis
- file integrity monitoring
- rootkit detection
- policy monitoring
- active response

Good deployment patterns:

- enroll critical servers first
- centralize alerting
- tune noisy rules
- correlate with vulnerability and patch data

### 10.3 Network Intrusion Detection

Network IDS monitors packets or flows crossing the network.

Common tools:

- Snort
- Suricata

Use cases:

- detect exploit attempts
- detect suspicious C2 traffic
- detect protocol abuse
- analyze east-west traffic in segmented environments

### 10.4 Snort and Suricata

Both tools support signature-based detection.

Suricata also provides strong multi-threaded performance and rich protocol analysis in many deployments.

Operational tips:

- place sensors where traffic visibility is meaningful
- keep rules updated
- tune out expected internal noise
- integrate alerts with central log or SIEM workflows

### 10.5 rkhunter

`rkhunter` checks for known rootkit indicators and suspicious system states.

It can inspect:

- file hashes
- hidden files
- unusual strings
- suspicious kernel module indicators

Use it as one signal among many.

A clean report does not prove the system is clean.

### 10.6 chkrootkit

`chkrootkit` also looks for signs of rootkits and anomalies.

Guidance:

- run from trusted administration paths
- interpret results cautiously
- correlate with package validation and incident evidence

### 10.7 Log Analysis for Attacks

Important logs include:

- auth logs
- sudo logs
- audit logs
- web access and error logs
- firewall logs
- DNS logs
- application-specific logs

Look for patterns such as:

- repeated auth failures
- successful logins after repeated failures
- new user creation
- unexpected `curl` or `wget` downloads
- shell spawn events from web services
- privilege escalation attempts
- traffic to rare external destinations

### 10.8 Behavioral Indicators

Indicators of compromise may include:

- unexplained CPU spikes
- unusual outbound connections
- new scheduled tasks
- unauthorized SSH keys
- replaced binaries
- disabled logging
- changed firewall rules
- processes listening on unusual ports

### 10.9 Baselines Matter

Detection is more effective when you know what normal looks like.

Baseline:

- normal service list
- normal listening ports
- normal cron jobs
- expected package set
- normal outbound destinations
- standard admin login windows

### 10.10 Detection Engineering Tips

- prioritize high-signal detections
- suppress known-good noise carefully
- preserve context in alerts
- test detections with realistic scenarios
- review missed incidents to improve rules

### 10.11 Example Investigation Commands

```bash
ss -tulpn
ps auxf
lsof -nP -i
last -a
journalctl -xe
sudo ausearch -ts recent
sudo find /etc/cron* -type f -print -exec cat {} \;
```

### 10.12 Summary

Intrusion detection depends on visibility, baselines, and correlation.

Use host-based tools such as OSSEC or Wazuh, network sensors such as Snort or Suricata, rootkit checks, and strong log analysis to improve detection maturity.

---

---

## Related Checklists, Command Reference, and Review Questions

### A.10 Intrusion Detection Checklist

- Confirm host IDS enrollment.
- Confirm network IDS coverage where applicable.
- Update rules and signatures.
- Review high-severity alerts.
- Review rootkit tool findings.
- Compare against patch and vulnerability context.
- Tune false positives.
- Document detection gaps.
- Test alert routing.
- Validate evidence retention.

### B.10 Intrusion Detection and Investigation Commands

```bash
ps auxf
ss -tulpn
lsof -nP -i
journalctl -xe
find /etc/cron* -type f -print -exec cat {} \;
rpm -Va
debsums -s
```

### C.10 Intrusion Detection

111. What is the difference between host-based and network-based IDS?
112. When would Suricata be useful?
113. When would OSSEC or Wazuh be useful?
114. Why are rootkit tools only one signal among many?
115. Why is baseline knowledge essential for detection?
116. What are examples of suspicious outbound activity?
117. Why should detection rules be tuned over time?
118. What is alert fatigue?
119. Why is process and network enumeration valuable during triage?
120. Why should detection logic be tested with realistic scenarios?
