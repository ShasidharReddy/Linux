# 01 - Virtualization Fundamentals
<div align="center"><pre>
┌──────────────────────────────────────────────────────────────┐
│                 Virtualization Fundamentals                 │
└──────────────────────────────────────────────────────────────┘
</pre></div>
This document explains the virtualization building blocks behind the rest of the virtual setup track. If you are planning to run the e-commerce platform first on virtual machines, read this file before [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md). If you eventually plan to move services into containers and Kubernetes, the same sizing, networking, and image-management ideas still matter.
## Goals
- understand Type 1 and Type 2 hypervisors
- install and manage KVM with libvirt
- create, clone, snapshot, and network VMs from the CLI
- understand basic vSphere and Proxmox workflows
- size virtual infrastructure for web, app, and database tiers
- create golden images with Packer for repeatable provisioning
## Hypervisor Types
Virtualization platforms fall into two broad groups.
### Type 1 Hypervisors
Type 1 hypervisors run directly on the hardware. They are commonly used in data centers and production virtualization clusters.
Examples:
- KVM on Linux hosts
- VMware ESXi
- Microsoft Hyper-V
- Xen-based enterprise distributions
Characteristics:
- better performance than desktop-oriented virtualization
- direct control over CPU, memory, storage, and networking
- better fit for clustering, live migration, HA, and production operations
- often integrated with centralized management platforms
### Type 2 Hypervisors
Type 2 hypervisors run on top of a host operating system such as Linux, Windows, or macOS.
Examples:
- VirtualBox
- VMware Workstation
- VMware Fusion
- Parallels Desktop
Characteristics:
- easier to start for labs and single-user machines
- ideal for local testing and OS experimentation
- lower operational complexity for small learning environments
- generally less suitable for production hosting at scale
## Architecture Comparison
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart LR
    subgraph T1[Type 1 Hypervisor]
        HW1[Physical Hardware] --> HV1[Hypervisor]
        HV1 --> VM11[VM 1]
        HV1 --> VM12[VM 2]
        HV1 --> VM13[VM 3]
    end
    subgraph T2[Type 2 Hypervisor]
        HW2[Physical Hardware] --> HOST[Host Operating System]
        HOST --> HV2[Hosted Hypervisor]
        HV2 --> VM21[VM A]
        HV2 --> VM22[VM B]
    end
~~~
## Choosing a Platform
| Platform | Category | Best For | Common Limitation |
| --- | --- | --- | --- |
| KVM + libvirt | Type 1-style on Linux | Labs, private cloud, production Linux virtualization | Requires Linux administration skills |
| ESXi + vCenter | Type 1 | Enterprise virtualization, clustering, mature management | Licensing and ecosystem cost |
| Hyper-V | Type 1 | Windows-centric environments | Linux workflows can feel secondary |
| VirtualBox | Type 2 | Desktop labs and quick demos | Not ideal for serious production use |
| VMware Workstation | Type 2 | Local dev/test and training | Single-host operational scope |
| Proxmox VE | Linux-based platform | Homelabs, SMB, mixed VM/LXC hosting | Feature depth differs from major enterprise stacks |
## KVM and libvirt Overview
KVM turns the Linux kernel into a hypervisor. libvirt provides APIs and tools such as `virsh`, `virt-install`, `virt-manager`, and storage/network abstractions. In practice:
- `qemu-kvm` provides virtualization and device emulation
- `libvirtd` or modular libvirt daemons manage objects
- `virsh` is the main CLI tool
- `virt-install` builds VMs from the command line
- `virt-customize` and cloud-init help automate guest initialization
## KVM/libvirt Stack
~~~mermaid
%%{init:{"theme":"neutral"}}%%
flowchart TD
    Admin[Administrator] --> Virsh[virsh / virt-install / virt-manager]
    Virsh --> Libvirt[libvirt API and daemons]
    Libvirt --> QEMU[QEMU]
    QEMU --> KVM[KVM kernel module]
    KVM --> HostKernel[Linux kernel]
    HostKernel --> Hardware[CPU Memory NIC Storage]
    Libvirt --> Networks[Virtual Networks]
    Libvirt --> Storage[Storage Pools and Volumes]
