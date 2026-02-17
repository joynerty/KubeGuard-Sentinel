# KubeGuard Sentinel - System Architecture

**Author:** Tyler Joyner (joynertyler75@gmail.com)  
**Project:** https://github.com/joynerty/kubeguard-sentinel

---

## 📐 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        User & Integration Layer                      │
│                                                                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌─────────────────┐   │
│  │ kubectl  │  │ Terraform│  │ GitLab CI │  │ Web Dashboard   │   │
│  │   CLI    │  │  Module  │  │ Pipeline  │  │  (Browser)      │   │
│  └────┬─────┘  └────┬─────┘  └─────┬─────┘  └────────┬────────┘   │
└───────┼─────────────┼──────────────┼──────────────────┼────────────┘
        │             │              │                  │
        └─────────────┴──────────────┴──────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Kubernetes API Server                           │
│                    (Authentication & Authorization)                  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    KubeGuard Sentinel Controller                     │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                   AUTHORIZATION LAYER                          │ │
│  │  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────┐ │ │
│  │  │ CRD Validator   │  │ Scope Checker    │  │ Audit Logger │ │ │
│  │  │ • Verify auth   │  │ • Namespace      │  │ • All actions│ │ │
│  │  │ • Check expiry  │  │ • Profile limits │  │ • Timestamps │ │ │
│  │  │ • Validate user │  │ • Exclusions     │  │ • Evidence   │ │ │
│  │  └─────────────────┘  └──────────────────┘  └──────────────┘ │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     SCANNER ENGINE                             │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐  │ │
│  │  │  Discovery   │  │  Enumeration │  │  RBAC Analysis     │  │ │
│  │  │  • Namespaces│  │  • Pods      │  │  • Roles           │  │ │
│  │  │  • Nodes     │  │  • SA        │  │  • Bindings        │  │ │
│  │  │  │Resources  │  │  • Secrets   │  │  • Permissions     │  │ │
│  │  └──────────────┘  └──────────────┘  └────────────────────┘  │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    EXECUTOR ENGINE                             │ │
│  │  ┌────────────────────────────────────────────────────────┐   │ │
│  │  │ Test Categories:                                        │   │ │
│  │  │  ✓ Privilege Escalation (cluster-admin, capabilities)  │   │ │
│  │  │  ✓ Service Account Abuse (default SA, token mounting)  │   │ │
│  │  │  ✓ Pod Breakout (privileged, hostPath, Docker socket)  │   │ │
│  │  │  ✓ Network Policies (missing policies, default-deny)   │   │ │
│  │  │  ✓ Cloud Identity (IRSA, Workload Identity)            │   │ │
│  │  └────────────────────────────────────────────────────────┘   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    REPORTER ENGINE                             │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐  │ │
│  │  │ Risk Scoring │  │  Compliance  │  │ Report Generation  │  │ │
│  │  │ • CVSS calc  │  │ • CIS mapping│  │ • JSON format      │  │ │
│  │  │ • Severity   │  │ • PCI-DSS    │  │ • HTML dashboard   │  │ │
│  │  │ • Aggregation│  │ • NIST 800-  │  │ • SARIF (GitLab)   │  │ │
│  │  └──────────────┘  └──────────────┘  └────────────────────┘  │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    POLICY ENGINE                               │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐  │ │
│  │  │   Sentinel   │  │  OPA/Rego    │  │  Admission Control │  │ │
│  │  │  Policies    │  │  Policies    │  │  (Future)          │  │ │
│  │  └──────────────┘  └──────────────┘  └────────────────────┘  │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STORAGE & OUTPUT LAYER                            │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌─────────────────┐   │
│  │   PVC    │  │    S3    │  │  GitLab   │  │  Notifications  │   │
│  │ Storage  │  │  Bucket  │  │ Artifacts │  │  (Slack/Email)  │   │
│  └──────────┘  └──────────┘  └───────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Component Interactions

### 1. Authorization Flow

```
User → kubectl apply scan.yaml
  ↓
Kubernetes API Server
  ↓
KubeGuard Controller
  ↓
Authorization Validator
  ├─→ Check ScanAuthorization CRD exists
  ├─→ Verify authorization not expired
  ├─→ Validate scope (namespaces, profile)
  ├─→ Check approval ticket format
  └─→ Log authorization event
      ↓
  [APPROVED] → Proceed to scanning
  [DENIED]   → Return error, log rejection
```

