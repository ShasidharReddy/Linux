# Troubleshooting Matrix and FAQ

← Back to [13-vm-ssh-access-issues.md](./13-vm-ssh-access-issues.md)

Closing guidance, symptom matrices, validation loops, and responder FAQ material.

---

## 13.7 📚 Troubleshooting Matrix

| Symptom | Likely causes | Best checks | Preferred fix |
|---|---|---|---|
| Cannot resolve hostname | DNS record missing, split DNS, bad local alias | Use `getent hosts`, `host`, `ssh -G` | Fix the record or local config |
| No route to host | Missing route, VPN path missing, gateway issue | Inspect `ip route`, compare working client | Repair source routing or VPN policy |
| Timeout to port 22 | Firewall drop, route issue, hung VM | Use `nc`, `traceroute`, flow logs, console checks | Open correct policy or recover the VM |
| Connection refused to port 22 | No SSH listener, daemon down, reject rule | Use `ss -ltnp`, `systemctl status sshd` | Start or repair SSH service and listener |
| Permission denied publickey | Wrong key, wrong user, bad permissions | Use `ssh -vvv`, `ssh-add -l`, auth logs | Install correct key and fix ownership |
| Permission denied publickey,password | Account issue or wrong auth method | Check account state and auth policy | Unlock, correct user, or fix auth method |
| Host key verification failed | Rebuilt host, wrong DNS, IP reuse | Validate fingerprint from trusted source | Update known_hosts safely |
| Works from bastion only | Private design or direct ingress blocked | Test second hop from bastion | Use bastion or adjust scoped ingress |
| Auth succeeds then disconnects | Shell, PAM, disk, or home dir issue | Check auth logs and shell environment | Repair post-auth environment |
| Can SSH but cannot read logs | Permissions or ownership issue | Inspect groups, ACLs, sudo path | Grant least-privilege access |
| Log access breaks after rotation | Bad logrotate ownership or ACL persistence | Inspect rotation policy and force test | Fix `create`, `su`, and ACL inheritance |
| Many teams need logs | Poor delegation model | Review current access requests and audit findings | Centralize logs with RBAC |

### 13.7.1 Extended symptom cards

#### Symptom Card 1: Laptop cannot resolve private VM name

- **Likely explanation:** VPN DNS or split-horizon issue
- **Immediate action:** Reconnect VPN and compare resolver output with a working client.

#### Symptom Card 2: Ping succeeds but SSH times out

- **Likely explanation:** TCP-specific filtering or host firewall
- **Immediate action:** Run `nc -vz target 22` and inspect packet captures or flow logs.

#### Symptom Card 3: SSH times out from home but works from office

- **Likely explanation:** Source-CIDR policy mismatch
- **Immediate action:** Compare inbound rule source ranges and home NAT IP.

#### Symptom Card 4: SSH refused after maintenance

- **Likely explanation:** Daemon failed to restart cleanly
- **Immediate action:** Run `sshd -t` and inspect `systemctl status sshd`.

#### Symptom Card 5: Known host changed after rebuild

- **Likely explanation:** Legitimate host identity change or wrong target
- **Immediate action:** Validate fingerprint from console or inventory.

#### Symptom Card 6: Public key works for one user but not another

- **Likely explanation:** Wrong `authorized_keys` location or file ownership
- **Immediate action:** Check user home, `.ssh`, and file permissions.

#### Symptom Card 7: Auth works only with one old RSA key

- **Likely explanation:** Algorithm mismatch or compatibility drift
- **Immediate action:** Review supported key algorithms on both client and server.

#### Symptom Card 8: App team cannot read journal logs

- **Likely explanation:** File access was granted but journal access was not
- **Immediate action:** Use `systemd-journal` or scoped sudo for journal queries.

#### Symptom Card 9: ACL works on file but not new rotated file

- **Likely explanation:** No default ACL or logrotate recreate problem
- **Immediate action:** Add default ACLs or fix logrotate create mode.

#### Symptom Card 10: Only bastion can reach target subnet

- **Likely explanation:** Expected private topology or missing direct route
- **Immediate action:** Document bastion dependency and test second hop regularly.

#### Symptom Card 11: Cloud rule exists but traffic still drops

- **Likely explanation:** Wrong tag, wrong NIC, wrong subnet, or NACL
- **Immediate action:** Review effective policy rather than intended policy.

#### Symptom Card 12: Session closes after MOTD

- **Likely explanation:** Shell startup, PAM, or home dir problem
- **Immediate action:** Check non-interactive command behavior and auth logs.

