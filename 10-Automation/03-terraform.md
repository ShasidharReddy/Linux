# Terraform for Linux

[Back to guide index](README.md)

Terraform is an Infrastructure as Code tool used to provision and manage infrastructure resources declaratively.

For Linux automation, Terraform commonly manages:

- Compute instances
- Networks
- Security groups
- Disks and volumes
- DNS records
- Load balancers
- IAM and access policies

## 3.1 Terraform Core Concepts

| Concept | Description |
|---|---|
| Provider | Plugin that talks to a platform API |
| Resource | Managed infrastructure object |
| Data source | Read-only lookup of external data |
| Module | Reusable set of Terraform code |
| State | Mapping between config and real resources |
| Plan | Preview of proposed changes |
| Apply | Execution of changes |
| Output | Exposed values from a configuration |

## 3.2 Why Terraform for Linux Environments

Terraform is not a replacement for all configuration management.

It is strongest at provisioning infrastructure.

Examples:

- Create a VPC and subnets
- Launch Linux instances
- Attach storage volumes
- Create security groups
- Generate DNS records for services

Then a tool like Ansible or Cloud-Init can perform instance configuration.

## 3.3 Installation

### Example on Linux

```bash
curl -fsSL https://releases.hashicorp.com/terraform/1.9.0/terraform_1.9.0_linux_amd64.zip -o terraform.zip
unzip terraform.zip
sudo install terraform /usr/local/bin/terraform
terraform version
```

## 3.4 Basic Workflow

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

## 3.5 Provider Configuration

### AWS example

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

### Variables file

```hcl
variable "aws_region" {
  type        = string
  description = "AWS region"
  default     = "us-east-1"
}
```

## 3.6 Resources

A resource represents something Terraform manages.

### Example: security group

```hcl
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## 3.7 Example: Linux Compute Instance

```hcl
resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public_a.id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = var.key_name
  user_data              = file("cloud-init/web.yaml")

  tags = {
    Name = "web-01"
    Role = "web"
    Env  = var.environment
  }
}
```

## 3.8 Networking Example

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.10.0.0/16"
  tags = {
    Name = "prod-vpc"
  }
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.10.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "prod-public-a"
  }
}
```

## 3.9 Provisioners

Provisioners can run commands or copy files, but should be used sparingly.

HashiCorp generally recommends relying more on image building and configuration management than provisioners.

### remote-exec example

```hcl
resource "null_resource" "bootstrap" {
  triggers = {
    instance_id = aws_instance.web.id
  }

  connection {
    type        = "ssh"
    host        = aws_instance.web.public_ip
    user        = "ec2-user"
    private_key = file(var.private_key_path)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo dnf install -y nginx",
      "sudo systemctl enable --now nginx"
    ]
  }
}
```

### file provisioner example

```hcl
provisioner "file" {
  source      = "files/app.conf"
  destination = "/home/ec2-user/app.conf"
}
```

## 3.10 Terraform Modules

Modules allow reuse and standardization.

### Root module calling a child module

```hcl
module "web_stack" {
  source        = "./modules/web-stack"
  vpc_cidr      = "10.20.0.0/16"
  subnet_cidr   = "10.20.1.0/24"
  instance_type = "t3.micro"
  environment   = "stage"
}
```

### Example module inputs

```hcl
variable "vpc_cidr" {
  type = string
}

variable "subnet_cidr" {
  type = string
}

variable "instance_type" {
  type = string
}
```

## 3.11 State Management

State is critical in Terraform.

It records the relationship between your code and real infrastructure.

### Local state

Suitable for experiments or very small projects.

### Remote state

Recommended for teams.

Benefits:

- Centralized state storage
- Locking support
- Better collaboration
- Reduced risk of conflicting applies

### Example remote backend on S3

