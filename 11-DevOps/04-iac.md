# Infrastructure as Code

[Back to guide index](README.md)

### 4.1 Why IaC matters in DevOps
Infrastructure as Code makes infrastructure versioned, reviewable, testable, and reproducible. Linux is the main execution environment for IaC tooling in CI pipelines and operations workflows.

### 4.2 Terraform overview
Terraform defines infrastructure in HCL and uses providers to manage cloud or platform resources.

Basic workflow:
```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

### 4.3 Terraform project structure
```text
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── versions.tf
├── terraform.tfvars
└── modules/
```

### 4.4 Terraform example
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
  region = var.region
}

resource "aws_s3_bucket" "artifacts" {
  bucket = var.bucket_name
}
```

### 4.5 Terraform state management
State is critical and sensitive.

Best practices:
- Use remote backends.
- Enable locking.
- Encrypt at rest.
- Restrict access.
- Never commit state files.

Popular backends:
- S3 + DynamoDB lock table.
- Terraform Cloud.
- Azure Storage.
- GCS.

### 4.6 Terraform Linux provisioning patterns
Common techniques:
- Use `cloud-init` to bootstrap instances.
- Pass startup scripts with metadata.
- Use config management after provisioning.
- Keep instance bootstrap idempotent.

Example user data:
```bash
#!/usr/bin/env bash
set -euo pipefail
apt update
apt install -y nginx
systemctl enable --now nginx
```

### 4.7 Terraform module practices
- Keep modules focused.
- Version modules.
- Document inputs/outputs.
- Avoid hidden behavior.
- Use `locals` for clarity.

### 4.8 CloudFormation overview
CloudFormation is AWS-native IaC using YAML or JSON templates.

Example snippet:
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-devops-artifacts
```

Strengths:
- Native AWS integration.
- Stack drift detection.
- Change sets.

Trade-offs:
- AWS-only.
- Verbose templates for complex cases.

### 4.9 Pulumi overview
Pulumi uses general-purpose languages like TypeScript, Python, Go, or C# to define infrastructure.

TypeScript example:
```typescript
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.Bucket("artifacts", {
  bucket: "my-devops-artifacts",
});
```

### 4.10 IaC comparison table
| Tool | Language | Best Fit | Notes |
|---|---|---|---|
| Terraform | HCL | Multi-cloud infrastructure | Large ecosystem |
| CloudFormation | YAML/JSON | AWS-native environments | Strong AWS support |
| Pulumi | TS/Python/Go/C# | Teams preferring general languages | Good developer ergonomics |

### 4.11 Linux provisioning layers
A practical stack often looks like:
1. Terraform creates VM, network, disk, IAM.
2. Cloud-init configures packages and users.
3. Ansible or scripts apply software config.
4. systemd manages runtime services.
5. Monitoring agents are installed at bootstrap.

### 4.12 Example cloud-init file
```yaml
#cloud-config
package_update: true
packages:
  - nginx
  - curl
write_files:
  - path: /etc/motd
    content: |
      Managed by cloud-init
runcmd:
  - systemctl enable --now nginx
```

### 4.13 Drift management
Drift happens when manual changes differ from declared state.

Mitigation:
- Limit manual changes.
- Use drift detection tools.
- Reconcile through pipelines.
- Review emergency changes post-incident.

### 4.14 Policy as code
Policy tools enforce guardrails such as:
- Required tags.
- Encryption enabled.
- No public buckets.
- Allowed instance types.

Examples:
- Open Policy Agent.
- Sentinel.
- Conftest.

### 4.15 IaC testing strategies
- `terraform validate`
- `terraform plan` in CI
- Static analysis with Checkov or tfsec
- Terratest for integration testing
- Policy tests

### 4.16 Secret management in IaC
- Avoid inline secrets in code.
- Use secret stores or encrypted values.
- Mark sensitive outputs.
- Limit state exposure.

### 4.17 Reusable Linux server baseline module
A reusable module often includes:
- Standard users/groups.
- SSH hardening.
- Cloud-init.
- Logging agent.
- Monitoring agent.
- Firewall rules.
- Common tags.

### 4.18 Example Terraform variable definitions
```hcl
variable "region" {
  type        = string
  description = "AWS region"
}

variable "bucket_name" {
  type        = string
  description = "Artifact bucket name"
}
```

### 4.19 Pipeline for IaC delivery
A safe IaC pipeline usually includes:
- Formatting check.
- Validation.
- Security scan.
- Plan artifact.
- Approval.
- Apply from controlled environment.

### 4.20 Production IaC checklist
- Remote state locked and encrypted.
- Least privilege credentials.
- Modules versioned.
- Review required.
- Cost visibility.
- Rollback or remediation strategy documented.

---
