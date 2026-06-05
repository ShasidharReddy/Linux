# Question 5: VM Provisioning, OS & Hardening

Once the cluster, network, firewall, and storage are ready, the next goal is to create a **VM factory**: fast, consistent, and secure from the first boot.

## Dependencies Before First VM

1. Hypervisor cluster healthy ✓
2. Network VLANs configured ✓
3. Firewall rules allowing VM traffic ✓
4. Shared storage mounted and tested ✓
5. DNS and DHCP/static IP plan ready ✓
6. NTP source available ✓

## VM provisioning dependency chain

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Hypervisor Cluster] --> B[Network and VLANs]
    B --> C[Firewall Policy]
    C --> D[Shared Storage]
    D --> E[Golden Image]
    E --> F[Terraform or Clone Workflow]
    F --> G[Cloud-Init]
    G --> H[Ansible Hardening]
~~~

## Provisioning Strategy Decision

| Method | Pros | Cons | Use When |
|--------|------|------|----------|
| Manual ISO install | Simple, no pipeline needed | Slow, inconsistent, not scalable | One-off labs |
| Golden image / template | Fast clone, consistent baseline | requires image maintenance | 10-50 VMs |
| Terraform + Proxmox provider | Declarative, version-controlled, idempotent | provider learning curve | 50+ VMs or IaC-first teams |
| Ansible + virt/proxmox modules | Flexible and integrates with config mgmt | more procedural than Terraform | config-heavy environments |
| PXE + Kickstart | fully automated from network boot | more up-front complexity | very large fleets |

## Recommendation

Use **Packer-built golden images + Terraform for provisioning + Ansible for post-provision configuration**.

Why:

- image provides a clean, tested baseline
- Terraform gives repeatable desired-state VM creation
- Ansible handles per-role config and hardening cleanly
- cloud-init bridges image and instance-specific identity settings

## Provisioning workflow

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    P[Packer Build] --> T[Template in Proxmox]
    T --> TF[Terraform Clone]
    TF --> CI[Cloud-Init Personalization]
    CI --> ANS[Ansible Hardening and Role Config]
    ANS --> MON[Monitoring and Compliance Checks]
~~~

## Golden Image Design

### What goes into the image

- base OS: Rocky Linux 9 or Ubuntu 22.04 LTS
- SSH daemon configured for key-based login
- qemu guest agent
- cloud-init
- chrony
- monitoring agent such as node_exporter
- base packages: `vim`, `curl`, `wget`, `lsof`, `tcpdump`, `strace`, `tmux`, `bash-completion`
- SELinux enforcing on RHEL-family or AppArmor defaults on Ubuntu
- CIS Level 1 baseline where it does not break the platform

### What stays out of the image

- hostname
- static IP address
- app-specific packages
- tenant secrets and certificates
- service-specific users and access policies
- database data or application content

## Golden image vs per-VM settings

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    A[Golden Image Baked] --> B[OS Packages]
    A --> C[QEMU Agent]
    A --> D[Cloud-Init]
    A --> E[Baseline Hardening]
    F[Per-VM at First Boot] --> G[Hostname and IP]
    F --> H[SSH Keys]
    F --> I[Role Packages]
    F --> J[Application Config]
~~~

## Packer template example (HCL)

~~~hcl
packer {
  required_plugins {
    proxmox = {
      source  = "github.com/hashicorp/proxmox"
      version = ">= 1.1.0"
    }
  }
}

source "proxmox-iso" "rocky9" {
  proxmox_url              = "https://pve01.infra.example.com:8006/api2/json"
  username                 = "root@pam"
  password                 = var.proxmox_password
  node                     = "pve01"
  insecure_skip_tls_verify = true

  vm_name     = "tmpl-rocky9-golden"
  template_description = "AM golden image"
  cpu_type    = "host"
  cores       = 2
  memory      = 4096
  scsi_controller = "virtio-scsi-pci"
  disks {
    disk_size    = "20G"
    format       = "raw"
    storage_pool = "am-nfs-vmdata"
    type         = "virtio"
  }
  network_adapters {
    bridge = "vmbr0"
    model  = "virtio"
  }

  boot_iso {
    type         = "scsi"
    iso_file     = "am-nfs-vmdata:iso/Rocky-9-latest-x86_64-minimal.iso"
    unmount      = true
  }

  ssh_username = "packer"
  ssh_password = var.packer_ssh_password
  boot_command = [
    "<up><wait><tab> inst.ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ks.cfg<enter>"
  ]
}

build {
  sources = ["source.proxmox-iso.rocky9"]

  provisioner "shell" {
    inline = [
      "sudo dnf install -y qemu-guest-agent cloud-init chrony vim curl wget tmux lsof tcpdump strace policycoreutils-python-utils",
      "sudo systemctl enable qemu-guest-agent chronyd",
      "sudo sed -i 's/^SELINUX=.*/SELINUX=enforcing/' /etc/selinux/config",
      "sudo cloud-init clean"
    ]
  }
}
~~~

