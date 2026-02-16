# 🚀 KubeGuard Sentinel - Quick Setup Guide for Your Environment

## For Users Who Want to Try This Project

This guide will help anyone deploy KubeGuard Sentinel in their own Kubernetes cluster in under 15 minutes.

---

## 📋 Prerequisites

Before starting, ensure you have:

- ✅ **Kubernetes cluster** (1.24+)
  - Minikube (local testing)
  - EKS, GKE, AKS (production)
  - Kind, k3s (development)
  
- ✅ **kubectl** installed and configured
  ```bash
  kubectl version --short
  ```

- ✅ **Helm 3.x** installed
  ```bash
  helm version
  ```

- ✅ **Cluster admin access** (or sufficient RBAC permissions)
  ```bash
  kubectl auth can-i create clusterrolebinding
  ```

---

## 🎯 Three Deployment Options

Choose the option that best fits your needs:

### **Option 1: Quick Start with Helm** (Recommended - 5 minutes)
### **Option 2: Full Infrastructure with Terraform** (10 minutes)
### **Option 3: Standalone Python Agent** (2 minutes - No cluster install)

---

## 🚀 Option 1: Quick Start with Helm

Perfect for: Testing, development, quick evaluation

### Step 1: Clone the Repository

```bash
# Clone from GitHub (or download the zip file)
git clone https://github.com/YOUR-USERNAME/kubeguard-sentinel.git
cd kubeguard-sentinel
```

### Step 2: Install via Helm

```bash
# Create namespace
kubectl create namespace kubeguard-system

# Install the Helm chart
helm install kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --set ethics.requireAuthorization=true \
  --set ethics.dryRunDefault=true \
  --set scanner.defaultProfile=safe

# Verify installation
kubectl get pods -n kubeguard-system
```

Expected output:
```
NAME                                  READY   STATUS    RESTARTS   AGE
kubeguard-sentinel-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
```

### Step 3: Create Your First Authorization

```bash
kubectl apply -f - <<EOF
apiVersion: kubeguard.io/v1alpha1
kind: ScanAuthorization
metadata:
  name: my-first-scan-auth
  namespace: kubeguard-system
spec:
  authorizedBy: "$(whoami)@example.com"
  approvalTicket: "QUICKSTART-001"
  validUntil: "2025-12-31T23:59:59Z"
  scope:
    namespaces:
      - default
    excludeNamespaces:
      - kube-system
      - kube-public
    maxRiskProfile: safe
EOF
```

### Step 4: Run Your First Scan

```bash
kubectl apply -f - <<EOF
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: my-first-scan
  namespace: kubeguard-system
spec:
  authorizationRef: my-first-scan-auth
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

### Step 5: View Results

```bash
# Watch scan progress
kubectl get securityscan my-first-scan -n kubeguard-system -w

# Once complete, view details
kubectl describe securityscan my-first-scan -n kubeguard-system

# Get the scanner pod name
POD=$(kubectl get pod -n kubeguard-system -l app=kubeguard-sentinel -o jsonpath='{.items[0].metadata.name}')

# Download the HTML report
kubectl cp kubeguard-system/$POD:/results/scan-results.html ./my-scan-results.html

# Open the report
open my-scan-results.html  # macOS
xdg-open my-scan-results.html  # Linux
start my-scan-results.html  # Windows
```

### Step 6: View the Dashboard

```bash
# Download the dashboard file
curl -O https://raw.githubusercontent.com/YOUR-USERNAME/kubeguard-sentinel/main/dashboard.html

# Open in browser
open kubeguard-dashboard.html
```

**✅ You're Done!** You've successfully run your first Kubernetes security scan.

---

## 🏗️ Option 2: Full Infrastructure with Terraform

Perfect for: Production deployments, automated infrastructure

### Step 1: Install Terraform

```bash
# macOS
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

### Step 2: Configure AWS/GCP/Azure Credentials

