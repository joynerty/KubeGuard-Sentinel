# 🛡️ KubeGuard Sentinel

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Go](https://img.shields.io/badge/go-%2300ADD8.svg?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![GitLab](https://img.shields.io/badge/gitlab-%23181717.svg?style=for-the-badge&logo=gitlab&logoColor=white)](https://gitlab.com/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge)](LICENSE)

**Ethical Kubernetes Security Testing Framework** | Automated vulnerability scanning with CVSS scoring and compliance mapping

> Proactively identify security vulnerabilities in Kubernetes clusters before they can be exploited, while maintaining strict ethical boundaries and comprehensive audit trails.

🌐 **[View Live Dashboard Demo →](https://joynerty.github.io/kubeguard-sentinel/)**

---

## 🎯 What is KubeGuard Sentinel?

KubeGuard Sentinel is a production-ready, open-source security testing framework that helps organizations identify Kubernetes vulnerabilities across five critical categories:

### Security Testing Categories

- 🔐 **Privilege Escalation** - Detects cluster-admin bindings, dangerous Linux capabilities (SYS_ADMIN, SYS_MODULE, etc.), RBAC misconfigurations, and service account token extraction vulnerabilities

- 🎭 **Service Account Abuse** - Identifies default SA usage, token automounting issues, cross-namespace access patterns, and legacy token secrets

- 🚪 **Pod Breakout** - Discovers privileged containers, dangerous host path mounts (/, /etc, /var/run), Docker socket exposure, and host namespace usage (hostNetwork, hostPID, hostIPC)

- 🌐 **Network Policy Validation** - Tests for missing network policies, default-deny enforcement gaps, egress restrictions, and pod-to-pod isolation failures

- ☁️ **Cloud Workload Identity** - Validates AWS IRSA, GCP Workload Identity, and Azure Pod Identity configurations, plus IAM permission analysis

### Key Features

✅ **Comprehensive Testing** - 5 major security categories, 20+ test scenarios covering real-world attack vectors  
✅ **CVSS-Based Scoring** - Industry-standard risk quantification (0-10 scale) with automated severity classification  
✅ **Compliance Mapping** - Automatic mapping to CIS Kubernetes Benchmark, PCI-DSS, NIST 800-190, and SOC 2  
✅ **Multiple Deployment Options** - Helm charts, Terraform modules, GitLab CI/CD, standalone Python agent  
✅ **Ethical Framework** - Mandatory authorization, comprehensive audit logging, dry-run defaults  
✅ **Rich Reporting** - JSON, HTML, and SARIF formats with GitHub Security integration  
✅ **Interactive Dashboard** - Real-time visualization with educational tooltips and findings explorer  
✅ **Terraform Sentinel Policies** - Policy-as-Code enforcement for automated compliance validation  

---

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (1.24+)
- kubectl configured with cluster access
- Helm 3.x (for Helm installation) OR Terraform 1.0+ (for Terraform deployment)

### Option 1: Install via Helm (3 commands)

```bash
# 1. Create namespace
kubectl create namespace kubeguard-system

# 2. Install via Helm
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --set ethics.requireAuthorization=true \
  --set ethics.dryRunDefault=true

# 3. Create authorization and run first scan
kubectl apply -f examples/quick-start-scan.yaml
```

### Option 2: Deploy via Terraform

```bash
# Navigate to Terraform directory
cd deployments/terraform

# Initialize and apply
terraform init
terraform apply -var="cluster_name=my-cluster" \
                -var="github_username=joynerty"
```

### Option 3: Run Standalone Python Agent

```bash
# Install dependencies
pip install kubernetes pyyaml

# Run scan
python scripts/agent.py \
  --scan-name quick-test \
  --auth-ref QUICKSTART-001 \
  --profile safe \
  --output results.json
```

**📚 [Complete Setup Guide →](docs/SETUP-GUIDE.md)**

---

## 📊 Example Output

### Terminal Output
```
╔═══════════════════════════════════════════════════╗
║               SCAN SUMMARY                        ║
╚═══════════════════════════════════════════════════╝

📊 Total Findings: 18
   🔴 Critical: 3  (cluster-admin SA, privileged pods, Docker socket mount)
   🟠 High:     5  (dangerous capabilities, missing network policies)
   🟡 Medium:   7  (default SA usage, token automounting)
   🟢 Low:      3  (informational findings)

🎯 Overall Risk Score: 6.8/10.0 (HIGH)

⚠️  CRITICAL FINDINGS DETECTED!
   • Service account production/jenkins-sa has cluster-admin
   • Privileged pod running in production namespace
   • Docker socket mounted in ci-cd/jenkins-worker
   
   Immediate remediation required.

📋 Compliance Scores:
   • CIS Kubernetes: 82%
   • PCI-DSS: 76%
   • NIST 800-190: 79%
   • SOC 2: 85%

✅ Reports saved to: /results/scan-2024-02-16-143052.{json,html,sarif}
```

### Interactive Dashboard

The web dashboard provides:
- Real-time security findings with severity classification
- Interactive risk score visualization with CVSS methodology
- Compliance framework scores and failed controls
- Educational tooltips explaining each finding
- Live scan terminal output
- Remediation recommendations prioritized by severity

**[View Live Demo →](https://joynerty.github.io/kubeguard-sentinel/)**

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    User Interface Layer                       │
│  kubectl | Terraform | GitLab CI | Python CLI | Dashboard   │
└───────────────────────────┬──────────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────────┐
│              KubeGuard Sentinel Controller                    │
│                                                              │
│  ┌─────────────────┐  ┌────────────────┐  ┌──────────────┐ │
│  │  Authorization  │  │ Scanner Engine │  │   Executor   │ │
│  │  • CRD validate │  │ • Discovery    │  │ • Run tests  │ │
│  │  • Audit log    │  │ • Enumerate    │  │ • Collect    │ │
│  │  • Scope check  │  │ • Analyze RBAC │  │ • Evidence   │ │
│  └─────────────────┘  └────────────────┘  └──────────────┘ │
│                                                              │
│  ┌─────────────────┐  ┌────────────────┐  ┌──────────────┐ │
│  │ Reporter Engine │  │ Policy Engine  │  │ Integrations │ │
│  │ • Risk scoring  │  │ • Sentinel     │  │ • Slack      │ │
│  │ • Compliance    │  │ • OPA/Rego     │  │ • SARIF      │ │
│  │ • Reports       │  │ • Validation   │  │ • Webhooks   │ │
│  └─────────────────┘  └────────────────┘  └──────────────┘ │
└──────────────────────────────────────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                    Storage & Output Layer                     │
│  PVC | S3 | GitLab Artifacts | GitHub Security | Grafana    │
└──────────────────────────────────────────────────────────────┘
```

**[Detailed Architecture Documentation →](docs/ARCHITECTURE.md)**

---

## 📦 Deployment Options

### 1. Helm Deployment (Recommended for Kubernetes)

```bash
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --create-namespace \
  --set scanner.defaultProfile=standard \
  --set reporting.slack.enabled=true
```

**Capabilities:**
- CRD-based scan management
- Scheduled automated scans (CronJob)
- Persistent storage (PVC or S3)
- Slack/Email notifications
- Prometheus metrics export
- GitLab/GitHub integration

**[Helm Deployment Guide →](docs/DEPLOYMENT.md#helm)**

### 2. Terraform Deployment (Infrastructure as Code)

```bash
cd deployments/terraform

# Create terraform.tfvars
cat > terraform.tfvars <<EOF
cluster_name             = "production-eks"
github_username          = "joynerty"
enable_irsa              = true
enable_gitlab_integration = true
slack_webhook_url        = "https://hooks.slack.com/services/YOUR/WEBHOOK"
EOF

terraform init
terraform apply
```

**Capabilities:**
- Full infrastructure automation
- AWS IRSA / GCP Workload Identity / Azure Pod Identity
- S3/GCS storage configuration
- GitLab CI/CD integration
- Terraform Sentinel policy enforcement
- Multi-cluster deployment

**[Terraform Deployment Guide →](docs/DEPLOYMENT.md#terraform)**

### 3. GitLab CI/CD Integration

```yaml
# .gitlab-ci.yml
include:
  - project: 'joynerty/kubeguard-sentinel'
    file: '/deployments/gitlab-ci/kubeguard-pipeline.yml'

variables:
  SCAN_PROFILE: "standard"
  CLUSTER_NAME: "production-gke"

k8s-security-scan:
  extends: .kubeguard-scan
  stage: security
  only:
    - main
    - merge_requests
```

**Capabilities:**
- Automated security scanning in CI/CD
- Merge request integration
- Terraform Sentinel policy validation
- Artifact storage
- Security dashboard integration
- Failure thresholds and gates

**[GitLab CI/CD Guide →](docs/GITLAB-CI.md)**

### 4. Python Standalone Agent

```bash
python scripts/agent.py \
  --scan-name ci-pipeline-scan \
  --auth-ref CI-AUTH-001 \
  --profile safe \
  --exclude-ns kube-system,kube-public \
  --output scan-results.json
```

**Capabilities:**
- No cluster installation required
- CI/CD pipeline friendly
- Lightweight execution
- Custom scripting support
- Direct Kubernetes API access

---

## 🔍 Test Categories Explained

### 1. Privilege Escalation Tests
```yaml
What it detects:
  ✓ Service accounts with cluster-admin role bindings
  ✓ Escalate/bind/impersonate RBAC permissions
  ✓ Dangerous Linux capabilities (SYS_ADMIN, SYS_MODULE, SYS_PTRACE, etc.)
  ✓ Service account token extraction vulnerabilities
  ✓ Wildcard RBAC permissions (*, */*, resources/*)
  ✓ Cluster-level role bindings for non-system accounts

Example finding:
  "ServiceAccount production/jenkins-sa has cluster-admin privileges
   via ClusterRoleBinding 'jenkins-admin'. This allows unrestricted
   access to all cluster resources and violates least privilege."

CVSS Score: 9.8 (CRITICAL)
CIS Control: 5.1.5 - Ensure that default service accounts are not actively used
Remediation: Create dedicated role with minimal permissions
```

### 2. Service Account Abuse Tests
```yaml
What it detects:
  ✓ Pods using default service account
  ✓ Service account token automounting enabled unnecessarily
  ✓ Cross-namespace service account access
  ✓ Legacy service account token secrets (not bound tokens)
  ✓ Service accounts with no pods using them
  ✓ Excessive service account permissions

Example finding:
  "15 pods across 3 namespaces use default service account instead
   of dedicated service accounts. This makes RBAC policies difficult
   to enforce and increases blast radius of compromises."

CVSS Score: 5.0 (MEDIUM)
CIS Control: 5.1.6 - Ensure that Service Account Tokens are only mounted where necessary
Remediation: Create app-specific service accounts with automountServiceAccountToken: false
```

### 3. Pod Breakout Tests
```yaml
What it detects:
  ✓ Privileged containers (privileged: true)
  ✓ Host path mounts (/, /etc, /var/run, /proc, /sys)
  ✓ Docker socket mounts (/var/run/docker.sock)
  ✓ Host network usage (hostNetwork: true)
  ✓ Host PID namespace (hostPID: true)
  ✓ Host IPC namespace (hostIPC: true)
  ✓ Containers running as root (runAsUser: 0)

Example finding:
  "Container 'build-agent' in pod ci-cd/jenkins-worker mounts
   /var/run/docker.sock from host. This grants full Docker daemon
   access, enabling arbitrary container creation and cluster takeover."

CVSS Score: 10.0 (CRITICAL)
CIS Control: 5.2.2 - Minimize the admission of privileged containers
Remediation: Use Kubernetes-native builds (Kaniko, BuildKit) instead of Docker-in-Docker
```

### 4. Network Policy Validation
```yaml
What it detects:
  ✓ Namespaces without NetworkPolicy resources
  ✓ Missing default-deny policies
  ✓ Overly permissive ingress rules (0.0.0.0/0)
  ✓ Missing egress restrictions
  ✓ Pod-to-pod isolation gaps
  ✓ DNS policy enforcement issues

Example finding:
  "Production namespace has no NetworkPolicy resources. All pods
   can communicate freely, violating zero-trust principles and
   enabling lateral movement in case of compromise."

CVSS Score: 7.0 (HIGH)
CIS Control: 5.3.2 - Ensure that all Namespaces have Network Policies defined
Remediation: Implement default-deny NetworkPolicy, then allow specific traffic
```

### 5. Cloud Workload Identity Tests
```yaml
What it detects:
  ✓ AWS IRSA role annotations on service accounts
  ✓ GCP Workload Identity bindings
  ✓ Azure Pod Identity configurations
  ✓ Excessive IAM permissions
  ✓ Cross-account access patterns
  ✓ Missing assume role policies

Example finding:
  "Service account 'data-processor' has IRSA role with s3:*
   permissions on all buckets. Excessive permissions increase
   risk if service account token is compromised."

CVSS Score: 6.5 (MEDIUM)
Remediation: Restrict IAM policy to specific buckets and required actions only
```

---

## 📈 Risk Scoring Methodology

### CVSS v3.1 Based Calculation

Each finding receives a CVSS score based on:

```
Attack Vector (AV):
  - Network (N): 0.85
  - Adjacent (A): 0.62
  - Local (L): 0.55
  - Physical (P): 0.20

Attack Complexity (AC):
  - Low (L): 0.77
  - High (H): 0.44

Privileges Required (PR):
  - None (N): 0.85
  - Low (L): 0.62
  - High (H): 0.27

User Interaction (UI):
  - None (N): 0.85
  - Required (R): 0.62

Scope (S):
  - Changed (C): Enables privilege escalation
  - Unchanged (U): Limited to compromised component

Impact (CIA):
  - High (H): 0.56
  - Low (L): 0.22
  - None (N): 0.00

Base Score = 
  IF (Impact ≤ 0) THEN 0
  ELSE
    Exploitability × Impact × Scope

Overall Risk Score =
  (Weighted Sum of Findings) / (Maximum Possible) × 10
  
  Weights:
    Critical (9.0-10.0): 10.0
    High     (7.0-8.9):  7.0
    Medium   (4.0-6.9):  4.0
    Low      (0.1-3.9):  1.0
```

### Risk Levels and Actions

| Score | Level | SLA | Action Required |
|-------|-------|-----|-----------------|
| 9.0-10.0 | 🔴 CRITICAL | 24 hours | Immediate remediation, isolate affected workloads |
| 7.0-8.9 | 🟠 HIGH | 7 days | Plan remediation this week, review with security team |
| 4.0-6.9 | 🟡 MEDIUM | 30 days | Address in next sprint, track in backlog |
| 0.1-3.9 | 🟢 LOW | 90 days | Include in routine hardening activities |
| 0.0 | ✅ PASS | - | No vulnerabilities detected |

---

## 🎯 Compliance Mapping

Findings are automatically mapped to compliance frameworks:

### CIS Kubernetes Benchmark
```
Supported Controls:
├─ 5.1.x - RBAC and Service Accounts
│  ├─ 5.1.1 - Ensure that service account tokens are only mounted where necessary
│  ├─ 5.1.5 - Ensure that default service accounts are not actively used
│  └─ 5.1.6 - Ensure that Service Account Tokens are only mounted where necessary
├─ 5.2.x - Pod Security Policies
│  ├─ 5.2.1 - Minimize the admission of privileged containers
│  ├─ 5.2.2 - Minimize the admission of containers with privilege escalation
│  ├─ 5.2.6 - Minimize the admission of root containers
│  └─ 5.2.8 - Minimize the admission of containers with capabilities assigned
└─ 5.3.x - Network Policies
   └─ 5.3.2 - Ensure that all Namespaces have Network Policies defined

Report Format:
{
  "cisKubernetes": {
    "score": 82.0,
    "passed": 16,
    "failed": 4,
    "failedControls": [
      "5.1.5 - Default service account usage detected",
      "5.2.1 - Privileged containers found",
      "5.3.2 - Missing network policies"
    ]
  }
}
```

### PCI-DSS Requirements
- Requirement 2: Do not use vendor-supplied defaults
- Requirement 6: Develop and maintain secure systems
- Requirement 10: Track and monitor all access to network resources

### NIST 800-190 Container Security
- 4.1 Image and Registry Security
- 4.2 Orchestrator Security
- 4.3 Container Runtime Security
- 4.4 Host Operating System Security

### SOC 2 Type II Controls
- CC6.1 - Logical and physical access controls
- CC6.6 - Vulnerability management
- CC7.1 - Change management

---

## 🔧 Configuration Examples

### Basic Scan Configuration
```yaml
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: basic-scan
  namespace: kubeguard-system
spec:
  authorizationRef: security-team-auth
  profile: safe  # safe | standard | aggressive
  tests:
    - privilegeEscalation
    - serviceAccountAbuse
    - networkPolicyValidation
  scope:
    namespaces:
      - production
      - staging
  reporting:
    format:
      - json
      - html
    severity: medium  # Show medium and above
```

### Advanced Production Scan
```yaml
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: production-compliance-scan
  namespace: kubeguard-system
spec:
  authorizationRef: compliance-audit-auth
  profile: standard
  tests:
    - all  # Run all test categories
  scope:
    namespaces:
      - production
      - prod-*
    excludeNamespaces:
      - kube-system
      - kube-public
      - kube-node-lease
  reporting:
    format:
      - json
      - html
      - sarif  # For GitHub Security
    severity: low  # Show all findings
    slack:
      enabled: true
      channel: "#security-alerts"
      webhookSecretName: slack-webhook
    email:
      enabled: true
      recipients:
        - security-team@company.com
        - oncall@company.com
  schedule:
    cron: "0 2 * * 0"  # Weekly on Sunday at 2 AM
  compliance:
    frameworks:
      - cisKubernetes
      - pciDss
      - nist800190
      - soc2
    minimumScore: 80.0  # Fail if below 80%
```

**[More Configuration Examples →](docs/EXAMPLES.md)**

---

## 🔐 Ethical Safeguards

KubeGuard Sentinel is designed with **ethics first**:

### Mandatory Authorization
Every scan requires a `ScanAuthorization` CRD:

```yaml
apiVersion: kubeguard.io/v1alpha1
kind: ScanAuthorization
metadata:
  name: production-scan-auth
  namespace: kubeguard-system
spec:
  # Who authorized this scan
  authorizedBy: "tyler.joyner@company.com"
  
  # Approval ticket for audit trail
  approvalTicket: "JIRA-SEC-1234"
  
  # Time-based expiry
  validUntil: "2024-12-31T23:59:59Z"
  
  # Scope limitations
  scope:
    namespaces:
      - production
    excludeNamespaces:
      - kube-system
    maxRiskProfile: standard  # Can't run aggressive without additional approval
  
  # Emergency contacts
  notificationContacts:
    - oncall@company.com
```

### Built-in Protections

- ✅ **Authorization Required** - All scans must reference a valid ScanAuthorization
- ✅ **Comprehensive Audit Logging** - Every action logged with timestamp, user, and scope
- ✅ **Dry-Run by Default** - Tests simulate without making actual changes
- ✅ **Namespace Exclusions** - Critical namespaces (kube-system) auto-excluded
- ✅ **Time-Based Expiry** - Authorizations expire automatically
- ✅ **Approval Workflows** - Aggressive tests require additional approval
- ✅ **Rate Limiting** - Prevents overwhelming API server
- ✅ **Resource Quotas** - Scanner pods have CPU/memory limits

### Audit Log Format
```json
{
  "timestamp": "2024-02-16T14:30:52Z",
  "action": "scan_initiated",
  "user": "tyler.joyner@company.com",
  "scanName": "production-weekly-scan",
  "authorizationRef": "production-scan-auth",
  "profile": "standard",
  "scope": {
    "namespaces": ["production", "staging"],
    "excludedNamespaces": ["kube-system"]
  },
  "findings": {
    "total": 18,
    "critical": 3,
    "high": 5,
    "medium": 7,
    "low": 3
  }
}
```

---

## 🔗 Integrations

### CI/CD Pipelines

**GitLab CI/CD:**
```yaml
# .gitlab-ci.yml
include:
  - remote: 'https://raw.githubusercontent.com/joynerty/kubeguard-sentinel/main/deployments/gitlab-ci/kubeguard-template.yml'

k8s-security-scan:
  stage: security
  extends: .kubeguard-scan
  variables:
    SCAN_PROFILE: "standard"
    FAIL_ON_CRITICAL: "true"
  artifacts:
    reports:
      security: scan-results.sarif
```

**GitHub Actions:**
```yaml
# .github/workflows/security-scan.yml
- name: Run KubeGuard Security Scan
  uses: joynerty/kubeguard-sentinel-action@v1
  with:
    scan-profile: 'standard'
    fail-on-critical: true
    upload-sarif: true
```

### Monitoring & Alerting

**Prometheus Metrics:**
```yaml
# Exported metrics
- kubeguard_scans_total{profile,status}
- kubeguard_findings_total{severity,category}
- kubeguard_scan_duration_seconds{profile}
- kubeguard_compliance_score{framework}
```

**Grafana Dashboard:**
```bash
kubectl apply -f monitoring/grafana-dashboard.yaml
# Dashboard shows: scan history, finding trends, compliance scores
```

### Notifications

- **Slack** - Real-time finding alerts with severity emojis
- **Email** - HTML scan completion reports
- **Webhooks** - Custom integrations (PagerDuty, Jira, etc.)
- **SARIF** - GitHub/GitLab Security tab integration

---

## 🛠️ Development

### Building from Source

```bash
# Clone repository
git clone https://github.com/joynerty/kubeguard-sentinel.git
cd kubeguard-sentinel

# Build Go binary
make build

# Run tests
make test

# Build Docker image
make docker-build

# Deploy locally
make kubectl-deploy
```

### Project Structure
```
kubeguard-sentinel/
├── cmd/
│   └── kubeguard/              # Main application entry point
├── pkg/
│   ├── scanner/               # Resource discovery engine
│   ├── executor/              # Security test execution
│   ├── reporter/              # Risk scoring & reporting
│   └── policy/                # Sentinel policy evaluation
├── scripts/
│   └── agent.py               # Standalone Python agent
├── deployments/
│   ├── helm/                  # Helm charts
│   ├── terraform/             # Terraform modules
│   │   ├── main.tf
│   │   └── modules/
│   │       └── sentinel/      # Sentinel policies
│   └── gitlab-ci/             # GitLab CI templates
├── docs/                      # Documentation
├── examples/                  # Example configurations
└── tests/                     # Test suites
```

### Running Tests

```bash
# Unit tests
go test -v ./...

# Integration tests (requires cluster)
make test-integration

# Python agent tests
pytest scripts/tests/

# Terraform validation
cd deployments/terraform && terraform validate
```

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-improvement`)
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass (`make test`)
6. Commit with descriptive messages (`git commit -m 'Add XYZ feature'`)
7. Push to your fork (`git push origin feature/amazing-improvement`)
8. Open a Pull Request

**[Contributing Guidelines →](CONTRIBUTING.md)**

### Development Guidelines

- Follow Go best practices and conventions
- Maintain test coverage above 80%
- Update documentation for new features
- Add examples for new scan types
- Ensure Terraform modules pass validation
- Test on multiple Kubernetes versions (1.24+)

---

## 📊 Project Statistics

- **Languages:** Go (60%), Python (25%), HCL (10%), HTML/CSS/JS (5%)
- **Lines of Code:** ~3,500 (excluding vendored dependencies)
- **Test Coverage:** 85%+ across core packages
- **Documentation:** 8 comprehensive guides + API reference
- **Deployment Options:** 4 (Helm, Terraform, GitLab CI, Standalone)
- **Platform Support:** AWS EKS, GCP GKE, Azure AKS, on-premises Kubernetes
- **Compliance Frameworks:** 4 (CIS Kubernetes, PCI-DSS, NIST 800-190, SOC 2)

---

## 📜 License

Apache License 2.0 - See [LICENSE](LICENSE) file for complete terms

Copyright 2024 Tyler Joyner

---

## 🙏 Acknowledgments

Built with inspiration and guidance from:
- **CNCF Security Technical Advisory Group (TAG)**
- **Kubernetes SIG Security**
- **OWASP Kubernetes Security Cheat Sheet**
- **NSA/CISA Kubernetes Hardening Guidance**
- **CIS Kubernetes Benchmark v1.8**

Special thanks to the open-source security community for tools, research, and best practices.

---

## 📧 Contact & Support

**Author:** Tyler Joyner  
**Email:** [joynertyler75@gmail.com](mailto:joynertyler75@gmail.com)  
**GitHub:** [@joynerty](https://github.com/joynerty)  
**Project:** [github.com/joynerty/kubeguard-sentinel](https://github.com/joynerty/kubeguard-sentinel)

**For Issues & Feature Requests:**  
[GitHub Issues](https://github.com/joynerty/kubeguard-sentinel/issues)

**For Questions & Discussions:**  
[GitHub Discussions](https://github.com/joynerty/kubeguard-sentinel/discussions)

**For Security Vulnerabilities in the Tool:**  
Email: [joynertyler75@gmail.com](mailto:joynertyler75@gmail.com) with subject "SECURITY"

---

## ⭐ Star History

If you find this project useful, please consider giving it a star!

[![Star History Chart](https://api.star-history.com/svg?repos=joynerty/kubeguard-sentinel&type=Date)](https://star-history.com/#joynerty/kubeguard-sentinel&Date)

---

<div align="center">

**Built with ❤️ for the Kubernetes security community**

[Documentation](docs/) • [Examples](docs/EXAMPLES.md) • [Architecture](docs/ARCHITECTURE.md) • [Contributing](CONTRIBUTING.md)

**Made by [@joynerty](https://github.com/joynerty) | Licensed under Apache 2.0**

</div>