## Terraform example for Proxmox VMs

~~~hcl
terraform {
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = ">= 0.60.0"
    }
  }
}

provider "proxmox" {
  endpoint = "https://pve01.infra.example.com:8006/api2/json"
  username = var.proxmox_username
  password = var.proxmox_password
  insecure = true
}

resource "proxmox_virtual_environment_vm" "web01" {
  name      = "web01"
  node_name = "pve02"
  tags      = ["prod", "web"]

  clone {
    vm_id = 9000
    full  = true
  }

  cpu {
    cores = 4
    type  = "host"
  }

  memory {
    dedicated = 8192
  }

  network_device {
    bridge = "vmbr0"
    vlan_id = 30
  }

  disk {
    datastore_id = "am-nfs-vmdata"
    interface    = "scsi0"
    size         = 50
  }

  initialization {
    datastore_id = "am-nfs-vmdata"
    user_account {
      username = "ops"
      keys     = [file("~/.ssh/am_ops_ed25519.pub")]
    }
    ip_config {
      ipv4 {
        address = "10.10.30.21/24"
        gateway = "10.10.30.1"
      }
    }
    dns {
      servers = ["10.10.10.53"]
      domain  = "infra.example.com"
    }
  }
}
~~~

## Ansible playbook for post-provision hardening

~~~yaml
---
- name: Harden baseline Linux VMs
  hosts: linux_vms
  become: true
  vars:
    disable_services:
      - cups
      - avahi-daemon
      - rpcbind
  tasks:
    - name: Ensure key packages exist
      package:
        name:
          - chrony
          - firewalld
          - audit
          - sudo
          - rsyslog
          - policycoreutils-python-utils
        state: present

    - name: Enable critical services
      service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop:
        - chronyd
        - firewalld
        - auditd

    - name: Disable unused services
      service:
        name: "{{ item }}"
        enabled: false
        state: stopped
      loop: "{{ disable_services }}"
      ignore_errors: true

    - name: Configure SSH hardening
      copy:
        dest: /etc/ssh/sshd_config.d/99-hardening.conf
        content: |
          PermitRootLogin no
          PasswordAuthentication no
          KbdInteractiveAuthentication no
          AllowGroups ops-admins
          MaxAuthTries 3
      notify: restart sshd

    - name: Set SELinux enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Add sysctl baseline
      copy:
        dest: /etc/sysctl.d/99-am.conf
        content: |
          net.ipv4.conf.all.rp_filter = 1
          net.ipv4.icmp_echo_ignore_broadcasts = 1
          kernel.randomize_va_space = 2
      notify: reload sysctl

  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted

    - name: reload sysctl
      command: sysctl --system
~~~

## CIS hardening checklist

| Area | Key Items |
|------|-----------|
| Filesystem | separate `/var/log`, `nodev` on removable media, sensible mount options |
| Network | disable unused protocols, set safe sysctl defaults, no forwarding unless required |
| Authentication | password policy, lockout, sudo restrictions, no shared admin accounts |
| Audit | auditd rules for privileged commands and config files |
| Services | disable cups, avahi, rpcbind, and other unused daemons |
| Firewall | firewalld enabled, default-deny stance for exposed hosts |

## Useful guest prep commands

~~~bash
systemctl enable --now qemu-guest-agent chronyd
cloud-init status --wait
journalctl -u cloud-init -b
sestatus || aa-status
~~~

## Cloud-init user-data example

~~~yaml
#cloud-config
hostname: web01
fqdn: web01.infra.example.com
manage_etc_hosts: true
users:
  - name: ops
    groups: [wheel]
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAA...REPLACE_ME
package_update: true
package_upgrade: false
runcmd:
  - [ systemctl, enable, --now, qemu-guest-agent ]
  - [ systemctl, enable, --now, chronyd ]
~~~

## Fleet scaling patterns

### Inventory example

~~~ini
[webservers]
web01
web02

[dbservers]
db01

[k8s_workers]
k8s-w01
k8s-w02
k8s-w03

[linux_vms:children]
webservers
dbservers
k8s_workers
~~~

### Role layout

- `common`
- `hardening`
- `monitoring`
- `web`
- `db`
- `k8s_node`

### Secret handling

Use **Ansible Vault** or a secret backend such as HashiCorp Vault. Do not place credentials directly into Terraform or plain inventory files.

## Golden image lifecycle

~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    A[Monthly Patch Window] --> B[Rebuild Image with Packer]
    B --> C[Test Clone in Non-Prod]
    C --> D[Security and Function Validation]
    D --> E[Promote Template ID]
    E --> F[Terraform Uses New Template]
~~~

## CI/CD pattern for images

1. source repo change or scheduled rebuild triggers pipeline
2. packer build runs against staging Proxmox node
3. smoke test VM is created automatically
4. security baseline and agent checks run
5. template is promoted and versioned
6. old template retained until rollback window closes