**For AWS EKS:**
```bash
aws configure
export AWS_PROFILE=your-profile
```

**For GCP GKE:**
```bash
gcloud auth application-default login
```

**For Azure AKS:**
```bash
az login
```

### Step 3: Set Up Terraform Variables

```bash
cd deployments/terraform

# Create variables file
cat > terraform.tfvars <<EOF
cluster_name                  = "my-k8s-cluster"
namespace                     = "kubeguard-system"
enable_irsa                   = true  # Set to false for non-AWS
default_scan_profile          = "safe"
storage_size                  = "10Gi"
enable_metrics                = true
ethics_require_authorization  = true

# Optional: Slack notifications
# slack_webhook_url = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# Optional: Email notifications
# email_recipients = ["security@company.com"]
EOF
```

### Step 4: Deploy

```bash
# Initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply changes
terraform apply

# Type 'yes' when prompted
```

### Step 5: Get Deployment Information

```bash
# View outputs
terraform output

# The output will show:
# - Namespace where deployed
# - Example scan command
# - Access instructions
```

### Step 6: Run Your First Scan

Use the command from `terraform output example_scan_command`:

```bash
kubectl apply -f - <<EOF
apiVersion: kubeguard.io/v1alpha1
kind: SecurityScan
metadata:
  name: terraform-scan
  namespace: kubeguard-system
spec:
  authorizationRef: initial-scan-auth  # Auto-created by Terraform
  profile: safe
  tests:
    - all
EOF
```

**✅ Production Ready!** Your infrastructure is deployed and scanning.

---

## 🐍 Option 3: Standalone Python Agent

Perfect for: CI/CD pipelines, one-off scans, no cluster installation needed

### Step 1: Install Dependencies

```bash
# Install Python dependencies
pip install kubernetes

# Or use the requirements file
pip install -r requirements.txt
```

### Step 2: Configure kubectl Access

```bash
# Ensure your kubeconfig is set up
export KUBECONFIG=~/.kube/config

# Or specify explicitly
export KUBECONFIG=/path/to/your/kubeconfig
```

### Step 3: Run a Scan

```bash
cd scripts

# Run the Python agent
python agent.py \
  --scan-name python-quick-scan \
  --auth-ref LOCAL-TEST-001 \
  --profile safe \
  --output ./results.json \
  --dry-run
```

### Step 4: View Results

```bash
# Pretty print JSON results
cat results.json | python -m json.tool

# Or open in your favorite JSON viewer
code results.json  # VS Code
cat results.json | jq '.'  # jq tool
```

**✅ Simple!** No cluster installation required.

---

## 🎨 Setting Up the Dashboard

### Option A: Use Locally

```bash
# Download the dashboard
curl -O https://raw.githubusercontent.com/YOUR-USERNAME/kubeguard-sentinel/main/kubeguard-dashboard.html

# Open in browser
open kubeguard-dashboard.html
```

### Option B: Host on GitHub Pages

```bash
# In your repository
git checkout -b gh-pages
cp kubeguard-dashboard.html index.html
git add index.html
git commit -m "Add dashboard to GitHub Pages"
git push origin gh-pages

# Enable GitHub Pages in repository Settings > Pages
# Your dashboard will be at: https://YOUR-USERNAME.github.io/kubeguard-sentinel/
```

### Option C: Deploy to Netlify/Vercel

```bash
# Netlify
netlify deploy --dir=. --prod

# Vercel
vercel --prod
```

---

## 🔧 Configuration Options

### Changing Scan Profile

Edit your scan YAML:

```yaml
spec:
  profile: safe      # Options: safe, standard, aggressive
```

**Profiles:**
- `safe`: Read-only, non-invasive (recommended for first scan)
- `standard`: Includes pod breakout simulations
- `aggressive`: Advanced testing (requires additional approval)

### Enabling Notifications