~~~
## Verifying Hardware Virtualization Support
Before installing packages, confirm the CPU exposes virtualization instructions.
### On Ubuntu or Debian
~~~bash
egrep -c '(vmx|svm)' /proc/cpuinfo
lscpu | grep Virtualization
kvm-ok
~~~
Expected result:
- Intel CPUs show `vmx`
- AMD CPUs show `svm`
- `kvm-ok` should report that KVM acceleration can be used
### On RHEL, Rocky, AlmaLinux, or CentOS Stream
~~~bash
egrep -c '(vmx|svm)' /proc/cpuinfo
lscpu | grep Virtualization
lsmod | grep kvm
~~~
If the count is `0`, check BIOS or UEFI settings and enable:
- Intel VT-x or Intel Virtualization Technology
- Intel VT-d if you need device passthrough
- AMD-V or SVM
- IOMMU for PCI passthrough use cases
## Installing KVM and libvirt
### Ubuntu 22.04 or 24.04
~~~bash
sudo apt-get update
sudo apt-get install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst bridge-utils cloud-image-utils cpu-checker
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
virsh --version
systemctl status libvirtd --no-pager
~~~
### RHEL 9 / Rocky / AlmaLinux 9
~~~bash
sudo dnf install -y @virt qemu-kvm virt-install libvirt libvirt-daemon-config-network bridge-utils guestfs-tools
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $USER
virsh --version
systemctl status libvirtd --no-pager
~~~
### Validate the Host
~~~bash
sudo virt-host-validate
~~~
Typical checks include:
- hardware virtualization support
- kernel modules such as `kvm_intel` or `kvm_amd`
- cgroup support
- bridge networking support
- device access for libvirt
## Basic libvirt Management
### List connections, hosts, and domains
~~~bash
virsh uri
virsh nodeinfo
virsh list --all
virsh net-list --all
virsh pool-list --all
~~~
### Useful service troubleshooting
~~~bash
sudo journalctl -u libvirtd -n 100 --no-pager
sudo ss -lntp | grep libvirt
sudo systemctl restart libvirtd
~~~
Common pitfall:
- If `virsh list` works only with `sudo`, your user may not be in the `libvirt` group yet, or you need a fresh login shell.
## Creating a VM with virt-install
The following example creates an Ubuntu VM from an ISO and attaches it to the default NAT network.
~~~bash
sudo mkdir -p /var/lib/libvirt/boot
sudo cp ~/Downloads/ubuntu-24.04-live-server-amd64.iso /var/lib/libvirt/boot/
virt-install \
  --name web01 \
  --memory 4096 \
  --vcpus 2 \
  --cpu host-passthrough \
  --disk path=/var/lib/libvirt/images/web01.qcow2,size=40,format=qcow2,bus=virtio \
  --os-variant ubuntu24.04 \
  --network network=default,model=virtio \
  --graphics none \
  --console pty,target_type=serial \
  --cdrom /var/lib/libvirt/boot/ubuntu-24.04-live-server-amd64.iso
~~~
Notes:
- `--graphics none` is useful for SSH- or terminal-driven labs
- `--cpu host-passthrough` maximizes guest CPU feature exposure
- `qcow2` supports snapshots and sparse allocation
- `virtio` usually gives better performance than emulated devices
### Unattended-style build with cloud-init seed ISO
~~~bash
cat > user-data <<'EOF'
#cloud-config
hostname: app01
manage_etc_hosts: true
users:
  - name: devops
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    shell: /bin/bash
    lock_passwd: false
    passwd: "$6$rounds=4096$example$examplehashedpassword"
ssh_pwauth: true
package_update: true
packages:
  - qemu-guest-agent
  - curl
  - vim