### 2. Scanning Workflow

```
Start Scan
  ↓
Discovery Phase (30-60 seconds)
  ├─→ List all namespaces (filter excluded)
  ├─→ Enumerate pods per namespace
  ├─→ List service accounts
  ├─→ Get RBAC roles and bindings
  └─→ Fetch network policies
      ↓
Execution Phase (5-20 minutes depending on profile)
  ├─→ Run Privilege Escalation Tests
  │   ├─ Check cluster-admin bindings
  │   ├─ Scan for dangerous capabilities
  │   └─ Detect escalation permissions
  ├─→ Run Service Account Tests
  │   ├─ Find default SA usage
  │   ├─ Check token automounting
  │   └─ Analyze cross-namespace access
  ├─→ Run Pod Breakout Tests
  │   ├─ Detect privileged containers
  │   ├─ Find host path mounts
  │   └─ Check Docker socket exposure
  ├─→ Run Network Policy Tests
  │   ├─ Verify policies exist
  │   └─ Check default-deny enforcement
  └─→ Run Cloud Identity Tests
      ├─ Validate IRSA annotations
      └─ Check IAM permissions
          ↓
Analysis Phase (1-2 minutes)
  ├─→ Calculate CVSS scores
  ├─→ Classify severity levels
  ├─→ Map to compliance frameworks
  └─→ Generate recommendations
      ↓
Reporting Phase (30 seconds)
  ├─→ Generate JSON report
  ├─→ Create HTML dashboard
  ├─→ Build SARIF for GitLab
  └─→ Update scan status CRD
      ↓
Notification Phase
  ├─→ Send Slack alerts (if configured)
  ├─→ Email scan summary
  └─→ Post to GitLab MR comments
      ↓
Complete
```

---

## 🏭 GitLab CI/CD Integration Architecture

```
GitLab Repository
  │
  ├─ .gitlab-ci.yml
  │    └─ includes: kubeguard-template.yml
  │
  └─ Pipeline Execution
      │
      ┌─────────────┐
      │  VALIDATE   │  Terraform validation
      │   STAGE     │  Sentinel policy checks
      └──────┬──────┘
             │
      ┌──────▼──────┐
      │    PLAN     │  Generate Terraform plan
      │   STAGE     │  Apply Sentinel policies
      └──────┬──────┘  Validate security configs
             │
      ┌──────▼──────┐
      │   DEPLOY    │  Apply Terraform
      │   STAGE     │  Deploy KubeGuard
      └──────┬──────┘  Wait for pods ready
             │
      ┌──────▼──────┐
      │    SCAN     │  Create ScanAuthorization
      │   STAGE     │  Run SecurityScan
      └──────┬──────┘  Wait for completion
             │          Extract results
      ┌──────▼──────┐
      │   REPORT    │  Generate markdown report
      │   STAGE     │  Post to MR comments
      └──────┬──────┘  Upload SARIF
             │
      ┌──────▼──────┐
      │  CLEANUP    │  Delete scan resources
      │   STAGE     │  Archive old scans
      └─────────────┘

Pipeline Artifacts:
  ├─ scan-results.json
  ├─ scan-results.html
  ├─ gl-sast-report.json (SARIF)
  ├─ terraform_outputs.json
  └─ security-report.md
```

### GitLab Security Dashboard Integration

```
SARIF Report → GitLab Security Dashboard
  │
  ├─ Vulnerability List
  │   ├─ Critical: cluster-admin service accounts
  │   ├─ High: Privileged containers
  │   └─ Medium: Default SA usage
  │
  ├─ Trend Analysis
  │   └─ Findings over time
  │
  └─ Merge Request Security Widget
      ├─ New vulnerabilities introduced
      ├─ Resolved vulnerabilities
      └─ Security status (Pass/Fail)
```

---

## 🔐 Terraform Sentinel Policy Workflow