## Common trade-offs

| Choice | Benefit | Cost |
|--------|---------|------|
| One image per OS family | less duplication | role config grows |
| Many app-specific images | faster workload boot | image sprawl |
| Immutable rebuild over in-place drift | clean rollback and repeatability | requires discipline and pipeline maturity |

## Verification checklist

| Check | Command | Healthy Signal |
|-------|---------|----------------|
| Clone speed | Terraform apply or template clone timing | predictable duration |
| cloud-init | `cloud-init status --wait` | done without errors |
| guest agent | `qm agent <vmid> ping` | responds |
| SSH access | ssh via bastion | key-based login works |
| hardening | `sestatus`, `firewall-cmd --state`, `auditctl -s` | enabled and active |

## Common mistakes

- stuffing secrets into the golden image
- using manual snowflake VMs for production roles
- failing to version templates
- hardening before cloud-init and guest agent logic are validated
- patching VMs in place forever without rebuilding images

## Exit criteria

You are ready for [06-linux-os-layer.md](./06-linux-os-layer.md) and [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) when:

- a golden image exists and is versioned
- Terraform can clone VMs repeatably
- cloud-init customization works
- Ansible hardening completes without manual steps


## Procurement & Cost Analysis

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Resource Planning

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## System Design & Architecture

### ADR summary

| ADR | Decision | Why | Alternative |
|-----|----------|-----|-------------|
| ADR-01 | golden images per OS family | cleaner lifecycle and fewer snowflakes | fully manual per-VM build rejected |
| ADR-02 | cloud-init for day-0 settings | simple, native, repeatable | baking environment data into image rejected |
| ADR-03 | Ansible for day-1/day-2 config | readable and broad ecosystem | heavy custom scripts rejected |
| ADR-04 | promotion rings for templates | limits blast radius | direct prod promotion rejected |

### Integration points

- Consumes compute/storage from [01-hypervisor-layer.md](./01-hypervisor-layer.md) and [04-shared-storage.md](./04-shared-storage.md).
- Supplies hardened Linux baselines to [06-linux-os-layer.md](./06-linux-os-layer.md) and worker/container hosts to [07-containers-and-monitoring.md](./07-containers-and-monitoring.md) and [09-kubernetes-deployment.md](./09-kubernetes-deployment.md).

## Planning & Timeline

> See [Common Procurement & Planning Guide](./10-common-procurement-and-planning.md) for procurement costs, resource planning, and implementation timelines.

## Advanced Production Configurations

### Compliance and fleet tooling

| Tool | Best Use | Cost Guidance |
|------|----------|---------------|
| OpenSCAP | CIS/STIG benchmark validation | Free |
| Nessus | agentless vuln scans and reporting | ~$3K-$5K+/yr depending on tier |
| Qualys | enterprise VMDR and compliance at scale | enterprise pricing |

- For 100+ VMs, organize by lifecycle rings: sandbox, non-prod, prod, and regulated.
- Add admission controls in Terraform/OpenTofu to reject missing owner, environment, TTL, and backup tags.
- DR pattern: rebuild from template + config rather than replicate every pet server where possible.
- Capacity alerts: dormant VM >30 days, snapshot age >14 days, template older than 35 days, failed compliance scan, orphaned IP or DNS records.
- Auto-remediation can shut down expired non-prod VMs after owner notice, but keep production actions manual.

## Building & Deployment Runbook

1. Build or refresh the image:
   ~~~bash
   packer init .
   packer build proxmox-ubuntu.pkr.hcl
   ~~~
2. Clone a smoke-test VM and verify day-0 config:
   ~~~bash
   terraform init
   terraform plan -var='template_id=9000'
   terraform apply -auto-approve
   cloud-init status --wait
   ~~~
3. Apply post-build hardening and app roles:
   ~~~bash
   ansible-playbook -i inventory/prod site.yml --limit smoke-test
   oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_server_l1 /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
   ~~~
4. Promote the template only after smoke test, vulnerability scan, and monitoring-agent validation succeed.
5. Release to production in rings, starting with low-risk workloads.
6. Validation gate: every provisioned VM must pass cloud-init, SSH, guest agent, monitoring, and hardening checks before the ticket is closed.

### Common mistakes to avoid during build

- Letting production teams bypass the pipeline “just once.”
- Treating snapshots as backups for long-term retention.
- Running no ownership review and then paying for idle VMs forever.
- Scanning images only after production deployment instead of in the promotion path.


## Cross-references

- Storage dependency: [04-shared-storage.md](./04-shared-storage.md)
- Linux operating model: [06-linux-os-layer.md](./06-linux-os-layer.md)
- Container host VMs later: [07-containers-and-monitoring.md](./07-containers-and-monitoring.md)
- Related repo reference: [../Virtual-Setup/08-ci-cd-and-automation.md](../Virtual-Setup/08-ci-cd-and-automation.md)