runcmd:
  - systemctl enable --now qemu-guest-agent
EOF
cat > meta-data <<'EOF'
instance-id: app01
local-hostname: app01
EOF
cloud-localds seed-app01.iso user-data meta-data
virt-install \
  --name app01 \
  --memory 8192 \
  --vcpus 4 \
  --cpu host-passthrough \
  --import \
  --disk path=/var/lib/libvirt/images/ubuntu-2404-template.qcow2,device=disk,format=qcow2,bus=virtio \
  --disk path=$(pwd)/seed-app01.iso,device=cdrom \
  --network network=default,model=virtio \
  --os-variant ubuntu24.04 \
  --graphics none \
  --noautoconsole
~~~
Practical tip:
- For repeated builds, keep a clean base image and generate a unique cloud-init seed per VM.
## virsh Lifecycle Commands
### Power operations
~~~bash
virsh start web01
virsh shutdown web01
virsh reboot web01
virsh destroy web01
virsh undefine web01
~~~
Use cases:
- `shutdown` asks the guest OS to shut down cleanly
- `destroy` is equivalent to pulling power; use it only when a VM is stuck
- `undefine` removes the libvirt definition but does not necessarily remove disks
### Information and console access
~~~bash
virsh dominfo web01
virsh domifaddr web01
virsh console web01
virsh domblklist web01
virsh dumpxml web01 > web01.xml
~~~
### Snapshot management
~~~bash
virsh snapshot-create-as web01 pre-nginx-install "Before installing Nginx"
virsh snapshot-list web01
virsh snapshot-revert web01 pre-nginx-install
virsh snapshot-delete web01 pre-nginx-install
~~~
Important note:
- Internal snapshots can affect performance and complicate backup. In production, external snapshots and image-level backup tools are often preferred.
### Cloning a VM
~~~bash
virt-clone \
  --original ubuntu-template \
  --name web02 \
  --file /var/lib/libvirt/images/web02.qcow2
~~~
After cloning, confirm:
- a unique hostname
- unique SSH host keys if required by policy
- correct static IP assignment or DHCP reservation
- cloud-init state is cleaned if a template was reused incorrectly
## Networking Modes
Virtual networking design matters for an e-commerce stack because each tier has different exposure and security requirements.
### NAT Network
The default libvirt network usually provides NAT via `virbr0`.
Pros:
- fast to set up
- private addressing
- outbound internet access for guests
Cons:
- inbound access requires port forwarding or a reverse proxy
- less natural for multi-host east-west traffic
Inspect default network:
~~~bash
virsh net-info default
virsh net-dumpxml default
ip addr show virbr0
~~~
### Bridge Network
Bridge networking places VMs directly on the same L2 segment as the host network.
Ubuntu bridge example with Netplan:
~~~yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: false
  bridges:
    br0:
      interfaces: [eno1]
      dhcp4: true
      parameters:
        stp: false
        forward-delay: 0
~~~
Apply it:
~~~bash
sudo netplan apply
ip addr show br0
~~~
Attach a VM to the bridge:
~~~bash
virt-install \
  --name web-bridge \
  --memory 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/web-bridge.qcow2,size=30 \
  --os-variant ubuntu24.04 \
  --network bridge=br0,model=virtio \
  --graphics none \
  --cdrom /var/lib/libvirt/boot/ubuntu-24.04-live-server-amd64.iso
~~~
### macvtap Network
macvtap gives guests near-direct attachment to a physical NIC without a Linux bridge.
Example:
~~~bash
virt-install \
  --name app-macvtap \
  --memory 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/app-macvtap.qcow2,size=30 \
  --os-variant ubuntu24.04 \
  --network type=direct,source=eno1,source_mode=bridge,model=virtio \
  --graphics none \
  --cdrom /var/lib/libvirt/boot/ubuntu-24.04-live-server-amd64.iso