```
Terraform Plan Generated
  ↓
Sentinel Policy Evaluation
  │
  ├─→ Policy 1: require-authorization.sentinel
  │   ├─ Check: authorizationRef exists
  │   ├─ Validate: proper format
  │   └─ Enforcement: hard-mandatory
  │
  ├─→ Policy 2: compliance-score.sentinel
  │   ├─ Check: production scans have compliance
  │   ├─ Validate: minimum scores
  │   └─ Enforcement: soft-mandatory
  │
  ├─→ Policy 3: block-critical.sentinel
  │   ├─ Check: critical findings = 0
  │   ├─ Action: block deployment if critical
  │   └─ Enforcement: hard-mandatory
  │
  ├─→ Policy 4: scan-profile.sentinel
  │   ├─ Check: profile matches environment
  │   ├─ Warn: if production uses 'safe'
  │   └─ Enforcement: advisory
  │
  └─→ Policy 5: namespace-scope.sentinel
      ├─ Check: no system namespace targeting
      ├─ Validate: protected namespaces excluded
      └─ Enforcement: soft-mandatory
          ↓
  [ALL PASS] → terraform apply allowed
  [ANY FAIL] → deployment blocked, report violations
```

---

## 💾 Data Flow & Storage

### Scan Data Lifecycle

```
1. Scan Initiated
   └─→ Generate unique scan ID
   
2. Discovery Data (Temporary)
   ├─→ Store in memory
   └─→ ~10-50MB depending on cluster size
   
3. Test Results (Working)
   ├─→ Store in controller memory
   └─→ ~5-20MB per scan
   
4. Final Reports (Persistent)
   ├─→ JSON: /results/scan-{id}.json (~1-5MB)
   ├─→ HTML: /results/scan-{id}.html (~2-8MB)
   ├─→ SARIF: /results/scan-{id}.sarif (~1-3MB)
   └─→ Storage: PVC or S3
   
5. Long-term Archival
   ├─→ Compress reports (gzip)
   ├─→ Move to S3/GCS/Azure Blob
   └─→ Retention: 90 days default
   
6. Audit Logs (Permanent)
   ├─→ All scan activities
   ├─→ Authorization events
   └─→ Store in dedicated audit DB/S3
```

### Storage Architecture

```
┌────────────────────────────────────────┐
│     KubeGuard Sentinel Pod             │
│  ┌──────────────────────────────────┐  │
│  │  /tmp (emptyDir)                 │  │
│  │  • Temporary scan data           │  │
│  │  • Working files                 │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │  /results (PVC mount)            │  │
│  │  • scan-*.json                   │  │
│  │  • scan-*.html                   │  │
│  │  • scan-*.sarif                  │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
              │
              ├─→ PersistentVolumeClaim (10Gi)
              │   └─→ StorageClass: fast-ssd
              │
              └─→ S3 Bucket (optional)
                  ├─→ Lifecycle: 90 days
                  └─→ Versioning: enabled
```

---

## 🔌 Integration Points

### 1. Kubernetes API Integration

```go
// Scanner uses standard Kubernetes client-go
clientset, err := kubernetes.NewForConfig(config)

// API calls used:
namespaces := clientset.CoreV1().Namespaces().List()
pods := clientset.CoreV1().Pods(ns).List()
serviceAccounts := clientset.CoreV1().ServiceAccounts(ns).List()
roleBindings := clientset.RbacV1().RoleBindings(ns).List()
networkPolicies := clientset.NetworkingV1().NetworkPolicies(ns).List()

// Rate limiting: 100 QPS, burst 200
```

### 2. GitLab API Integration

```bash
# Post scan results to merge request
curl --request POST \
  --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" \
  --data "body=Security scan results..." \
  "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/merge_requests/${CI_MERGE_REQUEST_IID}/notes"

# Upload SARIF report
curl --request POST \
  --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" \
  --form "file=@gl-sast-report.json" \
  "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/security/reports"
```

### 3. Slack Integration

```json
// Webhook payload
{
  "text": "🛡️ Security Scan Complete",
  "attachments": [{
    "color": "danger",
    "fields": [
      {"title": "Critical", "value": "3", "short": true},
      {"title": "High", "value": "5", "short": true},
      {"title": "Risk Score", "value": "6.8/10.0"}
    ]
  }]
}
```

---

## ⚡ Performance Characteristics

### Resource Usage

```
Scanner Pod (per scan):
├─ CPU: 0.5-2 cores (spikes during discovery)
├─ Memory: 512MB-2GB (depending on cluster size)
├─ Network: 100-500 API calls
└─ Duration:
    ├─ Safe profile: 2-5 minutes
    ├─ Standard profile: 5-15 minutes
    └─ Aggressive profile: 15-30 minutes

Cluster Impact:
├─ API Server load: Minimal (rate limited to 100 QPS)
├─ Network traffic: <10MB per scan
└─ Storage I/O: <100MB per scan
```

