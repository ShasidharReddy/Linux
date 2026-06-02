# Packer

[Back to guide index](README.md)

Packer builds machine images from automated templates.

For Linux environments, Packer is used to create:

- Cloud VM images
- Virtual machine templates
- Golden AMIs
- Base images with security baselines
- Prebaked application images

## 8.1 Why Packer Matters

Golden images reduce:

- Provisioning time
- Configuration drift
- Dependency on long bootstrap scripts
- Patch inconsistency across instances

## 8.2 Core Concepts

| Concept | Description |
|---|---|
| Builder | Creates the base machine image |
| Provisioner | Configures the image during build |
| Post-processor | Transforms or publishes artifacts |
| Source | Declares the image source definition |
| Template | HCL2 or JSON build definition |

## 8.3 HCL2 Template Structure

```hcl
packer {
  required_plugins {
    amazon = {
      source  = "github.com/hashicorp/amazon"
      version = ">= 1.2.0"
    }
  }
}

source "amazon-ebs" "al2023" {
  region        = "us-east-1"
  instance_type = "t3.micro"
  ssh_username  = "ec2-user"
  ami_name      = "al2023-nginx-{{timestamp}}"
  source_ami_filter {
    filters = {
      name                = "al2023-ami-*-x86_64"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    owners      = ["amazon"]
    most_recent = true
  }
}

build {
  name    = "linux-nginx"
  sources = ["source.amazon-ebs.al2023"]

  provisioner "shell" {
    inline = [
      "sudo dnf update -y",
      "sudo dnf install -y nginx",
      "sudo systemctl enable nginx"
    ]
  }
}
```

## 8.4 Builders

Common builders:

- amazon-ebs
- azure-arm
- googlecompute
- vmware-iso
- virtualbox-iso
- qemu

## 8.5 Provisioners

Common provisioners:

- shell
- ansible
- ansible-local
- file
- powershell

### shell provisioner example

```hcl
provisioner "shell" {
  script = "scripts/hardening.sh"
}
```

### ansible provisioner example

```hcl
provisioner "ansible" {
  playbook_file = "ansible/playbooks/image.yml"
}
```

## 8.6 Post-Processors

Post-processors can:

- Compress artifacts
- Create manifests
- Publish images
- Chain outputs to downstream tools

## 8.7 Example: Baseline Linux Image

```hcl
build {
  name    = "baseline-linux"
  sources = ["source.amazon-ebs.al2023"]

  provisioner "shell" {
    inline = [
      "sudo dnf update -y",
      "sudo dnf install -y curl vim git",
      "sudo dnf clean all"
    ]
  }

  provisioner "shell" {
    inline = [
      "sudo useradd -r -s /sbin/nologin node_exporter || true"
    ]
  }

  post-processor "manifest" {
    output = "manifest.json"
  }
}
```

## 8.8 Packer Workflow

```bash
packer init .
packer fmt .
packer validate .
packer build image.pkr.hcl
```

## 8.9 Image Strategy

Use Packer when you want:

- Faster instance launch times
- Pre-installed packages and agents
- Reduced first-boot complexity
- More controlled immutable deployments

## 8.10 Packer and Ansible

A common pattern is using Ansible as the image provisioner.

Benefits:

- Reuse configuration logic
- Standardize image content
- Validate roles before runtime

## 8.11 Best Practices for Packer

1. Keep images minimal but useful.
2. Apply security baseline packages and config.
3. Clean package caches and temporary files.
4. Tag images clearly with version and build date.
5. Test images before promotion.
6. Avoid embedding environment-specific secrets.
7. Use CI to build on merge, not manually.

## 8.12 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| Bloated images | Slow builds and patching | Keep images focused |
| Embedding secrets | Security incident | Fetch secrets at runtime |
| Unvalidated image changes | Broken fleet rollout | Test before publish |
| Long shell scripts | Hard maintenance | Use Ansible or modular scripts |

## 8.13 Packer Summary

Packer is the image factory for Linux automation, especially valuable in immutable and large-scale provisioning models.

---
