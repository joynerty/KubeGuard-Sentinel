# 🛡️ KubeGuard Sentinel

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Go](https://img.shields.io/badge/go-%2300ADD8.svg?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)](https://www.python.org/)
[![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge)](LICENSE)

**Ethical Kubernetes Security Testing Framework** | Automated vulnerability scanning with CVSS scoring and compliance mapping

> Proactively identify security vulnerabilities in Kubernetes clusters before they can be exploited, while maintaining strict ethical boundaries and audit trails.

![Dashboard Preview](assets/dashboard-preview.png)

---

## 🎯 What is KubeGuard Sentinel?

KubeGuard Sentinel is a production-ready security testing framework that helps organizations identify Kubernetes vulnerabilities across:

- 🔐 **Privilege Escalation** - Cluster-admin bindings, dangerous capabilities, RBAC misconfigurations
- 🎭 **Service Account Abuse** - Default SA usage, token extraction, cross-namespace access
- 🚪 **Pod Breakout** - Host mounts, privileged containers, Docker socket exposure
- 🌐 **Network Policies** - Policy gaps, egress restrictions, zero-trust validation
- ☁️ **Cloud Workload Identity** - AWS IRSA, GCP Workload Identity, Azure Pod Identity

### Key Features

✅ **Comprehensive Testing** - 5 major security categories, 20+ test scenarios  
✅ **CVSS-Based Scoring** - Industry-standard risk quantification (0-10 scale)  
✅ **Compliance Mapping** - CIS Kubernetes, PCI-DSS, NIST 800-190, SOC 2  
✅ **Multiple Deployment Options** - Helm, Terraform, standalone Python agent  
✅ **Ethical Framework** - Mandatory authorization, audit logging, dry-run defaults  
✅ **Rich Reporting** - JSON, HTML, SARIF (GitHub Security integration)  
✅ **Interactive Dashboard** - Real-time visualization and findings explorer  

---

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (1.24+)
- kubectl configured
- Helm 3.x (for Helm installation)

### Install in 3 Commands

```bash
# 1. Create namespace
kubectl create namespace kubeguard-system

# 2. Install via Helm
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --set ethics.requireAuthorization=true

# 3. Create authorization and run first scan
kubectl apply -f examples/quick-start-scan.yaml
```

**📚 [Full Setup Guide →](SETUP-GUIDE.md)**

---

## 📊 Example Output

### Risk Score Dashboard

```
╔═══════════════════════════════════════════════════╗
║               SCAN SUMMARY                        ║
╚═══════════════════════════════════════════════════╝

📊 Total Findings: 18
   🔴 Critical: 3
   🟠 High:     5
   🟡 Medium:   7
   🟢 Low:      3

🎯 Overall Risk Score: 6.8/10.0 (HIGH)

⚠️  CRITICAL FINDINGS DETECTED!
   Immediate remediation required.
```

### Interactive Web Dashboard

![Findings Dashboard](assets/findings-view.png)

**[View Live Demo →](https://your-username.github.io/kubeguard-sentinel/)**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  User Interface Layer                    │
│  kubectl | Terraform | Python CLI | Web Dashboard       │
└─────────────────────────┬───────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────┐
│              KubeGuard Sentinel Controller              │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │ Authorization    │  │ Scanner Engine   │           │
│  │ • Validate scope │  │ • Discovery      │           │
│  │ • Audit logging  │  │ • Enumeration    │           │
│  └──────────────────┘  └──────────────────┘           │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │ Executor Engine  │  │ Reporter Engine  │           │
│  │ • Run tests      │  │ • Risk scoring   │           │
│  │ • Collect data   │  │ • Compliance map │           │
│  └──────────────────┘  └──────────────────┘           │
└─────────────────────────────────────────────────────────┘
```

**[Detailed Architecture →](docs/ARCHITECTURE.md)**

---

## 📦 Deployment Options

### Option 1: Helm (Recommended)

```bash
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --create-namespace
```

**Supports:**
- Scheduled scans (CronJob)
- Persistent storage (PVC/S3)
- Slack/Email notifications
- Prometheus metrics

### Option 2: Terraform

```bash
cd deployments/terraform
terraform init
terraform apply -var="cluster_name=production-eks"
```

**Supports:**
- AWS IRSA integration
- GCP Workload Identity
- Automated infrastructure
- Multi-cluster setup

### Option 3: Python Agent (Standalone)

```bash
python scripts/agent.py \
  --scan-name quick-test \
  --auth-ref AUTH-001 \
  --profile safe \
  --output results.json
```

**Supports:**
- CI/CD integration
- No cluster installation
- Lightweight execution
- Custom scripting

---

## 🎓 Documentation

| Document | Description |
|----------|-------------|
| [Quick Start Guide](QUICKSTART.md) | Get up and running in 10 minutes |
| [Setup Guide](SETUP-GUIDE.md) | Detailed configuration instructions |
| [Architecture](docs/ARCHITECTURE.md) | System design and components |
| [Deployment Guide](docs/DEPLOYMENT.md) | Production deployment patterns |
| [Examples](docs/EXAMPLES.md) | Sample scan configurations |
| [API Reference](docs/API.md) | CRD specifications and API |

---

## 🔍 Test Categories

### 1. Privilege Escalation
```yaml
Tests:
  ✓ Cluster-admin role bindings
  ✓ Escalate/bind/impersonate permissions
  ✓ Dangerous capabilities (SYS_ADMIN, SYS_MODULE, etc.)
  ✓ Service account token extraction
```

### 2. Service Account Abuse
```yaml
Tests:
  ✓ Default service account usage
  ✓ Token automounting configuration
  ✓ Cross-namespace access patterns
  ✓ Legacy token secrets
```

### 3. Pod Breakout
```yaml
Tests:
  ✓ Privileged container detection
  ✓ Host path mounts (/, /etc, /var/run)
  ✓ Docker socket exposure
  ✓ Host namespace usage (hostNetwork, hostPID, hostIPC)
```

### 4. Network Policy Validation
```yaml
Tests:
  ✓ Network policy existence
  ✓ Default-deny enforcement
  ✓ Egress restrictions
  ✓ Pod-to-pod isolation
```

### 5. Cloud Workload Identity
```yaml
Tests:
  ✓ AWS IRSA configuration
  ✓ GCP Workload Identity bindings
  ✓ Azure Pod Identity setup
  ✓ IAM permission analysis
```

---

## 📈 Risk Scoring

### CVSS-Based Methodology

Each finding receives a CVSS v3.1 score based on:
- Attack Vector
- Attack Complexity
- Privileges Required
- User Interaction
- Impact (CIA)

### Risk Levels

| Score | Level | Action |
|-------|-------|--------|
| 9.0-10.0 | 🔴 CRITICAL | Immediate remediation |
| 7.0-8.9 | 🟠 HIGH | Fix this week |
| 4.0-6.9 | 🟡 MEDIUM | Next sprint |
| 0.1-3.9 | 🟢 LOW | Routine hardening |

---

## 🎯 Compliance Mapping

Findings are automatically mapped to:

- **CIS Kubernetes Benchmark** - Industry baseline
- **PCI-DSS** - Payment card security
- **NIST 800-190** - Container security
- **SOC 2** - Service organization controls

Example compliance report:
```json
{
  "complianceScore": {
    "cisKubernetes": 82.0,
    "pciDss": 76.0,
    "nist": 79.0,
    "soc2": 85.0
  },
  "failedControls": {
    "CIS Kubernetes": [
      "5.1.5 - Minimize cluster-admin bindings",
      "5.2.2 - Minimize privileged containers"
    ]
  }
}
```

---

## 🔧 Configuration

### Basic Scan

```yaml
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: my-scan
spec:
  authorizationRef: my-auth
  profile: safe
  tests:
    - privilegeEscalation
    - networkPolicyValidation
  reporting:
    format: ["json", "html"]
```

### Advanced Configuration

```yaml
spec:
  profile: standard
  tests:
    - all
  scope:
    namespaces: ["production", "staging"]
    excludeNamespaces: ["kube-system"]
  reporting:
    format: ["json", "html", "sarif"]
    severity: medium
    slack:
      enabled: true
      channel: "#security"
```

**[More Examples →](docs/EXAMPLES.md)**

---

## 🔐 Ethical Safeguards

KubeGuard Sentinel is designed with ethics first:

### Mandatory Authorization
```yaml
apiVersion: kubeguard.io/v1alpha1
kind: ScanAuthorization
metadata:
  name: production-auth
spec:
  authorizedBy: "security-team@company.com"
  approvalTicket: "JIRA-SEC-1234"
  validUntil: "2024-12-31T23:59:59Z"
  scope:
    namespaces: ["production"]
    maxRiskProfile: standard
```

### Built-in Protections
- ✅ Authorization required for all scans
- ✅ Comprehensive audit logging
- ✅ Dry-run mode by default
- ✅ Namespace exclusion lists
- ✅ Time-based authorization expiry
- ✅ Approval workflows for aggressive testing

---

## 🔗 Integrations

### CI/CD

**GitHub Actions:**
```yaml
- name: Security Scan
  run: |
    kubectl apply -f scan.yaml
    kubectl wait --for=condition=Complete securityscan/ci-scan
```

**GitLab CI:**
```yaml
security-scan:
  script:
    - kubectl apply -f scan.yaml
    - python scripts/wait-for-scan.py
```

### Monitoring

**Prometheus Metrics:**
```yaml
# Exposed metrics
- kubeguard_scans_total
- kubeguard_findings_by_severity
- kubeguard_scan_duration_seconds
```

**Grafana Dashboard:**
```bash
kubectl apply -f monitoring/grafana-dashboard.yaml
```

### Notifications

- **Slack** - Real-time finding alerts
- **Email** - Scan completion reports
- **Webhooks** - Custom integrations
- **SARIF** - GitHub Security integration

---

## 🛠️ Development

### Build from Source

```bash
# Build Go binary
make build

# Run tests
make test

# Build Docker image
make docker-build

# Install locally
make kubectl-deploy
```

### Project Structure

```
kubeguard-sentinel/
├── cmd/kubeguard/          # Application entry point
├── pkg/
│   ├── scanner/           # Discovery engine
│   ├── executor/          # Test execution
│   └── reporter/          # Risk scoring
├── scripts/               # Python agent
├── deployments/
│   ├── helm/             # Helm charts
│   └── terraform/        # Terraform modules
└── docs/                 # Documentation
```

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

**[Contributing Guidelines →](CONTRIBUTING.md)**

---

## 📊 Project Stats

- **Languages:** Go (60%), Python (25%), HCL (10%), HTML/CSS (5%)
- **Lines of Code:** ~3,500
- **Test Coverage:** Comprehensive test scenarios
- **Documentation:** 6 detailed guides
- **Deployment Options:** 3 (Helm, Terraform, Standalone)
- **Platform Support:** AWS, GCP, Azure, on-premises

---

## 📜 License

Apache License 2.0 - see [LICENSE](LICENSE) file for details

---

## 🙏 Acknowledgments

Built with inspiration from:
- CNCF Security TAG
- Kubernetes SIG Security
- OWASP Kubernetes Security Cheat Sheet
- NSA/CISA Kubernetes Hardening Guide
- CIS Kubernetes Benchmark

---

## 📧 Contact

- **Issues:** [GitHub Issues](https://github.com/YOUR-USERNAME/kubeguard-sentinel/issues)
- **Discussions:** [GitHub Discussions](https://github.com/YOUR-USERNAME/kubeguard-sentinel/discussions)
- **Security:** security@example.com (for vulnerabilities in the tool itself)

---

## ⭐ Star History

If you find this project useful, please consider giving it a star!

[![Star History Chart](https://api.star-history.com/svg?repos=YOUR-USERNAME/kubeguard-sentinel&type=Date)](https://star-history.com/#YOUR-USERNAME/kubeguard-sentinel&Date)

---

<div align="center">

**Built with ❤️ for the Kubernetes security community**

[Documentation](docs/) • [Examples](docs/EXAMPLES.md) • [Architecture](docs/ARCHITECTURE.md) • [Contributing](CONTRIBUTING.md)

</div>
