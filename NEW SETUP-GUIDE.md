# 🚀 KubeGuard Sentinel - Setup Guide

**Author:** Tyler Joyner  
**Email:** joynertyler75@gmail.com  
**GitHub:** [@joynerty](https://github.com/joynerty)  
**Project:** https://github.com/joynerty/kubeguard-sentinel

---

## 📋 Prerequisites

### Required Tools

- **Kubernetes cluster** (1.24+)
  - Minikube (local testing)
  - AWS EKS / GCP GKE / Azure AKS (production)
  - Kind or k3s (development)

- **kubectl** configured
  ```bash
  kubectl version --short
  # Should show client and server versions
  ```

- **Helm 3.x** (for Helm installation)
  ```bash
  helm version
  # Version: v3.12+
  ```

- **Terraform 1.0+** (for Terraform deployment)
  ```bash
  terraform version
  # Terraform v1.6+
  ```

- **Python 3.9+** (for standalone agent)
  ```bash
  python --version
  # Python 3.9+
  ```

### Required Permissions

- Cluster-wide **read access** for scanning
- Namespace **admin access** for kubeguard-system
- Ability to **create CRDs**
- (Optional) **Pod exec** permissions for advanced tests

---

## 🎯 Quick Start (5 Minutes)

### Option 1: Helm Installation

```bash
# 1. Clone repository
git clone https://github.com/joynerty/kubeguard-sentinel.git
cd kubeguard-sentinel

# 2. Create namespace
kubectl create namespace kubeguard-system

# 3. Install via Helm
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --set ethics.requireAuthorization=true \
  --set ethics.dryRunDefault=true

# 4. Verify installation
kubectl get pods -n kubeguard-system
```

### Option 2: Quick Python Agent

```bash
# 1. Install dependencies
pip install kubernetes pyyaml

# 2. Run a scan
python scripts/agent.py \
  --scan-name quick-test \
  --auth-ref QUICKSTART-001 \
  --profile safe \
  --output results.json

# 3. View results
cat results.json | python -m json.tool
```

---

## 📦 Detailed Deployment Options

### 🎭 Option 1: Helm Deployment (Recommended)

#### Basic Installation

```bash
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --create-namespace
```

#### Custom Configuration

Create `custom-values.yaml`:

```yaml
# Ethics and Safety
ethics:
  requireAuthorization: true
  auditLogging: true
  dryRunDefault: true
  excludeNamespaces:
    - kube-system
    - kube-public

# Scanner Configuration
scanner:
  defaultProfile: "standard"  # safe | standard | aggressive
  schedule: "0 2 * * 0"       # Weekly on Sunday at 2 AM
  maxConcurrentScans: 3

# Storage
storage:
  type: persistentVolumeClaim
  persistentVolumeClaim:
    size: 10Gi
    storageClass: standard

# Notifications
reporting:
  slack:
    enabled: true
    channel: "#security-alerts"
    webhookSecretName: slack-webhook
  email:
    enabled: true
    recipients:
      - security@company.com

# Resources
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

Install with custom values:

```bash
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --values custom-values.yaml
```

#### Verify Installation

```bash
# Check pods
kubectl get pods -n kubeguard-system

# Check CRDs
kubectl get crd | grep kubeguard

# View logs
kubectl logs -n kubeguard-system -l app=kubeguard-sentinel
```

---

### 🔧 Option 2: Terraform Deployment

#### AWS EKS with IRSA

```bash
cd deployments/terraform

# Create terraform.tfvars
cat > terraform.tfvars <<EOF
cluster_name             = "production-eks"
namespace                = "kubeguard-system"
enable_irsa              = true
default_scan_profile     = "standard"
storage_size             = "20Gi"

# Optional: Notifications
slack_webhook_url = "https://hooks.slack.com/services/YOUR/WEBHOOK"
email_recipients  = ["security@company.com"]
EOF

# Initialize and deploy
terraform init
terraform plan
terraform apply
```

#### GCP GKE with Workload Identity

```bash
cat > terraform.tfvars <<EOF
cluster_name                = "production-gke"
enable_workload_identity    = true
project_id                  = "my-gcp-project"
default_scan_profile        = "standard"
EOF

terraform init
terraform apply
```

#### Azure AKS with Pod Identity

```bash
cat > terraform.tfvars <<EOF
cluster_name              = "production-aks"
enable_pod_identity       = true
resource_group            = "my-resource-group"
default_scan_profile      = "standard"
EOF

terraform init
terraform apply
```

---

### 🦊 Option 3: GitLab CI/CD Integration

#### Step 1: Add Template to Repository

In your `.gitlab-ci.yml`:

```yaml
include:
  - project: 'joynerty/kubeguard-sentinel'
    file: '/deployments/gitlab-ci/kubeguard-template.yml'

variables:
  SCAN_PROFILE: "standard"
  FAIL_ON_CRITICAL: "true"
  KUBE_CONTEXT: ${KUBE_CONTEXT_PRODUCTION}

stages:
  - validate
  - deploy
  - scan
  - report
```

#### Step 2: Configure GitLab Variables

In GitLab Settings → CI/CD → Variables:

```
KUBE_CONTEXT_PRODUCTION = your-kubernetes-context
KUBE_CONTEXT_STAGING = your-staging-context
GITLAB_TOKEN = your-gitlab-api-token
SLACK_WEBHOOK_URL = https://hooks.slack.com/services/...
```

#### Step 3: Run Pipeline

```bash
git add .gitlab-ci.yml
git commit -m "Add KubeGuard security scanning"
git push origin main
```

The pipeline will automatically:
1. Validate Terraform and Sentinel policies
2. Plan infrastructure changes
3. Deploy KubeGuard (if approved)
4. Run security scans
5. Generate reports
6. Post to merge requests

---

### 🐍 Option 4: Python Standalone Agent

#### Installation

```bash
# Install dependencies
pip install kubernetes pyyaml

# Or use requirements file
pip install -r requirements.txt
```

#### Configuration

```bash
# Ensure kubectl is configured
export KUBECONFIG=~/.kube/config
kubectl cluster-info
```

#### Running Scans

**Safe scan (read-only):**
```bash
python scripts/agent.py \
  --scan-name dev-safe-scan \
  --auth-ref DEV-001 \
  --profile safe \
  --output results.json
```

**Standard scan:**
```bash
python scripts/agent.py \
  --scan-name prod-scan \
  --auth-ref PROD-001 \
  --profile standard \
  --exclude-ns kube-system,kube-public \
  --output scan-results.json
```

**View results:**
```bash
# Pretty print JSON
cat scan-results.json | python -m json.tool

# Or use jq
cat scan-results.json | jq '.'
```

---

## 🔐 Creating Scan Authorization

**CRITICAL:** All scans require a ScanAuthorization CRD.

```bash
kubectl apply -f - <<EOF
apiVersion: kubeguard.io/v1alpha1
kind: ScanAuthorization
metadata:
  name: production-auth
  namespace: kubeguard-system
spec:
  # Who authorized this scan
  authorizedBy: "tyler.joyner@company.com"
  
  # Reference ticket for audit trail
  approvalTicket: "JIRA-SEC-1234"
  
  # Expiration (30 days from now)
  validUntil: "2024-12-31T23:59:59Z"
  
  # Scope limitations
  scope:
    namespaces:
      - production
      - staging
    excludeNamespaces:
      - kube-system
      - kube-public
    maxRiskProfile: standard
  
  # Notification contacts
  notificationContacts:
    - oncall@company.com
EOF
```

Verify authorization:
```bash
kubectl get scanauthorization -n kubeguard-system
```

---

## 🎯 Running Your First Scan

### Safe Scan (Recommended)

```bash
kubectl apply -f - <<EOF
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: first-scan
  namespace: kubeguard-system
spec:
  authorizationRef: production-auth
  profile: safe
  tests:
    - privilegeEscalation
    - serviceAccountAbuse
    - networkPolicyValidation
  scope:
    namespaces:
      - default
  reporting:
    format:
      - json
      - html
    severity: low
EOF
```

### Monitor Progress

```bash
# Watch scan status
kubectl get securityscan first-scan -n kubeguard-system -w

# View detailed status
kubectl describe securityscan first-scan -n kubeguard-system

# Check logs
kubectl logs -n kubeguard-system -l app=kubeguard-sentinel -f
```

### Download Results

```bash
# Get pod name
POD=$(kubectl get pod -n kubeguard-system -l app=kubeguard-sentinel -o jsonpath='{.items[0].metadata.name}')

# Download reports
kubectl cp kubeguard-system/$POD:/results/scan-results.json ./results.json
kubectl cp kubeguard-system/$POD:/results/scan-results.html ./results.html

# Open HTML report
open results.html  # macOS
xdg-open results.html  # Linux
```

---

## 📊 Viewing the Dashboard

### Option 1: GitHub Pages

```bash
# Copy dashboard to index.html
cp dashboard.html index.html

# Push to GitHub
git add index.html
git commit -m "Add security dashboard"
git push origin main
```

Enable GitHub Pages:
1. Go to repository Settings → Pages
2. Source: **main** branch
3. Save

Dashboard will be live at: `https://joynerty.github.io/kubeguard-sentinel/`

### Option 2: Local Viewing

```bash
# Simply open the file
open dashboard.html
```

---

## ⚙️ Configuration

### Scan Profiles

**Safe Profile** (default):
- Read-only discovery
- RBAC enumeration
- Network policy checks
- No invasive tests
- **Recommended for:** First scans, development

**Standard Profile**:
- All safe tests
- Pod breakout simulation
- Service account analysis
- Cloud identity checks
- **Recommended for:** Production scans

**Aggressive Profile**:
- All standard tests
- Advanced exploitation scenarios
- Requires additional approval
- **Recommended for:** Security team deep-dives

### Notification Setup

#### Slack

```bash
# Create secret
kubectl create secret generic slack-webhook \
  --namespace kubeguard-system \
  --from-literal=webhook-url='https://hooks.slack.com/services/YOUR/WEBHOOK'

# Update Helm
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values \
  --set reporting.slack.enabled=true \
  --set reporting.slack.webhookSecretName=slack-webhook
```

#### Email

```bash
# Create SMTP secret
kubectl create secret generic smtp-creds \
  --namespace kubeguard-system \
  --from-literal=host='smtp.gmail.com' \
  --from-literal=port='587' \
  --from-literal=username='alerts@company.com' \
  --from-literal=password='your-app-password'

# Update Helm
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values \
  --set reporting.email.enabled=true \
  --set reporting.email.smtpSecretName=smtp-creds \
  --set 'reporting.email.recipients={security@company.com}'
```

---

## 🔍 Understanding Results

### Risk Score Interpretation

| Score | Level | Action |
|-------|-------|--------|
| 9.0-10.0 | 🔴 CRITICAL | Fix immediately (24h SLA) |
| 7.0-8.9 | 🟠 HIGH | Fix this week (7d SLA) |
| 4.0-6.9 | 🟡 MEDIUM | Next sprint (30d SLA) |
| 0.1-3.9 | 🟢 LOW | Routine hardening (90d SLA) |

### Common Findings

**Critical: Service Account with cluster-admin**
```yaml
Finding: ServiceAccount has cluster-admin role binding
Impact: Complete cluster compromise possible
Fix:
  1. kubectl delete clusterrolebinding <binding-name>
  2. Create specific role with minimal permissions
  3. kubectl create role app-role --verb=get,list --resource=pods
```

**High: Privileged Pod**
```yaml
Finding: Pod running with privileged: true
Impact: Container escape, host compromise
Fix:
  spec:
    securityContext:
      privileged: false  # Remove this
      capabilities:
        add: ["NET_ADMIN"]  # Add only needed capabilities
```

**Medium: Default Service Account**
```yaml
Finding: Pods using default service account
Impact: Overly permissive RBAC
Fix:
  1. kubectl create serviceaccount my-app-sa
  2. spec:
       serviceAccountName: my-app-sa
       automountServiceAccountToken: false
```

---

## 🐛 Troubleshooting

### Issue: Scan stuck in Pending

```bash
# Check authorization
kubectl get scanauthorization -n kubeguard-system

# View logs
kubectl logs -n kubeguard-system -l app=kubeguard-sentinel

# Describe scan
kubectl describe securityscan <scan-name> -n kubeguard-system
```

### Issue: Permission denied

```bash
# Check service account permissions
kubectl auth can-i --list \
  --as=system:serviceaccount:kubeguard-system:kubeguard-sentinel

# Verify RBAC
kubectl get clusterrolebinding | grep kubeguard
```

### Issue: No results available

```bash
# Check PVC
kubectl get pvc -n kubeguard-system

# Verify mount
kubectl describe pod -n kubeguard-system -l app=kubeguard-sentinel | grep -A5 Mounts
```

---

## 🔄 Upgrading

### Helm Upgrade

```bash
# Update repository
git pull origin main

# Upgrade release
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values
```

### Terraform Upgrade

```bash
# Update code
git pull origin main

# Apply changes
terraform plan
terraform apply
```

---

## 🗑️ Uninstalling

### Helm

```bash
# Delete release
helm uninstall kubeguard-sentinel -n kubeguard-system

# Delete namespace
kubectl delete namespace kubeguard-system

# Delete CRDs (optional)
kubectl delete crd securityscans.kubeguard.io scanauthorizations.kubeguard.io
```

### Terraform

```bash
terraform destroy
```

---

## 📚 Additional Resources

- **Architecture:** See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- **Examples:** See [docs/EXAMPLES.md](docs/EXAMPLES.md)
- **GitLab CI:** See [deployments/gitlab-ci/kubeguard-template.yml](deployments/gitlab-ci/kubeguard-template.yml)
- **Sentinel Policies:** See [deployments/terraform/modules/sentinel/](deployments/terraform/modules/sentinel/)

---

## 📧 Support

**Author:** Tyler Joyner  
**Email:** joynertyler75@gmail.com  
**GitHub:** [@joynerty](https://github.com/joynerty)  

**Issues:** https://github.com/joynerty/kubeguard-sentinel/issues  
**Discussions:** https://github.com/joynerty/kubeguard-sentinel/discussions

---

**Happy scanning! 🛡️**