### Scalability Limits

```
Tested Configurations:
├─ Small cluster: 10 nodes, 100 pods → 3 min scan
├─ Medium cluster: 50 nodes, 500 pods → 8 min scan
├─ Large cluster: 200 nodes, 2000 pods → 25 min scan
└─ XL cluster: 500+ nodes, 5000+ pods → 45 min scan

Concurrent Scans:
├─ Maximum: 3 concurrent scans (configurable)
├─ Queue: FIFO with priority
└─ Resource limits prevent overload
```

---

## 🔒 Security Considerations

### Scanner Security Posture

```yaml
Pod Security:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  
  containerSecurityContext:
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    runAsNonRoot: true
    capabilities:
      drop: [ALL]

Network Policy:
  # Scanner can only talk to API server
  policyTypes: [Ingress, Egress]
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            name: kube-system
      ports:
      - protocol: TCP
        port: 443  # API server only
```

### Data Protection

```
Sensitive Data Handling:
├─ Service account tokens: Never logged
├─ Secrets: Never extracted, only counted
├─ Pod specs: Stored in scan results (review needed)
└─ Audit logs: Encrypted at rest

Access Controls:
├─ RBAC: Least privilege for scanner SA
├─ Authorization: Mandatory ScanAuthorization CRD
├─ Audit: All actions logged with user attribution
└─ Encryption: TLS for all API communications
```

---

## 📊 Monitoring & Observability

### Prometheus Metrics

```prometheus
# Scan metrics
kubeguard_scans_total{profile,status}
kubeguard_scan_duration_seconds{profile,namespace}
kubeguard_findings_total{severity,category,namespace}
kubeguard_compliance_score{framework,namespace}

# System metrics
kubeguard_api_requests_total{endpoint,status}
kubeguard_scan_queue_length
kubeguard_active_scans
```

### Logging

```
Log Levels:
├─ INFO: Scan lifecycle events
├─ WARN: Non-critical issues, policy violations
├─ ERROR: Scan failures, API errors
└─ DEBUG: Detailed discovery data (disabled by default)

Log Format (JSON):
{
  "timestamp": "2024-02-16T14:30:52Z",
  "level": "INFO",
  "component": "scanner",
  "scanId": "scan-abc123",
  "message": "Scan completed successfully",
  "findings": 18,
  "duration": "8m32s"
}
```

---

## 🚀 Deployment Architectures

### 1. Single Cluster Deployment

```
┌──────────────────────────────────┐
│     Production Cluster           │
│                                  │
│  ┌────────────────────────────┐ │
│  │  kubeguard-system NS       │ │
│  │  ├─ Controller Pod         │ │
│  │  ├─ PVC (results)          │ │
│  │  └─ ServiceMonitor         │ │
│  └────────────────────────────┘ │
│                                  │
│  ┌────────────────────────────┐ │
│  │  Application Namespaces    │ │
│  │  (Scanned targets)         │ │
│  └────────────────────────────┘ │
└──────────────────────────────────┘
```

### 2. Multi-Cluster Hub-Spoke

```
┌─────────────────────────────────────┐
│      Hub Cluster (Central)          │
│  ┌───────────────────────────────┐  │
│  │  KubeGuard Central Dashboard  │  │
│  │  • Aggregate reports          │  │
│  │  • Cross-cluster view         │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
              │
      ┌───────┴───────┬───────────┐
      ▼               ▼           ▼
┌──────────┐   ┌──────────┐  ┌──────────┐
│ Prod     │   │ Staging  │  │   Dev    │
│ Cluster  │   │ Cluster  │  │ Cluster  │
│ Scanner  │   │ Scanner  │  │ Scanner  │
└──────────┘   └──────────┘  └──────────┘
```

### 3. GitLab CI/CD Ephemeral

```
GitLab Runner
  └─→ Spin up kubectl pod
      └─→ Apply SecurityScan CRD
          └─→ Wait for completion
              └─→ Download results
                  └─→ Cleanup
```

---

**For questions or contributions:**  
Tyler Joyner - joynertyler75@gmail.com  
GitHub: https://github.com/joynerty/kubeguard-sentinel
