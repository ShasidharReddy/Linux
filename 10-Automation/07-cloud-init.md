# Cloud-Init

[Back to guide index](README.md)

Cloud-Init is a first-boot initialization system used widely in cloud images.

It configures instances early in their lifecycle based on metadata and user-provided data.

## 7.1 Common Use Cases

- Create users and SSH keys
- Install packages
- Write configuration files
- Run bootstrap commands
- Configure hostname
- Grow disks and filesystems
- Execute early instance customization

## 7.2 User-Data Types

Common user-data formats include:

- Shell scripts
- cloud-config YAML
- MIME multi-part archives
- Boothooks
- Include files

## 7.3 Basic Shell Script Example

```bash
#!/bin/bash
set -euxo pipefail
apt-get update
apt-get install -y nginx
systemctl enable --now nginx
```

## 7.4 cloud-config YAML Example

```yaml
#cloud-config
package_update: true
packages:
  - nginx
  - curl
users:
  - name: deploy
    groups: [sudo]
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... user@example
write_files:
  - path: /var/www/html/index.html
    permissions: '0644'
    content: |
      Provisioned by Cloud-Init
runcmd:
  - systemctl enable --now nginx
```

## 7.5 Useful cloud-config Modules

| Module | Purpose |
|---|---|
| users | Create users |
| ssh_authorized_keys | Configure SSH keys |
| packages | Install packages |
| write_files | Create files |
| runcmd | Run commands late in boot |
| bootcmd | Run early boot commands |
| growpart | Expand partition |
| resizefs | Resize filesystem |
| apt | Configure apt behavior |
| yum_repos | Configure yum repos |

## 7.6 bootcmd vs runcmd

| Directive | Timing | Typical Use |
|---|---|---|
| bootcmd | Early boot | Low-level setup |
| runcmd | Late in boot | Service enablement and final commands |

## 7.7 Example: Baseline Host Bootstrap

```yaml
#cloud-config
hostname: web-01
manage_etc_hosts: true
package_update: true
packages:
  - nginx
  - fail2ban
users:
  - default
  - name: deploy
    shell: /bin/bash
    groups: [sudo]
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... deploy@example
write_files:
  - path: /etc/motd
    permissions: '0644'
    content: |
      Managed by Cloud-Init
runcmd:
  - systemctl enable --now nginx
  - systemctl enable --now fail2ban
```

## 7.8 Multi-Part MIME Archives

Multi-part user-data lets you combine several content types.

Example use cases:

- cloud-config plus shell script
- cloud-config plus custom part handlers
- Separate app bootstrap and system bootstrap

## 7.9 Debugging Cloud-Init

Important files and commands:

```bash
cloud-init status --long
cloud-init analyze show
cloud-init analyze blame
journalctl -u cloud-init -u cloud-config -u cloud-final
cat /var/log/cloud-init.log
cat /var/log/cloud-init-output.log
```

## 7.10 Re-running Cloud-Init for Testing

Be careful with this in non-ephemeral systems.

```bash
sudo cloud-init clean
sudo reboot
```

## 7.11 Best Practices for Cloud-Init

1. Keep first boot concise.
2. Use it for bootstrap, not long-term convergence.
3. Prefer cloud-config over large opaque shell scripts.
4. Log outputs clearly.
5. Test on the target distribution image.
6. Avoid complex app deployments directly in user-data when better handled elsewhere.

## 7.12 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| Huge shell-only scripts | Hard debugging | Use structured cloud-config |
| Using Cloud-Init for long-term state | Limited reconciliation | Hand off to config management |
| Distribution-specific assumptions | Boot failures | Test per image family |
| Silent bootstrap failures | Hidden drift | Review logs and health checks |

## 7.13 Cloud-Init with Terraform

Cloud-Init is commonly embedded as `user_data` or rendered templates in Terraform.

This is an excellent combination for Linux provisioning pipelines.

## 7.14 Cloud-Init Summary

Use Cloud-Init for fast, repeatable first-boot customization, then hand over ongoing configuration to other tools when needed.

---