```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "linux/prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

## 3.12 Workspaces

Workspaces can separate state instances but are not always the best way to model environments.

For larger organizations, separate directories or repositories per environment are often clearer.

## 3.13 Outputs

```hcl
output "web_public_ip" {
  value = aws_instance.web.public_ip
}
```

## 3.14 Data Sources

Data sources fetch existing information.

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

## 3.15 Variables and tfvars

### variables.tf

```hcl
variable "environment" {
  type = string
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

### prod.tfvars

```hcl
environment   = "prod"
instance_type = "t3.small"
```

### Run with tfvars

```bash
terraform plan -var-file=prod.tfvars
terraform apply -var-file=prod.tfvars
```

## 3.16 Locals

```hcl
locals {
  name_prefix = "${var.environment}-web"
  common_tags = {
    Env     = var.environment
    Managed = "terraform"
  }
}
```

## 3.17 Dependency Graph

Terraform builds a dependency graph automatically from references.

You can use explicit `depends_on` only when needed.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public_a.id

  depends_on = [aws_internet_gateway.main]
}
```

## 3.18 Lifecycle Meta-Arguments

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type

  lifecycle {
    create_before_destroy = true
  }
}
```

Useful lifecycle controls:

- create_before_destroy
- prevent_destroy
- ignore_changes

Use ignore_changes carefully to avoid masking drift.

## 3.19 for_each and count

### for_each example

```hcl
variable "web_hosts" {
  type = map(string)
  default = {
    web01 = "t3.micro"
    web02 = "t3.micro"
  }
}

resource "aws_instance" "web" {
  for_each      = var.web_hosts
  ami           = data.aws_ami.amazon_linux.id
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

## 3.20 Example Repository Layout

```text
terraform/
├── main.tf
├── providers.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── prod.tfvars
├── stage.tfvars
├── modules/
│   ├── network/
│   └── compute/
└── cloud-init/
    └── web.yaml
```

## 3.21 Terraform and Linux Bootstrap

A common pattern:

1. Terraform creates the instance.
2. Cloud-Init bootstraps the OS.
3. Ansible applies full configuration.
4. Monitoring validates health.

## 3.22 Example: Instance with Cloud-Init

```hcl
resource "aws_instance" "app" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t3.small"
  subnet_id              = aws_subnet.public_a.id
  vpc_security_group_ids = [aws_security_group.web.id]
  user_data              = file("cloud-init/app.yaml")

  tags = {
    Name = "app-01"
  }
}
```

### app.yaml

```yaml
#cloud-config
package_update: true
packages:
  - nginx
write_files:
  - path: /var/www/html/index.html
    content: |
      Provisioned with Terraform and Cloud-Init
runcmd:
  - systemctl enable --now nginx
```

## 3.23 State Security

Protect Terraform state because it may contain:

- Resource IDs
- IP addresses
- Potentially sensitive rendered values
- Provider metadata

Best practices:

- Use encrypted remote backends.
- Restrict backend access.
- Avoid storing secrets in plain variables.
- Use secret managers and provider integrations.

## 3.24 Importing Existing Resources

If infrastructure already exists, Terraform can import resources.

```bash
terraform import aws_instance.web i-0123456789abcdef0
```

After import, write matching configuration and validate the plan carefully.

## 3.25 Plan Review Practices

Never apply unreviewed changes in production.

Recommended workflow:

1. Run `terraform fmt`.
2. Run `terraform validate`.
3. Generate a plan.
4. Review the plan in CI.
5. Require approval.
6. Apply with controlled credentials.

## 3.26 Testing and Policy

Common checks:

- terraform fmt -check
- terraform validate
- tflint
- tfsec or other security scanners
- OPA or Sentinel policy checks
- Integration tests in ephemeral environments

## 3.27 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| Manual cloud console changes | Drift | Import or revert through code |
| Shared local state | Corruption risk | Use remote backend with locking |
| Overuse of provisioners | Fragile builds | Prefer images and config management |
| Giant root module | Hard reuse | Split into modules |
| Sensitive values in outputs | Exposure | Mark outputs sensitive and restrict access |

## 3.28 Example Production Pattern

- Network module creates VPC and subnets.
- Security module defines security groups.
- Compute module creates instances.
- Cloud-Init performs first boot.
- Ansible configures applications.
- CI runs plan on pull request and apply on approval.

## 3.29 Terraform Summary

Terraform is the provisioning backbone for many Linux environments.

Use it to manage infrastructure resources predictably, and pair it with configuration tools for full lifecycle automation.

---