~~~
Caution:
- macvtap guests may not be able to talk directly to the host on the same interface depending on the chosen mode.
## Creating Custom Virtual Networks
Network XML for an isolated backend network:
~~~xml
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
~~~
Apply it:
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
virsh net-list --all
~~~
## Storage Pools and Volumes
libvirt storage pools make it easier to manage images consistently.
### Directory-based pool
~~~bash
sudo mkdir -p /vmstore/images
virsh pool-define-as ecommerce-images dir --target /vmstore/images
virsh pool-build ecommerce-images
virsh pool-start ecommerce-images
virsh pool-autostart ecommerce-images
virsh pool-info ecommerce-images
~~~
Create a volume:
~~~bash
virsh vol-create-as ecommerce-images web03.qcow2 40G --format qcow2
virsh vol-list ecommerce-images
virsh vol-path web03.qcow2 --pool ecommerce-images
~~~
### LVM-backed pool
~~~bash
sudo pvcreate /dev/nvme1n1
sudo vgcreate vg_vmdata /dev/nvme1n1
virsh pool-define-as vm-lvm logical --source-name vg_vmdata --target /dev/vg_vmdata
virsh pool-build vm-lvm
virsh pool-start vm-lvm
virsh pool-autostart vm-lvm
~~~
Advantages:
- fast block storage access
- useful for databases and high-I/O guests
- integrates with snapshots at the LVM layer if planned carefully
### Resizing a disk
~~~bash
qemu-img info /var/lib/libvirt/images/db01.qcow2
qemu-img resize /var/lib/libvirt/images/db01.qcow2 +50G
virsh blockresize db01 vda 150G
~~~
Guest-side reminder:
- resizing the virtual disk is only step one
- inside the guest you still need partition, filesystem, or LVM expansion
## VM Resource Sizing for E-Commerce Tiers
A practical lab or production design should size VMs differently based on workload patterns.
### Baseline tier sizing
| Tier | vCPU | RAM | Storage | Notes |
| --- | --- | --- | --- | --- |
| Web | 2 | 4 GB | 40 GB | Nginx, static content, TLS termination in small setups |
| App | 4 | 8 GB | 60 GB | PHP-FPM or Node.js, workers, app logs |
| DB | 8 | 32 GB | 200 GB SSD/NVMe | MySQL primary or replica with binary logs |
### Detailed sizing by scale
| Scale | Web Nodes | App Nodes | DB Nodes | Cache | Search | Approx Visitors/Day |
| --- | --- | --- | --- | --- | --- | --- |
| Lab | 1 x 2 vCPU / 4 GB | 1 x 2 vCPU / 4 GB | 1 x 4 vCPU / 8 GB | shared | optional | < 1,000 |
| Small | 2 x 2 vCPU / 4 GB | 2 x 4 vCPU / 8 GB | 1 x 8 vCPU / 16 GB | 1 x 2 vCPU / 2 GB | optional | 1,000 - 10,000 |
| Medium | 2 x 4 vCPU / 8 GB | 2-4 x 4 vCPU / 8 GB | 1 primary + 1 replica, each 8 vCPU / 32 GB | 1-2 x 2 vCPU / 4 GB | 3 x 4 vCPU / 8 GB | 10,000 - 100,000 |
| Large | 4+ x 4 vCPU / 8 GB | 4+ x 8 vCPU / 16 GB | clustered or HA DB | dedicated Redis cluster | 3-6 node search cluster | 100,000+ |
Sizing guidance:
- prioritize RAM for MySQL buffer pools and Elasticsearch heap
- watch CPU steal time on oversubscribed hosts
- put database and search storage on low-latency media
- separate noisy neighbors by pool, host, or cluster
## Example Host Capacity Planning
| Host | Physical CPU | RAM | Recommended VM Mix |
| --- | --- | --- | --- |
| Lab host | 8 cores | 32 GB | 2 web, 2 app, 1 DB, 1 utility VM |
| Mid-size host | 16 cores | 64 GB | 4 web, 4 app, 1 DB, 1 replica, 2 infra VMs |
| Data node host | 24 cores | 128 GB | 2 DB or search-heavy VMs with reserved I/O |
Practical rule:
- leave 10-20% CPU and memory headroom on the hypervisor for host services, page cache, live migration, and burst conditions
## vSphere Basics
VMware vSphere combines ESXi hypervisors with vCenter management.
### ESXi host setup overview
Typical workflow:
1. Install ESXi on physical servers.
2. Configure management networking and DNS.
3. Add local or shared datastores.
4. Join hosts to vCenter.
5. Place hosts into clusters.
6. Configure DRS, HA, and vMotion if licensed.
Key concepts:
- ESXi host: the physical hypervisor node
- datastore: VMFS, NFS, vSAN, or other storage backing VMs
- port group: network definition used by VMs
- template: a reusable golden image
- resource pool: a logical reservation/limit boundary
### vCenter management
vCenter gives you:
- inventory and RBAC
- centralized template cloning
- cluster management
- alarms and task history
- distributed virtual switches
- lifecycle operations across many hosts
Common admin actions in vSphere:
- create a datacenter and cluster
- add ESXi hosts
- create standard or distributed switches
- upload ISO images
- build a VM template
- clone from template into web, app, and DB roles
### VM templates and cloning
Typical template workflow:
1. Build a clean Linux VM.
2. Install guest tools.
3. Patch packages.
4. Clear temporary host identity data.
5. Convert the VM to a template.
6. Clone from template for each environment.
Best practices:
- keep one template per OS major version
- update templates monthly with security patches
- install cloud-init if supported in your image pipeline
- document the software baseline in the template description
## Proxmox VE Overview
Proxmox VE is a Debian-based virtualization platform that manages both KVM virtual machines and LXC containers.
### Why teams use Proxmox
- web UI plus CLI access
- integrated clustering for small and medium environments
- built-in storage options like ZFS and Ceph integration
- useful for labs, homelabs, and SMB production deployments
### Quick setup outline
1. Install Proxmox VE from ISO.
2. Configure the management bridge, usually `vmbr0`.
3. Upload ISO images or cloud images.
4. Create a VM template.
5. Clone VMs for web, app, and DB roles.
CLI examples on a Proxmox host:
~~~bash
pveversion
qm list
qm config 9000
qm clone 9000 201 --name web01
qm start 201
~~~
If you want enterprise-style clustering with easier setup than some large vendor stacks, Proxmox is a practical option for this track.
## Golden Images with Packer
Golden images reduce drift and speed up provisioning. For the e-commerce platform, a base Ubuntu or Rocky image with security updates, guest agent, and baseline packages is ideal.
### Packer workflow
1. start from a cloud image or unattended installer ISO
2. run shell provisioners
3. install guest agent and core packages
4. clean package cache and SSH host-specific state
5. export the qcow2 artifact
6. import or copy it into a libvirt storage pool
### Example `packer.json`
~~~json
{
  "builders": [
    {
      "type": "qemu",
      "name": "ubuntu-2404-kvm",
      "iso_url": "https://cloud-images.ubuntu.com/releases/24.04/release/ubuntu-24.04-server-cloudimg-amd64.img",
      "iso_checksum": "file:https://cloud-images.ubuntu.com/releases/24.04/release/SHA256SUMS",
      "output_directory": "output-ubuntu2404",
      "disk_image": true,
      "format": "qcow2",
      "accelerator": "kvm",
      "memory": 4096,
      "cpus": 2,
      "headless": true,
      "ssh_username": "ubuntu",
      "ssh_timeout": "30m",
      "ssh_private_key_file": "~/.ssh/id_rsa",
      "vm_name": "ubuntu-2404-template.qcow2",
      "net_device": "virtio-net",
      "disk_interface": "virtio",
      "shutdown_command": "sudo cloud-init clean && sudo systemctl poweroff"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "sudo apt-get update",
        "sudo apt-get dist-upgrade -y",
        "sudo apt-get install -y qemu-guest-agent curl vim git ca-certificates",
        "sudo systemctl enable qemu-guest-agent",
        "sudo truncate -s 0 /etc/machine-id",
        "sudo rm -f /var/lib/dbus/machine-id",
        "sudo apt-get clean"
      ]
    }
  ]
}
~~~
Build it:
~~~bash
packer init .
packer validate packer.json
packer build packer.json
~~~
Move the output image into libvirt storage:
~~~bash
sudo cp output-ubuntu2404/ubuntu-2404-template.qcow2 /var/lib/libvirt/images/
qemu-img info /var/lib/libvirt/images/ubuntu-2404-template.qcow2
~~~
## Template Hygiene Checklist
Before converting any VM into a template:
- remove shell history if policy requires it
- rotate or remove SSH host keys if re-generated on first boot
- clear `/etc/machine-id`
- clean package cache
- stop application-specific services
- document default credentials or disable them
- ensure cloud-init or sysprep tooling is configured correctly
## Common virsh Operations Cheat Sheet
~~~bash
virsh list --all
virsh start app01
virsh shutdown app01
virsh reboot app01
virsh suspend app01
virsh resume app01
virsh domifaddr app01
virsh dumpxml app01
virsh snapshot-list app01
virsh pool-list --all
virsh vol-list ecommerce-images
virsh net-list --all
~~~
## Troubleshooting Checklist
### VM will not start
Check:
~~~bash
virsh start web01
journalctl -xe --no-pager | tail -n 50
virsh dominfo web01
~~~
Possible causes:
- missing KVM module
- storage file path changed or permissions broken
- bridge interface missing
- insufficient host memory
- unsupported CPU mode after host changes
### Guest has no network
Check:
~~~bash
virsh domiflist web01
virsh net-list --all
ip addr show virbr0
sudo nft list ruleset | less
~~~
Possible causes:
- custom libvirt network not started
- bridge not attached to a physical NIC
- DHCP disabled without static guest configuration
- firewall blocking forwarding or DNS
### Poor performance
Check:
~~~bash
top
vmstat 1 5
iostat -xz 1 5
virsh domstats db01
~~~
Tune:
- use virtio devices
- pin vCPUs for critical workloads if needed
- isolate database workloads from web tiers
- move high-I/O guests to faster storage pools
- avoid excessive snapshot chains
## Practical Design Advice for the E-Commerce Track
- Put web VMs on a frontend bridge or routed segment.
- Put app and database VMs on private networks.
- Keep database disks on dedicated storage pools.
- Use templates for every repeated node role.
- Snapshot only around high-risk changes, not as a permanent backup strategy.
- Use automation from day one, even in the lab.
## Suggested Build Sequence
1. prepare host BIOS and package dependencies
2. configure libvirt and validate KVM
3. build one golden image with Packer
4. define frontend, backend, and data networks
5. clone web, app, and DB VMs from the template
6. layer configuration management or cloud-init on top
This sequence leads directly into [02-vm-based-ecommerce-setup.md](./02-vm-based-ecommerce-setup.md), where the full six-VM commerce environment is provisioned and configured. If you want to compare this VM-first model with containers, continue later with [03-docker-fundamentals.md](./03-docker-fundamentals.md).
## Common Pitfalls
- treating snapshots as backups
- forgetting storage pool autostart on reboot
- building templates without cleaning unique host identity artifacts
- oversubscribing memory on the hypervisor and causing swap pressure
- using one flat network for frontend, backend, and database traffic
- sizing MySQL and Elasticsearch like ordinary app servers
## Summary
Virtualization gives you isolation, predictable resource boundaries, and clean migration paths from physical infrastructure. KVM with libvirt is an excellent place to learn the mechanics because it is scriptable, production-capable, and directly useful in labs and private cloud environments. vSphere and Proxmox add broader platform management models, but the same fundamentals still apply.
[← Back to Virtual Setup](./README.md)