#### Symptom Card 13: Repeated retries now always fail

- **Likely explanation:** Fail2ban or lockout triggered by earlier bad attempts
- **Immediate action:** Check ban or lockout state before retrying again.

#### Symptom Card 14: Users keep asking for root to view logs

- **Likely explanation:** No delegated observability model
- **Immediate action:** Implement RBAC-backed centralized logging.

#### Symptom Card 15: SSH works by IP but not by name

- **Likely explanation:** DNS or SSH alias drift
- **Immediate action:** Compare `ssh -G`, `getent hosts`, and inventory.

#### Symptom Card 16: One VM in a fleet is unreachable

- **Likely explanation:** Host-specific drift or health issue
- **Immediate action:** Compare with a healthy sibling instance.

#### Symptom Card 17: Serial console works but network does not

- **Likely explanation:** Guest booted but network path or firewall still broken
- **Immediate action:** Inspect NIC, routes, and policy from the guest.

#### Symptom Card 18: Only IPv6 path fails

- **Likely explanation:** Unexpected AAAA record or IPv6 policy gap
- **Immediate action:** Force IPv4 once to isolate and inspect IPv6 rules.

#### Symptom Card 19: Sudo log reads are questioned by compliance

- **Likely explanation:** Access lacks clear audit linkage
- **Immediate action:** Tie sudo usage to named accounts and incident/change references.

#### Symptom Card 20: Engineers use stale IPs after failover

- **Likely explanation:** Inventory or documentation drift
- **Immediate action:** Adopt stable DNS names and update runbooks after every failover.

## 13.8 🏁 Closing Guidance

- Treat VM reachability, SSH transport, SSH authentication, and log authorization as four separate layers.
- Use the smallest safe change that restores access and keep an audit trail for every privileged action.
- For recurring log access needs, move away from manual sudo and toward groups, ACLs, or centralized logging with RBAC.
- For recurring access outages, codify firewall, bastion, DNS, and SSH settings in managed configuration rather than tribal knowledge.
- After every incident, update the runbook with the exact symptom, root cause, and validated fix path.

## 13.9 📒 Expanded Validation and Recovery Notes

### VM validation loop

- Retest from the original failing source and at least one control source.
- Confirm the VM state in the cloud console or hypervisor after the fix.
- Verify the expected IP, NIC attachment, and route table did not drift.
- If a firewall change was made, prove only the intended source can now connect.
- Record the exact rule, change ID, and rollback point.

### SSH validation loop

- Retest with `ssh -vvv` once to confirm the failure stage is gone.
- Confirm `sshd` is active and listening on the intended address and port.
- Verify the intended user can log in with the approved auth method.
- If a host key changed, confirm clients updated only after identity verification.
- Document any key, account, or config changes in the change log.

### Log access validation loop

- Confirm the user can read exactly the intended logs and not unrelated sensitive data.
- Test the chosen model after a service restart or rotation event if applicable.
- Validate that sudo, ACL, or group access is auditable and documented.
- Ensure application processes still write logs normally after permission changes.
- Update runbooks so future responders use the approved path first.

## 13.10 📗 FAQ for On-Call Responders

### FAQ 1: Should I trust ping as proof that SSH should work?

No. ICMP can be blocked while TCP/22 works, and the reverse can also be true.

### FAQ 2: What is the first command I should run for a hostname problem?

Use `getent hosts target` and compare it with `ssh -G target` output.

### FAQ 3: What does `Connection refused` usually mean?

It usually means you reached the host and no service is listening or an active reject happened.

### FAQ 4: What does `Connection timed out` usually mean?

It usually points to a dropped packet, bad route, or an unresponsive host.

### FAQ 5: Why do I care about `sshd -t`?

It validates SSH daemon configuration before a restart so you do not make the outage worse.

### FAQ 6: Why not just `chmod 644` the log file?

Because it is often too broad, may expose sensitive data, and may break again after rotation.

### FAQ 7: When should I use ACLs instead of groups?

Use ACLs when one team needs narrow access to one path without broader group exposure.

### FAQ 8: Why is centralized logging better for app teams?

It gives them searchable access without root shell and scales better for many teams.

### FAQ 9: What if the VM is healthy in the console but unreachable?

Check effective policy, routes, and whether the guest OS or firewall is actually listening and responding.

### FAQ 10: What if only one engineer fails to connect?

Suspect local config, VPN, endpoint policy, or stale SSH client settings first.