**Slack:**
```bash
# Create secret
kubectl create secret generic slack-webhook \
  --namespace kubeguard-system \
  --from-literal=webhook-url='YOUR_WEBHOOK_URL'

# Update Helm
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values \
  --set reporting.slack.enabled=true \
  --set reporting.slack.webhookSecretName=slack-webhook
```

**Email:**
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

### Scheduling Automated Scans

```bash
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values \
  --set scanner.schedule="0 2 * * 0"  # Sunday at 2 AM
```

---

## 📊 Understanding Your Results

### Risk Score Interpretation

| Score | Level | Action |
|-------|-------|--------|
| 9.0-10.0 | 🔴 CRITICAL | Fix immediately |
| 7.0-8.9 | 🟠 HIGH | Plan this week |
| 4.0-6.9 | 🟡 MEDIUM | Next sprint |
| 0.1-3.9 | 🟢 LOW | Routine hardening |

### Common Findings and Fixes

**Finding: "Service Account with cluster-admin"**
```yaml
# ❌ Bad
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: ci-cd

# ✅ Good
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-role
  namespace: ci-cd
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create"]
```

**Finding: "Privileged Pod"**
```yaml
# ❌ Bad
securityContext:
  privileged: true

# ✅ Good
securityContext:
  privileged: false
  capabilities:
    add: ["NET_ADMIN"]  # Only specific capability needed
```

**Finding: "Default Service Account Usage"**
```yaml
# ❌ Bad
spec:
  serviceAccountName: default

# ✅ Good
spec:
  serviceAccountName: my-app-sa
  automountServiceAccountToken: false
```

---

## 🐛 Troubleshooting

### Issue: "Authorization failed"

**Solution:** Ensure you created a ScanAuthorization:
```bash
kubectl get scanauthorization -n kubeguard-system
```

### Issue: "Pod not starting"

**Solution:** Check pod logs:
```bash
kubectl logs -n kubeguard-system -l app=kubeguard-sentinel
kubectl describe pod -n kubeguard-system -l app=kubeguard-sentinel
```

### Issue: "No results available"

**Solution:** Verify PVC is mounted:
```bash
kubectl get pvc -n kubeguard-system
kubectl describe pod -n kubeguard-system -l app=kubeguard-sentinel | grep -A5 Volumes
```

### Issue: "RBAC permission denied"

**Solution:** Verify service account permissions:
```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:kubeguard-system:kubeguard-sentinel
```

---

## 🔄 Updating

### Helm Update
```bash
# Pull latest changes
git pull

# Upgrade release
helm upgrade kubeguard-sentinel ./deployments/helm \
  --namespace kubeguard-system \
  --reuse-values
```

### Terraform Update
```bash
# Pull latest changes
git pull

# Apply updates
terraform apply
```

---

## 🗑️ Uninstalling

### Helm
```bash
# Delete Helm release
helm uninstall kubeguard-sentinel -n kubeguard-system

# Delete namespace
kubectl delete namespace kubeguard-system

# Delete CRDs (optional)
kubectl delete crd securityscans.kubeguard.io
kubectl delete crd scanauthorizations.kubeguard.io
```

### Terraform
```bash
terraform destroy
```

---

## 📚 Additional Resources

- **Full Documentation:** See `docs/` folder
- **Architecture Details:** `docs/ARCHITECTURE.md`
- **Deployment Guide:** `docs/DEPLOYMENT.md`
- **Example Scans:** `docs/EXAMPLES.md`
- **GitHub Issues:** Report bugs or request features

---

## 🤝 Contributing

Found a bug? Have a feature request?

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

---

## 📧 Support

- **Documentation:** Check the `docs/` folder
- **Issues:** GitHub Issues
- **Questions:** Open a discussion on GitHub

---

## ⭐ Star the Project

If you find this useful, please star the repository on GitHub!

```bash
# Share with your team
git clone https://github.com/YOUR-USERNAME/kubeguard-sentinel.git
```

---

**Built with ❤️ for the Kubernetes security community**

Happy scanning! 🛡️
