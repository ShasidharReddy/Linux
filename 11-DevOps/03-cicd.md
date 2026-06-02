# CI/CD on Linux

[Back to guide index](README.md)

### 3.1 Why CI/CD relies on Linux
CI/CD systems frequently run on Linux because Linux offers strong automation, package availability, shell scripting, container support, and lower overhead for ephemeral build agents.

### 3.2 CI/CD core concepts
- Continuous Integration: merge and validate code frequently.
- Continuous Delivery: keep software releasable.
- Continuous Deployment: automatically deploy every validated change.

### 3.3 CI/CD pipeline stages
```mermaid
graph LR
A["Source Commit"] --> B["Build"]
B --> C["Unit Tests"]
C --> D["Security Scan"]
D --> E["Package Artifact"]
E --> F["Deploy to Staging"]
F --> G["Integration Tests"]
G --> H["Approval Gate"]
H --> I["Deploy to Production"]
I --> J["Monitor and Rollback Logic"]
```

### 3.4 Linux build agent responsibilities
A Linux build agent commonly handles:
- Source checkout.
- Dependency installation.
- Build tool execution.
- Test execution.
- Container image builds.
- Artifact upload.
- Secrets retrieval.
- Deployment orchestration.

### 3.5 Jenkins installation on Linux
Ubuntu example:
```bash
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre curl
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
```

### 3.6 Jenkins service operations
```bash
sudo systemctl status jenkins
sudo journalctl -u jenkins -n 200 --no-pager
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 3.7 Jenkins pipeline concepts
- Controller manages orchestration.
- Agents execute jobs.
- Pipelines are code in `Jenkinsfile`.
- Plugins add SCM, build, scan, and deploy integrations.

### 3.8 Declarative Jenkinsfile example
```groovy
pipeline {
  agent any

  environment {
    REGISTRY = 'registry.example.com'
    IMAGE = 'myapp'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'make build'
      }
    }

    stage('Test') {
      steps {
        sh 'make test'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .'
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'registry-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login $REGISTRY -u $USER --password-stdin'
          sh 'docker push $REGISTRY/$IMAGE:$BUILD_NUMBER'
        }
      }
    }
  }
}
```

### 3.9 Scripted Jenkins pipeline example
```groovy
node {
  stage('Checkout') {
    checkout scm
  }
  stage('Build') {
    sh 'npm ci'
    sh 'npm run build'
  }
  stage('Test') {
    sh 'npm test'
  }
}
```

### 3.10 Jenkins agent models
| Agent Type | Description | Best Use |
|---|---|---|
| Static VM | Persistent Linux node | Stable enterprise workloads |
| Dynamic container | Ephemeral container agents | Scale-out parallel jobs |
| Kubernetes agent | Pod-per-build model | Cloud-native CI |
| Bare metal | Direct host execution | Specialized toolchains |

### 3.11 Jenkins hardening tips
- Use HTTPS and reverse proxy.
- Disable unnecessary plugins.
- Store secrets in credential store.
- Back up `$JENKINS_HOME`.
- Run agents with least privilege.
- Prefer ephemeral agents.
- Restrict shell access on shared builders.

### 3.12 GitHub Actions on Linux
GitHub Actions uses workflows defined in `.github/workflows/*.yml`. Linux runners are common for builds, tests, packaging, and deployment.

Example workflow:
```yaml
name: ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
```

### 3.13 Self-hosted GitHub Actions runners on Linux
Why self-hosted runners:
- Custom network access.
- Specialized dependencies.
- Faster builds with cached tooling.
- Compliance requirements.

Setup outline:
1. Create a runner in repository/org settings.
2. Download runner package.
3. Configure runner with token.
4. Install as a service.

```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/v2.317.0/actions-runner-linux-x64-2.317.0.tar.gz
tar xzf ./actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/org/repo --token RUNNER_TOKEN
sudo ./svc.sh install
sudo ./svc.sh start
```

### 3.14 Self-hosted runner operational guidance
- Use dedicated service accounts.
- Isolate runners per trust level.
- Avoid sharing privileged runners across teams.
- Auto-scale ephemeral runners for safer workloads.
- Clean workspaces between jobs.

### 3.15 GitLab CI on Linux
GitLab CI uses `.gitlab-ci.yml` and GitLab Runner. Linux runners can use shell, Docker, or Kubernetes executors.

Example `.gitlab-ci.yml`:
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - make build

unit_test:
  stage: test
  script:
    - make test

deploy_prod:
  stage: deploy
  script:
    - ./deploy.sh
  when: manual
  only:
    - main
```

### 3.16 GitLab Runner installation
```bash
curl -L --output gitlab-runner.deb https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb
sudo dpkg -i gitlab-runner.deb
sudo gitlab-runner register
sudo systemctl enable --now gitlab-runner
```

### 3.17 CI/CD secrets handling on Linux
Never hardcode secrets in pipeline files. Prefer:
- Jenkins Credentials.
- GitHub Actions Secrets.
- GitLab CI Variables.
- Vault integration.
- Short-lived cloud identities.

### 3.18 Artifact handling in pipelines
Artifacts may include:
- Build outputs.
- Test reports.
- Coverage reports.
- Container images.
- Helm charts.
- SBOMs.

### 3.19 Build cache strategies
- Language package caches.
- Docker layer caching.
- BuildKit cache exports.
- Remote cache services.
- Immutable dependencies.

### 3.20 Linux troubleshooting for build agents
Check:
- Disk space.
- Memory pressure.
- CPU saturation.
- Package mirror access.
- DNS resolution.
- Clock drift.
- Credentials and token expiry.
- Container daemon health.

Useful commands:
```bash
df -h
free -h
journalctl -u docker -n 100 --no-pager
ss -tulpn
curl -v https://registry.example.com/v2/
```

### 3.21 Example deployment gate logic
```bash
#!/usr/bin/env bash
set -euo pipefail

if ! curl -fsS https://staging.example.com/health; then
  echo "Staging health check failed"
  exit 1
fi

echo "Promoting release"
```

### 3.22 Promotion patterns
| Pattern | Description | Notes |
|---|---|---|
| Rebuild per environment | Build separately in each env | Less reproducible |
| Promote same artifact | Build once, promote unchanged | Best for traceability |
| Blue/green | Parallel environments | Easier rollback |
| Canary | Partial traffic shift | Better risk management |

### 3.23 Deployment rollback basics
- Keep prior artifact versions available.
- Use immutable image tags plus digests.
- Maintain DB migration rollback plans.
- Automate rollback triggers where safe.

### 3.24 Pipeline quality gates
- Unit tests.
- Integration tests.
- Linting.
- SAST.
- Dependency scanning.
- Image scanning.
- Policy checks.
- Manual approval for sensitive stages.

### 3.25 Sample Linux CI node bootstrap checklist
- Install language runtimes.
- Install container tooling.
- Configure DNS and CA certificates.
- Configure system limits.
- Set up logging and monitoring.
- Harden SSH and sudo.
- Rotate runner tokens.

---
