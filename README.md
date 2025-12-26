# DevOps Interview Questions [Updated December 2025] 🔥

#### Updates 🔥
- [12/2025] Added scenario-based questions
- [9/2025] Added some questions frequently asked in MNC Interviews
- [7/2025] Added advanced questions across all DevOps domains
- [6/2025] Included real-world troubleshooting scenarios from top tech companies
- [5/2025] Added GitOps, Platform Engineering, and SRE-focused questions

---

## 📋 Table of Contents

1. [CI/CD & Pipeline Engineering](#1-cicd--pipeline-engineering)
2. [Infrastructure as Code (Terraform & Ansible)](#2-infrastructure-as-code-terraform--ansible)
3. [Docker & Containerization](#3-docker--containerization)
4. [Kubernetes](#4-kubernetes)
5. [Cloud Platforms (AWS/Azure/GCP)](#5-cloud-platforms-awsazuregcp)
6. [Monitoring, Logging & Observability](#6-monitoring-logging--observability)
7. [Linux & System Administration](#7-linux--system-administration)
8. [Networking & Load Balancing](#8-networking--load-balancing)
9. [Security & DevSecOps](#9-security--devsecops)
10. [Git & Version Control](#10-git--version-control)
11. [Scripting & Automation](#11-scripting--automation)
12. [Troubleshooting & Incident Response](#12-troubleshooting--incident-response)
13. [Architecture & System Design](#13-architecture--system-design)
14. [GitOps & Platform Engineering](#14-gitops--platform-engineering)
15. [SRE Concepts](#15-sre-concepts)

---

## 1. CI/CD & Pipeline Engineering

### Medium Level

**Q1: Your Jenkins pipeline takes 45 minutes to complete. The team complains about slow feedback loops. How would you optimize it?**

<details>
<summary>💡 Approach</summary>

- **Parallelize stages**: Run unit tests, linting, and security scans concurrently
- **Use caching**: Cache dependencies (npm, pip, maven) between builds
- **Incremental builds**: Only build/test changed components using tools like Nx or Turborepo
- **Agent optimization**: Use dedicated agents with SSDs, more RAM, and faster network
- **Split test suites**: Distribute tests across multiple agents
- **Skip unnecessary steps**: Use conditional execution based on changed files
- **Artifact management**: Upload/download artifacts instead of rebuilding

```groovy
// Example: Parallel stages in Jenkins
stage('Tests') {
    parallel {
        stage('Unit Tests') { steps { sh 'npm run test:unit' } }
        stage('Integration Tests') { steps { sh 'npm run test:integration' } }
        stage('Security Scan') { steps { sh 'trivy image myapp:latest' } }
    }
}
```
</details>

---

**Q2: You have a microservices architecture with 20 services. How would you handle deployment when services have interdependencies?**

<details>
<summary>💡 Approach</summary>

- **Dependency graph**: Build a directed acyclic graph (DAG) of service dependencies
- **Contract testing**: Use Pact or similar for consumer-driven contract tests
- **Feature flags**: Deploy behind flags to decouple deployment from release
- **Canary deployments**: Gradually roll out changes to detect issues early
- **Backward compatibility**: Maintain API versioning, never break contracts
- **Service mesh**: Use Istio/Linkerd for traffic shifting during deployments
- **Deployment order**: Deploy dependencies first (bottom-up approach)

</details>

---

**Q3: Your team accidentally pushed secrets to a public GitHub repository. Walk me through your incident response.**

<details>
<summary>💡 Approach</summary>

**Immediate actions (within minutes):**
1. Rotate ALL exposed credentials immediately
2. Revoke tokens/API keys from the provider's console
3. Use `git filter-branch` or BFG Repo-Cleaner to remove secrets from history
4. Force push the cleaned history
5. Contact GitHub to clear caches if needed

**Post-incident:**
- Audit logs for any unauthorized access using exposed credentials
- Implement pre-commit hooks with tools like `detect-secrets` or `gitleaks`
- Set up GitHub secret scanning and push protection
- Use external secret management (HashiCorp Vault, AWS Secrets Manager)
- Conduct a blameless postmortem

```bash
# Remove secrets from git history using BFG
bfg --replace-text passwords.txt my-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```
</details>

---

### Advanced Level

**Q4: Design a CI/CD pipeline for a monorepo containing 50+ microservices. How do you ensure only affected services are built and deployed?**

<details>
<summary>💡 Approach</summary>

- **Affected analysis**: Use tools like Nx, Turborepo, or Bazel for change detection
- **Build graph**: Maintain dependency graph, rebuild only affected services and their dependents
- **Tagging strategy**: Tag commits with affected service names for deployment filtering
- **Matrix builds**: Dynamically generate build matrix based on changed paths
- **Shared libraries**: Version shared libs independently, trigger rebuilds on changes
- **Caching layers**: Use distributed caching (Nx Cloud, Turborepo Remote Cache)

```yaml
# GitHub Actions: Dynamic matrix based on changed paths
jobs:
  detect-changes:
    outputs:
      services: ${{ steps.changes.outputs.services }}
    steps:
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            api: 'services/api/**'
            auth: 'services/auth/**'
            billing: 'services/billing/**'
  
  build:
    needs: detect-changes
    strategy:
      matrix:
        service: ${{ fromJson(needs.detect-changes.outputs.services) }}
```
</details>

---

**Q5: Your production deployment failed at 2 AM, and the rollback also failed because the database migration was already applied. How do you handle this?**

<details>
<summary>💡 Approach</summary>

**Immediate actions:**
1. Assess impact: Is the system partially functional?
2. Consider forward-fix if rollback isn't viable
3. Apply database rollback migration if safe (down migration)

**If down migration is risky:**
- Deploy a hotfix version compatible with new schema
- Use feature flags to disable broken functionality
- Apply expand-contract pattern for future migrations

**Prevention strategies:**
- **Expand-contract migrations**: Add new columns first, migrate data, then remove old
- **Blue-green with database versioning**: Both versions work with same schema
- **Separate deployment from migration**: Deploy code that handles both schemas
- **Database rollback scripts**: Always have tested rollback procedures

```sql
-- Expand-Contract Pattern Example
-- Step 1: Expand (add new column)
ALTER TABLE users ADD COLUMN email_v2 VARCHAR(255);

-- Step 2: Migrate (application writes to both)
UPDATE users SET email_v2 = email WHERE email_v2 IS NULL;

-- Step 3: Contract (after all services updated)
ALTER TABLE users DROP COLUMN email;
ALTER TABLE users RENAME COLUMN email_v2 TO email;
```
</details>

---

**Q6: How would you implement a zero-downtime deployment for a stateful application with active WebSocket connections?**

<details>
<summary>💡 Approach</summary>

- **Connection draining**: Configure graceful shutdown, stop accepting new connections while completing existing ones
- **Sticky sessions**: Use session affinity during transition period
- **Session externalization**: Store session state in Redis/external store
- **WebSocket reconnection**: Implement client-side automatic reconnection with exponential backoff
- **Load balancer configuration**: Set connection draining timeout appropriately
- **Pod disruption budgets**: Ensure minimum available replicas during rolling updates

```yaml
# Kubernetes: Graceful shutdown for WebSocket apps
apiVersion: v1
kind: Pod
spec:
  terminationGracePeriodSeconds: 60
  containers:
  - name: websocket-app
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 30"]  # Allow connection draining
```
</details>

---

## 2. Infrastructure as Code (Terraform & Ansible)

### Medium Level

**Q7: Your Terraform state file is corrupted, and you have 100+ resources in production. How do you recover?**

<details>
<summary>💡 Approach</summary>

**If you have state backups:**
1. Restore from last known good state backup
2. Run `terraform plan` to identify drift
3. Fix discrepancies with `terraform import` or `terraform state rm`

**If no backups exist:**
1. Create new state file
2. Use `terraform import` for each resource
3. Consider using `terraformer` to generate HCL from existing infrastructure
4. Validate with `terraform plan` (should show no changes)

**Prevention:**
- Enable S3 versioning for state bucket
- Use state locking with DynamoDB
- Regular state backups
- Consider Terraform Cloud/Enterprise for managed state

```bash
# Import existing resources
terraform import aws_instance.web i-1234567890abcdef0
terraform import aws_s3_bucket.data my-bucket-name

# Use terraformer to reverse-engineer existing infra
terraformer import aws --resources=ec2_instance,s3 --regions=us-east-1
```
</details>

---

**Q8: You need to manage infrastructure across 5 AWS accounts and 3 environments (dev, staging, prod). How do you structure your Terraform code?**

<details>
<summary>💡 Approach</summary>

**Recommended structure:**
```
infrastructure/
├── modules/                    # Reusable modules
│   ├── vpc/
│   ├── eks/
│   └── rds/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf         # Points to dev state bucket
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── accounts/
    ├── shared-services/
    ├── workloads-dev/
    ├── workloads-staging/
    └── workloads-prod/
```

**Key practices:**
- Use **workspaces** sparingly (prefer directory-based separation)
- **Remote state data sources** to reference across environments
- **Assume roles** across accounts with proper IAM setup
- **Version pinning** for modules and providers
- **Terragrunt** for DRY configuration across environments

```hcl
# Using assume_role for cross-account access
provider "aws" {
  alias  = "production"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::PROD_ACCOUNT_ID:role/TerraformRole"
  }
}
```
</details>

---

**Q9: Your Terraform apply is stuck on a resource that's taking forever. How do you troubleshoot and what are your options?**

<details>
<summary>💡 Approach</summary>

**Troubleshooting:**
1. Check AWS Console for resource status (e.g., RDS creation can take 15+ mins)
2. Enable debug logging: `TF_LOG=DEBUG terraform apply`
3. Check if resource is genuinely stuck vs. just slow

**Options if truly stuck:**
- `Ctrl+C` to interrupt (state may be inconsistent)
- Use `-target` to skip problematic resource temporarily
- Run `terraform refresh` to sync state
- Manually fix in console, then import/remove from state

**Common slow operations:**
- RDS instances (10-20 mins)
- EKS clusters (15-20 mins)
- CloudFront distributions (15-30 mins)
- NAT Gateways (5-10 mins)

```bash
# Force unlock if state is locked
terraform force-unlock LOCK_ID

# Target specific resources
terraform apply -target=aws_instance.web
```
</details>

---

### Advanced Level

**Q10: How would you implement a self-service infrastructure platform where developers can provision their own environments without direct access to Terraform?**

<details>
<summary>💡 Approach</summary>

**Architecture:**
1. **Git-based workflow**: Developers submit PRs with variable files
2. **Atlantis/Terraform Cloud**: Automated plan/apply on PR events
3. **Service catalog**: Pre-approved infrastructure patterns as modules
4. **Policy as Code**: Use OPA/Sentinel to enforce guardrails
5. **RBAC**: Limit what each team can provision

**Components:**
- **Backstage/Port**: Developer portal for requesting infrastructure
- **Crossplane**: Kubernetes-native infrastructure management
- **Custom API**: Wrapper service with approval workflows

```yaml
# Example: Developer submits this YAML
apiVersion: platform.company.io/v1
kind: Environment
metadata:
  name: team-alpha-dev
spec:
  tier: development
  components:
    - type: kubernetes-namespace
      name: team-alpha
    - type: postgres
      size: small
    - type: redis
      size: micro
```
</details>

---

**Q11: You're migrating from manually created infrastructure to Terraform. You have 500+ resources. Describe your strategy.**

<details>
<summary>💡 Approach</summary>

**Phase 1: Discovery**
- Use AWS Config, Resource Groups, or cloud-specific tools to inventory resources
- Identify dependencies and relationships
- Categorize by criticality and complexity

**Phase 2: Import**
- Use `terraformer` for bulk import
- Validate generated code against best practices
- Refactor into proper module structure

**Phase 3: Iterative Rollout**
- Start with non-critical resources (S3, IAM policies)
- Progress to stateless workloads (EC2, Lambda)
- Finally, migrate stateful resources (RDS, ElastiCache)
- Validate each phase with `terraform plan` showing no changes

**Phase 4: Governance**
- Implement CI/CD for Terraform
- Add policy enforcement (Checkov, tfsec)
- Enable drift detection

```bash
# Bulk import with terraformer
terraformer import aws \
  --resources=vpc,subnet,security_group,ec2_instance,s3,rds \
  --regions=us-east-1,us-west-2 \
  --profile=production
```
</details>

---

**Q12: How do you handle Terraform state drift in a large organization where people occasionally make manual changes?**

<details>
<summary>💡 Approach</summary>

**Detection:**
- Scheduled `terraform plan` in CI (drift detection jobs)
- AWS Config rules to detect non-compliant resources
- CloudTrail alerts for manual console changes

**Prevention:**
- **Strict IAM policies**: Read-only access to console for most users
- **Breakglass procedures**: Documented process for emergency manual changes
- **Tagging enforcement**: Resources without "managed-by: terraform" tag trigger alerts

**Remediation:**
- Auto-remediation pipeline that runs `terraform apply` on drift
- Slack/PagerDuty alerts for detected drift
- Quarterly reconciliation reviews

```yaml
# GitHub Actions: Drift Detection
name: Terraform Drift Detection
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
jobs:
  drift-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Terraform Plan
        run: |
          terraform init
          terraform plan -detailed-exitcode -out=plan.out
          if [ $? -eq 2 ]; then
            echo "DRIFT DETECTED" 
            # Send alert
          fi
```
</details>

---

## 3. Docker & Containerization

### Medium Level

**Q13: Your Docker image is 2.5GB. The security team wants it under 500MB. How do you reduce the size?**

<details>
<summary>💡 Approach</summary>

1. **Multi-stage builds**: Separate build and runtime stages
2. **Alpine/Distroless base images**: Switch from ubuntu to alpine or distroless
3. **Remove unnecessary files**: Clear caches, temp files, docs
4. **Combine RUN commands**: Reduce layers
5. **Use .dockerignore**: Exclude unnecessary files from context
6. **Don't install dev dependencies**: Only production deps in final image

```dockerfile
# Before: 2.5GB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]

# After: ~150MB
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```
</details>

---

**Q14: A container works perfectly locally but fails in production with "Permission denied" errors. How do you debug?**

<details>
<summary>💡 Approach</summary>

**Common causes:**
1. **User mismatch**: Container runs as non-root in prod but root locally
2. **Read-only filesystem**: Production may mount volumes as read-only
3. **Security contexts**: Kubernetes securityContext restrictions
4. **SELinux/AppArmor**: Additional security layers in production

**Debugging steps:**
```bash
# Check running user
docker exec container_id whoami
docker exec container_id id

# Check file permissions
docker exec container_id ls -la /path/to/file

# Check mounted volumes
docker inspect container_id | jq '.[0].Mounts'

# Run with different user
docker run --user 1000:1000 myimage
```

**Fix:**
```dockerfile
# Ensure proper permissions in Dockerfile
RUN chown -R node:node /app
USER node
```
</details>

---

**Q15: How would you implement container resource limits in a way that prevents noisy neighbor problems without over-provisioning?**

<details>
<summary>💡 Approach</summary>

**Strategy:**
1. **Baseline profiling**: Monitor actual resource usage for 2 weeks
2. **Set requests based on P50**: Normal operation level
3. **Set limits based on P99**: Peak usage with buffer
4. **Use vertical pod autoscaler (VPA)**: Automatic right-sizing recommendations

**Example based on profiling:**
```yaml
# If app uses: P50=256MB, P99=512MB, max spike=768MB
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "768Mi"
    cpu: "500m"
```

**Monitoring:**
- Alert when containers approach 80% of limits
- Track throttling events for CPU
- Monitor OOMKilled events for memory
</details>

---

### Advanced Level

**Q16: Design a container build system that supports 10 teams building 50+ images daily with security scanning, caching, and reproducibility.**

<details>
<summary>💡 Approach</summary>

**Architecture:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Git Push  │────▶│   CI System  │────▶│  Registry   │
└─────────────┘     │  (Actions/   │     │  (ECR/GCR)  │
                    │   Jenkins)   │     └─────────────┘
                    └──────┬───────┘            │
                           │                    ▼
                    ┌──────▼───────┐     ┌─────────────┐
                    │ Build Cache  │     │ Vulnerability│
                    │  (BuildKit)  │     │   Scanner   │
                    └──────────────┘     └─────────────┘
```

**Key components:**
- **Kaniko/BuildKit**: Rootless, cacheable builds
- **Shared cache**: S3/GCS for layer caching across teams
- **Trivy/Grype**: Vulnerability scanning in pipeline
- **Cosign**: Image signing for supply chain security
- **Base image management**: Curated, scanned base images updated weekly
- **Reproducibility**: Pin all versions, use digest-based references

```yaml
# Buildkit with cache export
docker buildx build \
  --cache-from=type=s3,region=us-east-1,bucket=build-cache \
  --cache-to=type=s3,region=us-east-1,bucket=build-cache \
  --push -t myapp:latest .
```
</details>

---

**Q17: A container is experiencing intermittent connectivity issues, but only in production Kubernetes cluster. How do you approach debugging?**

<details>
<summary>💡 Approach</summary>

**Step-by-step debugging:**

1. **Verify DNS resolution:**
```bash
kubectl exec -it pod-name -- nslookup service-name
kubectl exec -it pod-name -- cat /etc/resolv.conf
```

2. **Check network policies:**
```bash
kubectl get networkpolicies -n namespace
kubectl describe networkpolicy policy-name
```

3. **Test from ephemeral debug container:**
```bash
kubectl debug pod-name -it --image=nicolaka/netshoot -- /bin/bash
# Inside: tcpdump, curl, dig, traceroute
```

4. **Check for DNS caching issues:**
- CoreDNS logs: `kubectl logs -n kube-system -l k8s-app=kube-dns`
- ndots configuration causing slow lookups

5. **Examine kube-proxy and CNI:**
```bash
kubectl logs -n kube-system -l k8s-app=kube-proxy
# Check iptables rules on node
```

6. **Intermittent issues checklist:**
- Connection timeouts: Check readiness probes, pod restarts
- DNS timeouts: Adjust ndots, use FQDN
- Service mesh sidecar issues (Istio/Envoy)
</details>

---

## 4. Kubernetes

> 📘 **For comprehensive Kubernetes interview questions, refer to the dedicated guide:**
> 
> **[Kubernetes Interview Questions - Detailed Guide](https://github.com/JustInCache/kubernetes-interview-questions)**
> 
> The dedicated repository covers:
> - Core K8s concepts (Pods, Deployments, Services)
> - Networking and Ingress
> - Storage and StatefulSets
> - Security (RBAC, Network Policies)
> - Troubleshooting scenarios
> - Advanced topics (Service Mesh, Operators)

### Quick Scenario Questions

**Q18: A pod is stuck in `ImagePullBackOff`. Walk through your debugging process.**

<details>
<summary>💡 Approach</summary>

```bash
# 1. Get pod events
kubectl describe pod pod-name | grep -A 10 Events

# 2. Check image name for typos
kubectl get pod pod-name -o jsonpath='{.spec.containers[*].image}'

# 3. Verify image exists
docker pull image-name:tag

# 4. Check imagePullSecrets
kubectl get pod pod-name -o jsonpath='{.spec.imagePullSecrets}'
kubectl get secret docker-secret -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d

# 5. Test registry authentication from node
# SSH to node, then: docker login registry.example.com
```

**Common causes:**
- Wrong image name/tag
- Private registry without imagePullSecrets
- Registry rate limiting (Docker Hub)
- Network issues to registry
</details>

---

**Q19: CPU throttling is affecting your application performance, but you've set generous limits. What's happening?**

<details>
<summary>💡 Approach</summary>

**Understanding CFS throttling:**
- Kubernetes uses CPU quotas based on limits
- Default CFS period is 100ms
- If limit is 1 CPU, container gets 100ms every 100ms
- Burst workloads can be throttled even below limit

**Diagnosis:**
```bash
# Check throttling metrics
kubectl top pod pod-name
cat /sys/fs/cgroup/cpu/cpu.stat  # Inside container
# Look for nr_throttled, throttled_time
```

**Solutions:**
1. **Increase limits**: Give more headroom for bursts
2. **Remove CPU limits**: Only set requests (controversial but effective)
3. **Use Burstable QoS**: Allows bursting when resources available
4. **Tune application**: Reduce thread count, optimize hot paths

```yaml
# Option: Remove CPU limits (set only requests)
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    memory: "512Mi"  # Keep memory limit, remove CPU limit
```
</details>

---

## 5. Cloud Platforms (AWS/Azure/GCP)

### Medium Level

**Q20: Your application suddenly started throwing "Rate Exceeded" errors from AWS APIs. How do you handle this?**

<details>
<summary>💡 Approach</summary>

**Immediate mitigation:**
1. **Implement exponential backoff**: Already built into AWS SDKs
2. **Add jitter**: Prevent thundering herd
3. **Reduce API call frequency**: Cache responses where possible

**Long-term solutions:**
```python
# Custom backoff configuration (Python/boto3)
from botocore.config import Config

config = Config(
    retries={
        'max_attempts': 10,
        'mode': 'adaptive'  # Adaptive retry mode
    }
)
client = boto3.client('dynamodb', config=config)
```

**Architectural changes:**
- **Batch operations**: Use batch APIs instead of individual calls
- **SQS for decoupling**: Queue requests during spikes
- **Service quotas**: Request limit increases from AWS
- **Caching**: Use ElastiCache for frequently accessed data
- **Event-driven**: Use EventBridge instead of polling
</details>

---

**Q21: You need to migrate a 5TB database to AWS with minimal downtime (under 5 minutes). Describe your approach.**

<details>
<summary>💡 Approach</summary>

**Strategy: AWS DMS with CDC**

1. **Initial bulk load (during maintenance window):**
   - Create DMS replication instance
   - Full load from source to target RDS
   - Takes ~12-24 hours for 5TB

2. **Continuous replication (CDC):**
   - DMS captures ongoing changes
   - Keeps target in sync with source
   - Monitor replication lag

3. **Cutover (< 5 minutes):**
   - Stop writes to source
   - Wait for CDC to catch up (< 1 min)
   - Switch DNS/application to target
   - Verify and enable writes

```
Source DB ──CDC──▶ DMS Instance ──▶ Target RDS
    │                                    │
    └─── Full Load (~24hrs) ────────────┘
    └─── CDC (ongoing) ──────────────────┘
```

**Risk mitigation:**
- Rehearse cutover in staging
- Prepare rollback plan
- Pre-validate data with AWS DMS validation
</details>

---

**Q22: Design a multi-region active-active architecture on AWS for a global e-commerce application.**

<details>
<summary>💡 Approach</summary>

**Architecture:**
```
                    ┌─────────────┐
                    │ Route 53    │
                    │ (Latency)   │
                    └──────┬──────┘
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌─────────┐  ┌─────────┐  ┌─────────┐
        │ US-East │  │ EU-West │  │ AP-South│
        │   ALB   │  │   ALB   │  │   ALB   │
        └────┬────┘  └────┬────┘  └────┬────┘
             │            │            │
        ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
        │   EKS   │  │   EKS   │  │   EKS   │
        └────┬────┘  └────┬────┘  └────┬────┘
             │            │            │
        ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
        │ Aurora  │◀─┼─Global ─┼─▶│ Aurora  │
        │ Primary │  │   DB    │  │ Replica │
        └─────────┘  └─────────┘  └─────────┘
```

**Key components:**
- **Route 53 latency routing**: Direct users to nearest region
- **Aurora Global Database**: < 1 second replication lag
- **ElastiCache Global Datastore**: Session/cache replication
- **S3 Cross-Region Replication**: Static assets
- **DynamoDB Global Tables**: Shopping cart, preferences

**Conflict resolution:**
- Use timestamps for last-write-wins
- Event sourcing for critical data
- Region-specific ID prefixes to avoid collisions
</details>

---

### Advanced Level

**Q23: You're running a high-traffic application on AWS. Costs have tripled in 3 months. How do you approach cost optimization?**

<details>
<summary>💡 Approach</summary>

**Phase 1: Visibility (Week 1)**
- Enable Cost Explorer and Cost Anomaly Detection
- Tag all resources for cost allocation
- Identify top 5 cost drivers

**Phase 2: Quick Wins (Week 2-3)**
```bash
# Find unused resources
- Unattached EBS volumes
- Unused Elastic IPs
- Orphaned snapshots
- Idle load balancers
- Over-provisioned RDS instances
```

**Phase 3: Right-sizing (Week 3-4)**
- Use AWS Compute Optimizer recommendations
- Downsize instances based on CloudWatch metrics
- Switch gp2 to gp3 EBS volumes (20% savings)

**Phase 4: Purchasing Models (Ongoing)**
```
| Workload Type    | Strategy                      |
|------------------|-------------------------------|
| Steady-state     | Reserved Instances (1-3 yr)   |
| Variable         | Savings Plans (compute)       |
| Fault-tolerant   | Spot Instances (70-90% off)   |
| Dev/Test         | Scheduled scaling (nights/weekends off) |
```

**Phase 5: Architecture Review**
- Move to Graviton instances (40% better price-performance)
- Serverless for variable workloads
- Data tiering (S3 Intelligent-Tiering, Glacier)
- NAT Gateway costs → NAT Instances or Gateway Endpoints
</details>

---

**Q24: An EC2 instance in a private subnet cannot reach the internet to download packages. Walk through your troubleshooting.**

<details>
<summary>💡 Approach</summary>

**Systematic debugging:**

```bash
# 1. Verify instance metadata
curl http://169.254.169.254/latest/meta-data/

# 2. Check route table
# Private subnet should have: 0.0.0.0/0 → NAT Gateway
aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=subnet-xxx"

# 3. Verify NAT Gateway
aws ec2 describe-nat-gateways --filter "Name=vpc-id,Values=vpc-xxx"
# NAT Gateway must be in PUBLIC subnet with EIP

# 4. Check Security Groups
# Outbound: Allow 443, 80 to 0.0.0.0/0

# 5. Check NACLs
# Often forgotten - verify both inbound and outbound rules

# 6. Verify NAT Gateway's route table
# NAT Gateway's subnet must have: 0.0.0.0/0 → Internet Gateway
```

**Common issues:**
- NAT Gateway in private subnet (must be in public)
- Missing route to NAT Gateway in private route table
- NACL blocking ephemeral ports (1024-65535 inbound)
- Security group blocking outbound traffic
- NAT Gateway route table missing IGW route
</details>

---

## 6. Monitoring, Logging & Observability

### Medium Level

**Q25: Your Prometheus server is consuming 80GB of RAM and scrape jobs are timing out. How do you optimize?**

<details>
<summary>💡 Approach</summary>

**Immediate relief:**
1. **Reduce retention**: Lower `--storage.tsdb.retention.time`
2. **Increase scrape interval**: From 15s to 30s or 60s for non-critical targets
3. **Drop unnecessary metrics**: Use `metric_relabel_configs`

```yaml
# Drop high-cardinality metrics
scrape_configs:
  - job_name: 'kubernetes-pods'
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'go_gc_.*|python_gc_.*'  # Drop verbose metrics
        action: drop
      - source_labels: [__name__]
        regex: '.*_bucket'  # Drop histogram buckets if not needed
        action: drop
```

**Architectural improvements:**
- **Federation**: Hierarchical Prometheus for large clusters
- **Remote write**: Offload to Thanos/Cortex/Mimir for long-term storage
- **Vertical scaling**: More RAM, SSD storage
- **Recording rules**: Pre-compute expensive queries

**Cardinality investigation:**
```promql
# Find high-cardinality metrics
topk(10, count by (__name__)({__name__=~".+"}))
```
</details>

---

**Q26: Logs from your microservices are difficult to correlate. How do you implement distributed tracing?**

<details>
<summary>💡 Approach</summary>

**Implementation strategy:**

1. **Choose tracing backend**: Jaeger, Zipkin, or managed (AWS X-Ray, Datadog APM)
2. **Instrument applications**: Use OpenTelemetry (vendor-neutral)
3. **Propagate context**: Ensure trace headers flow through all services

```python
# Python with OpenTelemetry
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Setup
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter(endpoint="jaeger:4317"))
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

# Usage
@app.route('/api/order')
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("order.type", "express")
        # ... order logic
```

**Correlation pattern:**
```
Log format: [timestamp] [trace_id] [span_id] [service] message
```

**Key headers to propagate:**
- `traceparent` (W3C standard)
- `X-Request-ID` (legacy)
</details>

---

### Advanced Level

**Q27: Design an alerting strategy that minimizes alert fatigue while ensuring critical issues are caught.**

<details>
<summary>💡 Approach</summary>

**Alert hierarchy:**

```
┌─────────────────────────────────────────────────┐
│            Page (Wake someone up)               │
│  - Service completely down                      │
│  - Data loss risk                               │
│  - Security breach                              │
├─────────────────────────────────────────────────┤
│            Ticket (Business hours)              │
│  - Degraded performance                         │
│  - Approaching capacity limits                  │
│  - Non-critical service issues                  │
├─────────────────────────────────────────────────┤
│            Log/Dashboard (Awareness)            │
│  - Informational events                         │
│  - Minor anomalies                              │
│  - Debugging information                        │
└─────────────────────────────────────────────────┘
```

**Alert quality guidelines:**
- **Symptom-based, not cause-based**: "High error rate" not "Pod restarted"
- **Actionable**: Every alert must have a runbook
- **Time-delayed**: Flap prevention (e.g., "for 5m" in Prometheus)
- **SLO-based**: Alert on burn rate, not individual errors

```yaml
# Good alert: SLO burn rate
- alert: HighErrorBurnRate
  expr: |
    sum(rate(http_requests_total{status=~"5.."}[5m])) 
    / sum(rate(http_requests_total[5m])) 
    > 14.4 * 0.001  # Burning through 14.4x error budget
  for: 5m
  labels:
    severity: page
  annotations:
    summary: "Error budget burn rate critical"
    runbook_url: "https://wiki/runbooks/high-error-rate"
```

**Regular maintenance:**
- Monthly alert review: silence unused alerts
- Postmortem analysis: was alert useful?
- Alert effectiveness metrics
</details>

---

## 7. Linux & System Administration

### Medium Level

**Q28: A server is running out of disk space, but you can't find any large files. What could be happening?**

<details>
<summary>💡 Approach</summary>

**Common causes:**

1. **Deleted files still held open by processes:**
```bash
# Find deleted but open files
lsof +L1 | grep deleted

# Solution: Restart the process holding the file
# Or truncate: echo "" > /proc/<pid>/fd/<fd_number>
```

2. **Hidden files/directories:**
```bash
du -sh /var/* 2>/dev/null | sort -hr | head -20
du -sh .[^.]* * 2>/dev/null | sort -hr  # Include hidden
```

3. **Reserved blocks for root:**
```bash
# Check reserved space (default 5% on ext4)
tune2fs -l /dev/sda1 | grep -i reserved

# Reduce if safe (not root filesystem)
tune2fs -m 1 /dev/sda1
```

4. **Sparse files appearing smaller than actual:**
```bash
ls -lsh  # First column shows actual disk usage
```

5. **Bind mounts hiding data:**
```bash
# Unmount and check underneath
umount /mnt/data && ls /mnt/data
```
</details>

---

**Q29: System load is high but CPU usage appears low. What's the explanation?**

<details>
<summary>💡 Approach</summary>

**Load average includes:**
- Processes running on CPU
- Processes waiting for CPU
- **Processes in uninterruptible sleep (D state)** - Usually I/O wait

**Diagnosis:**
```bash
# Check for I/O wait
vmstat 1 5  # Look at 'wa' column

# Check for processes in D state
ps aux | awk '$8 ~ /D/'

# Identify I/O bottleneck
iotop -oP

# Check disk latency
iostat -x 1 5  # Look at await, %util columns
```

**Common causes:**
- Slow disk (failing drive, RAID rebuild)
- NFS mount issues (network latency)
- Saturated disk I/O
- Memory pressure causing swap I/O

**Solutions:**
- Add more RAM to reduce disk I/O
- Upgrade to SSD/NVMe
- Fix network issues for NFS
- Identify and optimize I/O-heavy processes
</details>

---

**Q30: How would you investigate a memory leak in a Linux process without restarting it?**

<details>
<summary>💡 Approach</summary>

**Step 1: Confirm memory growth**
```bash
# Watch memory usage over time
watch -n 5 'ps -p <pid> -o pid,vsz,rss,pmem,cmd'

# Or use pidstat
pidstat -r -p <pid> 5
```

**Step 2: Analyze memory maps**
```bash
# View memory regions
cat /proc/<pid>/smaps | grep -E "^(Size|Rss|Swap):" | awk '{sum+=$2} END {print sum/1024 " MB"}'

# Detailed breakdown
pmap -x <pid>
```

**Step 3: Use gdb for heap analysis (careful in production)**
```bash
# Attach to running process
gdb -p <pid>
(gdb) info proc mappings
(gdb) dump memory /tmp/heap.bin 0x<start> 0x<end>
(gdb) detach
```

**Step 4: Use specialized tools**
```bash
# For Java
jmap -histo:live <pid>
jcmd <pid> GC.heap_info

# For Python
# Add: import tracemalloc; tracemalloc.start()
```

**Step 5: Monitor with Valgrind (dev/staging only)**
```bash
valgrind --leak-check=full ./myprogram
```
</details>

---

### Advanced Level

**Q31: Design a process for kernel upgrades across 500 servers with zero downtime.**

<details>
<summary>💡 Approach</summary>

**Strategy: Rolling canary updates with live patching**

**Phase 1: Preparation**
- Test kernel in staging environment
- Run load tests and chaos engineering
- Prepare rollback procedure
- Document known issues and workarounds

**Phase 2: Canary deployment (5% of fleet)**
```bash
# Group servers by criticality
- Tier 1: Canary hosts (non-critical, diverse workloads)
- Tier 2: Non-production
- Tier 3: Production non-critical
- Tier 4: Production critical
```

**Phase 3: Rolling updates with kexec (faster reboot)**
```bash
# Kexec: Skip BIOS, direct kernel load
kexec -l /boot/vmlinuz-new --initrd=/boot/initrd-new --reuse-cmdline
kexec -e
```

**Phase 4: For truly zero-downtime (live patching)**
```bash
# Use kpatch/livepatch for security patches
kpatch install livepatch-module.ko

# Verify
kpatch list
```

**Automation pipeline:**
```yaml
# Ansible rolling update
- hosts: servers
  serial: "10%"  # Update 10% at a time
  max_fail_percentage: 5
  tasks:
    - name: Upgrade kernel
      yum: name=kernel state=latest
    - name: Reboot
      reboot:
        reboot_timeout: 300
    - name: Verify services
      wait_for:
        port: "{{ service_port }}"
```
</details>

---

**Q32: A process is consuming 100% CPU. How do you identify what it's doing without stopping it?**

<details>
<summary>💡 Approach</summary>

**Layer 1: System-level analysis**
```bash
# Get initial overview
top -p <pid> -H  # Show threads
htop -p <pid>

# Check what the process is doing
strace -p <pid> -c  # Summary of syscalls
strace -p <pid> -f  # Follow forks, real-time
```

**Layer 2: CPU profiling**
```bash
# Using perf (most powerful)
perf top -p <pid>  # Real-time hotspots
perf record -p <pid> -g sleep 30  # Record for 30s
perf report  # Analyze

# Generate flame graph
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

**Layer 3: Application-specific**
```bash
# For Java
jstack <pid>  # Thread dump
async-profiler for flame graphs

# For Python
py-spy top --pid <pid>
py-spy record -o profile.svg --pid <pid>

# For Node.js
node --inspect=<port> && Chrome DevTools
```

**Layer 4: Check for infinite loops**
```bash
# Attach with gdb
gdb -p <pid>
(gdb) thread apply all bt  # All thread backtraces
(gdb) detach
```
</details>

---

## 8. Networking & Load Balancing

### Medium Level

**Q33: Users report intermittent connection timeouts. Your monitoring shows the servers are healthy. How do you investigate?**

<details>
<summary>💡 Approach</summary>

**Step-by-step investigation:**

```bash
# 1. Check connection states
netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c | sort -rn
ss -s  # Socket statistics summary

# 2. Look for connection queue overflow
netstat -s | grep -i listen
# "times the listen queue of a socket overflowed"

# 3. Check for TIME_WAIT accumulation
ss -tan state time-wait | wc -l

# 4. Verify load balancer health checks
curl -v http://backend:health

# 5. Check for SYN flood or conntrack exhaustion
dmesg | grep -i "conntrack"
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max
```

**Common causes and fixes:**
| Symptom | Cause | Fix |
|---------|-------|-----|
| Large listen queue | Slow app accept | Increase `somaxconn`, optimize app |
| Many TIME_WAIT | Short-lived connections | Enable `tw_reuse`, use connection pooling |
| Conntrack full | Too many connections | Increase conntrack_max, reduce timeout |
| Packet drops | Network saturation | Scale horizontally, increase bandwidth |

```bash
# Tune kernel parameters
sysctl -w net.core.somaxconn=65535
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.netfilter.nf_conntrack_max=1000000
```
</details>

---

**Q34: Explain how you would debug a situation where only 30% of requests reach your backend servers through a load balancer.**

<details>
<summary>💡 Approach</summary>

**Possible scenarios:**

1. **Health check failures:**
```bash
# Check LB perspective
aws elb describe-instance-health --load-balancer-name my-lb

# Check backend health endpoint
for server in $(cat servers.txt); do
  curl -o /dev/null -s -w "%{http_code} - $server\n" http://$server/health
done
```

2. **Uneven weight distribution:**
```bash
# Check if weights are configured correctly
# In nginx:
upstream backend {
    server 192.168.1.1 weight=3;  # Gets 3x traffic
    server 192.168.1.2 weight=1;
}
```

3. **Sticky sessions gone wrong:**
- Users stuck to unhealthy servers
- Session affinity causing imbalance

4. **Connection draining:**
- Servers in draining state after deployment

5. **Network ACLs/Security groups:**
- Some backends blocked at network level

**Debugging approach:**
```bash
# Compare requests received at each backend
# Backend 1
grep "GET /api" access.log | wc -l

# Check load balancer access logs
# AWS ALB logs in S3 show target selection

# Verify from LB directly
tcpdump -i eth0 port 80 -c 100
```
</details>

---

### Advanced Level

**Q35: Design a global load balancing strategy for an application that needs to handle 1 million concurrent connections.**

<details>
<summary>💡 Approach</summary>

**Architecture:**

```
                     ┌──────────────────────┐
                     │    DNS (Route 53)    │
                     │  GeoDNS / Latency    │
                     └──────────┬───────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   US Region   │       │   EU Region   │       │  APAC Region  │
│   Anycast IP  │       │   Anycast IP  │       │   Anycast IP  │
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
┌───────▼───────┐       ┌───────▼───────┐       ┌───────▼───────┐
│   L4 (NLB)    │       │   L4 (NLB)    │       │   L4 (NLB)    │
│ 10M conn/s    │       │ 10M conn/s    │       │ 10M conn/s    │
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
┌───────▼───────┐       ┌───────▼───────┐       ┌───────▼───────┐
│   L7 (Envoy)  │       │   L7 (Envoy)  │       │   L7 (Envoy)  │
│   Cluster     │       │   Cluster     │       │   Cluster     │
└───────────────┘       └───────────────┘       └───────────────┘
```

**Key components:**

1. **DNS layer (Global):**
   - GeoDNS for regional routing
   - Health-based failover
   - TTL: 60 seconds for quick failover

2. **L4 Load Balancer:**
   - AWS NLB / GCP Network LB
   - Handles millions of connections
   - Low latency, no SSL termination

3. **L7 Load Balancer:**
   - Envoy / NGINX / HAProxy
   - SSL termination, routing logic
   - Connection pooling to backends

**Kernel tuning for 1M connections:**
```bash
# Per server handling 100k connections
sysctl -w net.core.somaxconn=65535
sysctl -w net.ipv4.ip_local_port_range="1024 65535"
sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sysctl -w fs.file-max=2097152
ulimit -n 1048576
```
</details>

---

## 9. Security & DevSecOps

### Medium Level

**Q36: Your security scan found 500 vulnerabilities in your container images. How do you prioritize and address them?**

<details>
<summary>💡 Approach</summary>

**Prioritization matrix:**

```
                    ┌─────────────────────────────────────┐
                    │           CVSS Score                │
                    │   Low    Medium    High   Critical  │
┌───────────────────┼─────────────────────────────────────┤
│ Internet-facing   │   P4      P3       P1      P0      │
│ Internal service  │   P5      P4       P2      P1      │
│ Dev/Test only     │   P6      P5       P4      P3      │
└───────────────────┴─────────────────────────────────────┘
```

**Step 1: Reduce noise**
```bash
# Focus on fixable vulnerabilities
trivy image myapp:latest --ignore-unfixed

# Filter by severity
trivy image myapp:latest --severity HIGH,CRITICAL
```

**Step 2: Triage by exploitability**
- Check EPSS score (Exploit Prediction Scoring System)
- Known exploits in the wild (KEV catalog)
- Network exposure required?

**Step 3: Systematic fixes**
```dockerfile
# Update base image (fixes many at once)
FROM node:18.19-alpine  # Instead of node:18-alpine

# Pin and update dependencies
RUN apk update && apk upgrade --no-cache
```

**Step 4: Automation**
- Block builds with Critical CVEs
- Weekly automated base image updates
- Dependabot/Renovate for dependencies
- SLA: Critical=24h, High=7d, Medium=30d
</details>

---

**Q37: How would you implement secrets management for an application running in Kubernetes with zero secrets in git?**

<details>
<summary>💡 Approach</summary>

**Option 1: External Secrets Operator + AWS Secrets Manager**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: db-secret
  data:
    - secretKey: password
      remoteRef:
        key: prod/database
        property: password
```

**Option 2: HashiCorp Vault with Sidecar Injector**
```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/agent-inject-secret-db-creds: "secret/data/db"
    vault.hashicorp.com/role: "myapp"
spec:
  containers:
    - name: myapp
      # Secret available at /vault/secrets/db-creds
```

**Option 3: SOPS + Age for encrypted secrets in git**
```bash
# Encrypt secrets
sops --encrypt --age <public-key> secrets.yaml > secrets.enc.yaml

# Decrypt in CI/CD
sops --decrypt secrets.enc.yaml | kubectl apply -f -
```

**Best practices:**
- Rotate secrets automatically
- Audit secret access
- Use short-lived tokens where possible
- Enable secrets versioning
</details>

---

### Advanced Level

**Q38: Design a supply chain security strategy for your CI/CD pipeline.**

<details>
<summary>💡 Approach</summary>

**SLSA Framework Implementation:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Supply Chain Security                      │
├─────────────────────────────────────────────────────────────┤
│  Source         │  Build           │  Deploy                 │
│  ────────       │  ─────           │  ──────                 │
│  • Signed       │  • Hermetic      │  • Signed images        │
│    commits      │    builds        │  • Admission            │
│  • Branch       │  • SBOM          │    controller           │
│    protection   │    generation    │  • Runtime              │
│  • Dep scanning │  • Provenance    │    protection           │
└─────────────────────────────────────────────────────────────┘
```

**Implementation layers:**

**1. Source integrity:**
```yaml
# Require signed commits
git config --global commit.gpgsign true

# GitHub branch protection:
- Require signed commits
- Require PR reviews
- No force pushes to main
```

**2. Dependency security:**
```bash
# Lock dependencies with checksums
npm ci --ignore-scripts  # Use package-lock.json
pip install --require-hashes -r requirements.txt
```

**3. Build attestation:**
```yaml
# Generate SLSA provenance
- uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
  with:
    image: myimage
    digest: ${{ steps.build.outputs.digest }}
```

**4. Image signing:**
```bash
# Sign with Cosign
cosign sign --key cosign.key myregistry.io/myapp:latest

# Verify in Kubernetes with admission controller (Kyverno)
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-signature
spec:
  rules:
    - name: check-signature
      match:
        resources:
          kinds: ["Pod"]
      verifyImages:
        - image: "myregistry.io/*"
          key: |-
            -----BEGIN PUBLIC KEY-----
            ...
            -----END PUBLIC KEY-----
```

**5. SBOM generation:**
```bash
# Generate SBOM
syft myimage:latest -o spdx-json > sbom.json

# Attach to image
cosign attach sbom --sbom sbom.json myregistry.io/myapp:latest
```
</details>

---

## 10. Git & Version Control

### Medium Level

**Q39: Your team's git history is messy with merge commits everywhere. How would you implement a cleaner workflow?**

<details>
<summary>💡 Approach</summary>

**Recommended: Trunk-based development with squash merges**

```
main ──●──●──●──●──●──●──●──●──●──●──▶
        ▲     ▲     ▲     ▲     ▲
        │     │     │     │     │
       feat  fix   feat  fix  feat
        │     │     │     │     │
        └─────┴─────┴─────┴─────┘
        Short-lived feature branches
        (Squash merge to main)
```

**Implementation:**

1. **Branch protection rules:**
```
- Require squash merging (no merge commits)
- Require linear history
- Delete branches after merge
- Require up-to-date before merge
```

2. **Rebase workflow:**
```bash
# Before merging, always rebase on main
git fetch origin
git rebase origin/main

# Interactive rebase to clean commits
git rebase -i HEAD~5
```

3. **Conventional commits:**
```
feat(auth): add OAuth2 login support
fix(api): handle null response from payment service
docs(readme): update deployment instructions
```

4. **Pre-commit hooks:**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/commitizen-tools/commitizen
    hooks:
      - id: commitizen
        stages: [commit-msg]
```

**Benefits:**
- Clean, linear history
- Easy to bisect for bugs
- Meaningful commit messages
- Simple rollbacks
</details>

---

**Q40: A developer accidentally merged a feature branch to production that wasn't ready. How do you handle the rollback?**

<details>
<summary>💡 Approach</summary>

**Option 1: Revert the merge commit (safest)**
```bash
# Find the merge commit
git log --oneline --merges -5

# Revert the merge (keep the mainline)
git revert -m 1 <merge-commit-sha>
git push origin main

# When ready to re-merge later:
# Revert the revert, then merge
git revert <revert-commit-sha>
```

**Option 2: Reset (if not yet pulled by others)**
```bash
# Hard reset to before the merge
git reset --hard HEAD~1
git push --force-with-lease origin main

# ⚠️ Only if: few people, quick action, you're certain
```

**Option 3: Feature flag (if already deployed widely)**
```javascript
// Disable feature without reverting code
if (featureFlags.isEnabled('new-checkout')) {
  return newCheckoutFlow();
}
return oldCheckoutFlow();
```

**Post-incident:**
- Add branch protection rules
- Require PR approvals
- Add CI checks that must pass
- Consider environment promotion (dev → staging → prod branches)

**Prevention automation:**
```yaml
# GitHub Action: Block direct pushes to main
name: Block Direct Push
on: push
jobs:
  check:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "::error::Direct pushes to main are not allowed"
          exit 1
```
</details>

---

## 11. Scripting & Automation

### Medium Level

**Q41: Write a script to find all pods in a Kubernetes cluster that have been restarting frequently and notify via Slack.**

<details>
<summary>💡 Approach</summary>

```bash
#!/bin/bash
# find-crashloop-pods.sh

RESTART_THRESHOLD=5
SLACK_WEBHOOK="${SLACK_WEBHOOK_URL}"

# Find pods with restart count > threshold
PROBLEM_PODS=$(kubectl get pods --all-namespaces -o json | jq -r --argjson threshold "$RESTART_THRESHOLD" '
  .items[] | 
  select(.status.containerStatuses != null) |
  select(.status.containerStatuses[].restartCount > $threshold) |
  {
    namespace: .metadata.namespace,
    name: .metadata.name,
    restarts: .status.containerStatuses[0].restartCount,
    reason: (.status.containerStatuses[0].state | keys[0])
  } |
  "\(.namespace)/\(.name) - Restarts: \(.restarts) - State: \(.reason)"
')

if [ -n "$PROBLEM_PODS" ]; then
  # Format message
  MESSAGE=$(cat <<EOF
{
  "blocks": [
    {
      "type": "header",
      "text": {"type": "plain_text", "text": "⚠️ Pods with High Restart Count"}
    },
    {
      "type": "section",
      "text": {"type": "mrkdwn", "text": "\`\`\`${PROBLEM_PODS}\`\`\`"}
    }
  ]
}
EOF
)
  
  # Send to Slack
  curl -s -X POST -H 'Content-type: application/json' \
    --data "$MESSAGE" "$SLACK_WEBHOOK"
fi
```

**Enhanced version with Python:**
```python
#!/usr/bin/env python3
from kubernetes import client, config
import requests
import os

config.load_incluster_config()  # or load_kube_config() locally
v1 = client.CoreV1Api()

THRESHOLD = 5
SLACK_WEBHOOK = os.environ['SLACK_WEBHOOK_URL']

problem_pods = []
pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:
    if pod.status.container_statuses:
        for cs in pod.status.container_statuses:
            if cs.restart_count > THRESHOLD:
                problem_pods.append({
                    'namespace': pod.metadata.namespace,
                    'name': pod.metadata.name,
                    'restarts': cs.restart_count,
                    'container': cs.name
                })

if problem_pods:
    msg = "\n".join([f"{p['namespace']}/{p['name']} ({p['container']}): {p['restarts']} restarts" 
                     for p in problem_pods])
    requests.post(SLACK_WEBHOOK, json={"text": f"⚠️ High restart pods:\n```{msg}```"})
```
</details>

---

**Q42: Automate the process of cleaning up old Docker images that are older than 30 days but keep at least the 5 most recent tags per repository.**

<details>
<summary>💡 Approach</summary>

```python
#!/usr/bin/env python3
"""Docker registry cleanup script"""

import boto3
from datetime import datetime, timedelta
from collections import defaultdict

def cleanup_ecr_images(dry_run=True):
    ecr = boto3.client('ecr')
    cutoff_date = datetime.now(tz=None) - timedelta(days=30)
    keep_count = 5
    
    # Get all repositories
    repos = ecr.describe_repositories()['repositories']
    
    for repo in repos:
        repo_name = repo['repositoryName']
        print(f"\nProcessing: {repo_name}")
        
        # Get all images in repository
        images = []
        paginator = ecr.get_paginator('describe_images')
        for page in paginator.paginate(repositoryName=repo_name):
            images.extend(page['imageDetails'])
        
        # Sort by push date (newest first)
        images.sort(key=lambda x: x.get('imagePushedAt', datetime.min), reverse=True)
        
        # Identify images to delete
        to_delete = []
        for i, image in enumerate(images):
            pushed_at = image.get('imagePushedAt')
            tags = image.get('imageTags', ['<untagged>'])
            
            # Keep first N images regardless of age
            if i < keep_count:
                print(f"  KEEP (recent): {tags[0]}")
                continue
            
            # Delete if older than cutoff
            if pushed_at and pushed_at.replace(tzinfo=None) < cutoff_date:
                to_delete.append({'imageDigest': image['imageDigest']})
                print(f"  DELETE: {tags[0]} - {pushed_at}")
            else:
                print(f"  KEEP (not old enough): {tags[0]}")
        
        # Execute deletion
        if to_delete and not dry_run:
            ecr.batch_delete_image(
                repositoryName=repo_name,
                imageIds=to_delete[:100]  # API limit
            )
            print(f"  Deleted {len(to_delete)} images")
        elif to_delete:
            print(f"  Would delete {len(to_delete)} images (dry run)")

if __name__ == '__main__':
    import sys
    dry_run = '--execute' not in sys.argv
    if dry_run:
        print("DRY RUN MODE - use --execute to actually delete")
    cleanup_ecr_images(dry_run=dry_run)
```

**Lifecycle policy approach (preferred for ECR):**
```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 5 tagged images",
      "selection": {
        "tagStatus": "tagged",
        "countType": "imageCountMoreThan",
        "countNumber": 5
      },
      "action": {"type": "expire"}
    },
    {
      "rulePriority": 2,
      "description": "Delete untagged images older than 1 day",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countNumber": 1,
        "countUnit": "days"
      },
      "action": {"type": "expire"}
    }
  ]
}
```
</details>

---

## 12. Troubleshooting & Incident Response

### Medium Level

**Q43: Production is down. Walk me through your first 5 minutes of incident response.**

<details>
<summary>💡 Approach</summary>

**Minute 0-1: Acknowledge and assess**
```
1. Acknowledge the alert
2. Quick health check:
   - Can I access the application?
   - What's the error message users see?
   - When did it start? (correlate with deployments/changes)
```

**Minute 1-2: Determine scope**
```bash
# Quick checks
curl -I https://myapp.com/health
kubectl get pods -A | grep -v Running
aws ecs describe-services --cluster prod | jq '.services[].runningCount'
```

**Minute 2-3: Check recent changes**
```bash
# What was deployed recently?
kubectl rollout history deployment/myapp
git log --oneline -5 main

# Any infrastructure changes?
terraform show -json | jq '.values.root_module.resources | length'
```

**Minute 3-4: Decide on immediate action**
```
IF recent deployment → ROLLBACK
IF traffic spike → SCALE UP
IF dependency down → FAILOVER
IF unclear → CONTINUE INVESTIGATION
```

**Minute 4-5: Communicate and escalate**
```
- Update status page
- Post in incident channel
- Tag on-call for affected services
- Start incident bridge if major
```

**Parallel actions:**
- Someone investigates
- Someone communicates
- Someone prepares rollback

**Quick rollback command (have ready):**
```bash
kubectl rollout undo deployment/myapp
# or
aws ecs update-service --cluster prod --service myapp --task-definition myapp:previous
```
</details>

---

**Q44: After a deployment, 5% of users report seeing errors while others work fine. How do you debug this?**

<details>
<summary>💡 Approach</summary>

**Potential causes for partial failures:**

1. **Canary/rolling deployment in progress**
```bash
kubectl get pods -o wide  # Check if old and new versions running
kubectl rollout status deployment/myapp
```

2. **Cache inconsistencies**
```bash
# Some users hitting cached responses, some hitting new code
# Check if CDN, Redis, or local caches involved
redis-cli KEYS "session:*" | head
```

3. **Session affinity issues**
```bash
# Users stuck to specific pods
kubectl describe service myapp | grep Session
```

4. **Feature flag targeting**
```bash
# 5% might be in experiment group
curl -s https://api.launchdarkly.com/flags/myfeature | jq '.targeting'
```

5. **Regional/edge issues**
```bash
# Check if specific regions affected
grep "500" access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head
# Map IPs to regions
```

**Debugging approach:**
```bash
# 1. Find pattern in affected users
grep "ERROR" app.log | awk '{print $user_id}' | sort | uniq > affected_users.txt

# 2. Check request characteristics
# Are they using specific endpoints, browsers, regions?

# 3. Compare pod logs
kubectl logs -l app=myapp --prefix | grep ERROR

# 4. Check load balancer distribution
kubectl top pods  # Is traffic evenly distributed?

# 5. Test specific pods directly
kubectl port-forward pod/myapp-abc123 8080:8080
curl localhost:8080/api/test
```
</details>

---

### Advanced Level

**Q45: Describe how you would implement and run a game day (chaos engineering exercise) for your production systems.**

<details>
<summary>💡 Approach</summary>

**Phase 1: Planning (2 weeks before)**

```markdown
## Game Day Runbook

### Objective
Test system resilience when primary database fails

### Scope
- Production-like staging environment
- Duration: 2 hours
- Rollback trigger: Error rate > 10% for 5 minutes

### Participants
- Game Master: Controls chaos
- Observers: Document behavior
- Responders: Fix issues (hands-off until asked)

### Experiments
1. Kill primary RDS instance
2. Inject 500ms latency to cache
3. Exhaust connection pool
```

**Phase 2: Preparation**

```yaml
# Chaos Mesh experiment definition
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: db-failure
spec:
  action: pod-kill
  mode: one
  selector:
    labelSelectors:
      "app": "postgres"
  scheduler:
    cron: "@every 30m"  # For continuous chaos
```

**Phase 3: Execution**

```bash
# Pre-game checklist
- [ ] Stakeholders notified
- [ ] Status page updated with maintenance
- [ ] Rollback procedures tested
- [ ] Monitoring dashboards ready
- [ ] Communication channel active

# Execute chaos
chaos-mesh inject -f db-failure.yaml

# Monitor
watch -n 5 'curl -s http://myapp/metrics | grep error_rate'
```

**Phase 4: Documentation**

```markdown
## Findings

| Time | Event | Expected | Actual | Gap |
|------|-------|----------|--------|-----|
| 10:05 | DB killed | Failover in 30s | Failover in 2min | Need to tune health checks |
| 10:07 | App recovery | Resume in 60s | Manual restart needed | Connection pool not recovering |

## Action Items
1. Reduce RDS health check interval (P1)
2. Implement connection pool reset on error (P2)
3. Add runbook for manual recovery (P3)
```

**Tools:**
- AWS Fault Injection Simulator
- Chaos Mesh (Kubernetes)
- Gremlin (SaaS)
- Litmus Chaos
</details>

---

## 13. Architecture & System Design

### Medium Level

**Q46: Design a log aggregation system that handles 1TB of logs per day from 500 servers.**

<details>
<summary>💡 Approach</summary>

**Architecture:**

```
Servers (500)              Collection           Storage & Query
┌─────────┐              ┌───────────┐         ┌─────────────┐
│ App     │──filebeat──▶│           │         │             │
│ Server  │              │  Kafka    │────────▶│OpenSearch   │
└─────────┘              │  Cluster  │         │  Cluster    │
                         │           │         │             │
┌─────────┐              │  (Buffer) │         │ Hot: 7 days │
│ App     │──filebeat──▶│           │────────▶│ Warm: 30d   │
│ Server  │              └───────────┘         │ Cold: S3    │
└─────────┘                    │               └─────────────┘
                               │                      │
                         ┌─────▼─────┐          ┌─────▼─────┐
                         │  Logstash │          │  Grafana  │
                         │ (Process) │          │   Query   │
                         └───────────┘          └───────────┘
```

**Capacity planning:**
```
1TB/day = ~11.5 MB/s average
Peak (2x) = ~23 MB/s

Kafka:
- 3 brokers minimum
- Replication factor 2
- Retention: 24 hours
- Partitions: 12 per topic (for parallelism)

OpenSearch:
- 3 data nodes (hot) - 2TB NVMe each
- 2 data nodes (warm) - 4TB SSD each
- 3 master nodes (small instances)
- Index per day, rollover at 50GB
```

**Key optimizations:**
```yaml
# Filebeat - batch and compress
output.kafka:
  hosts: ["kafka:9092"]
  topic: "logs-%{[agent.name]}"
  compression: gzip
  bulk_max_size: 2048

# OpenSearch - optimize mapping
mappings:
  properties:
    message:
      type: text
      index: false  # Don't index raw message
    level:
      type: keyword  # Fast filtering
    timestamp:
      type: date
```

**Cost optimization:**
- S3 for cold storage (> 30 days)
- Use OpenSearch index lifecycle policies
- Sample verbose logs (debug → 10%)
</details>

---

**Q47: Design a feature flag system that supports gradual rollouts and A/B testing.**

<details>
<summary>💡 Approach</summary>

**System design:**

```
┌─────────────────────────────────────────────────────────────┐
│                     Feature Flag Service                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌─────────────────┐    │
│  │  Admin   │───▶│  Flag API    │───▶│  PostgreSQL     │    │
│  │  UI      │    │  Server      │    │  (Flag configs) │    │
│  └──────────┘    └──────┬───────┘    └─────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│                  ┌──────────────┐                           │
│                  │    Redis     │                           │
│                  │  (Cache +    │                           │
│                  │   Pub/Sub)   │                           │
│                  └──────┬───────┘                           │
│                         │                                    │
└─────────────────────────┼────────────────────────────────────┘
                          │
         ┌────────────────┼────────────────┐
         ▼                ▼                ▼
    ┌─────────┐      ┌─────────┐      ┌─────────┐
    │  App 1  │      │  App 2  │      │  App 3  │
    │  (SDK)  │      │  (SDK)  │      │  (SDK)  │
    └─────────┘      └─────────┘      └─────────┘
```

**Flag evaluation logic:**
```python
class FeatureFlag:
    def is_enabled(self, flag_key: str, user: User) -> bool:
        flag = self.get_flag(flag_key)
        
        # Kill switch check
        if flag.globally_disabled:
            return False
        if flag.globally_enabled:
            return True
            
        # User targeting
        if user.id in flag.enabled_users:
            return True
        if user.id in flag.disabled_users:
            return False
            
        # Percentage rollout (consistent hashing)
        hash_value = mmh3.hash(f"{flag_key}:{user.id}") % 100
        if hash_value < flag.rollout_percentage:
            return True
            
        # Segment matching
        for segment in flag.segments:
            if self.user_matches_segment(user, segment):
                return True
                
        return flag.default_value
```

**Gradual rollout example:**
```yaml
flag: new_checkout_flow
enabled: true
rollout:
  - percentage: 1    # Day 1: 1% of users
    start: "2025-01-01"
  - percentage: 10   # Day 3: 10%
    start: "2025-01-03"
  - percentage: 50   # Day 5: 50%
    start: "2025-01-05"
  - percentage: 100  # Day 7: GA
    start: "2025-01-07"
targeting:
  - segment: beta_users
    value: true
  - attribute: country
    operator: in
    values: ["US", "CA"]
    value: true
```

**SDK design principles:**
- Local caching with TTL
- Fallback to defaults on failure
- Async flag updates via SSE/WebSocket
- Minimal latency impact (< 1ms)
</details>

---

## 14. GitOps & Platform Engineering

### Medium Level

**Q48: Compare ArgoCD and Flux. When would you choose one over the other?**

<details>
<summary>💡 Approach</summary>

**Feature comparison:**

| Feature | ArgoCD | Flux |
|---------|--------|------|
| UI | Rich built-in UI | Third-party (Weave GitOps) |
| Multi-cluster | Native support | Requires setup |
| RBAC | Built-in, fine-grained | Kubernetes native |
| Helm support | Native | Native |
| Kustomize | Native | Native |
| Resource hooks | Yes (PreSync, Sync, PostSync) | Limited |
| Diff preview | Yes (in UI) | CLI only |
| Notifications | Built-in | Separate controller |

**Choose ArgoCD when:**
```
✓ You need a polished UI for developers
✓ Managing many clusters from one place
✓ Complex deployment workflows with hooks
✓ Compliance requirements (audit logs, RBAC)
✓ Team is less GitOps-experienced (UI helps)
```

**Choose Flux when:**
```
✓ Kubernetes-native approach preferred
✓ Already using Helm operator patterns
✓ Simpler, more composable architecture
✓ Want tighter integration with source control
✓ Building a platform with custom controllers
```

**Hybrid approach (common):**
```yaml
# Use Flux for infrastructure (Terraform Controller, Crossplane)
# Use ArgoCD for application deployments

flux/
├── infrastructure/     # Flux manages
│   ├── terraform/
│   └── crossplane/
└── apps/              # ArgoCD manages
    ├── frontend/
    └── backend/
```
</details>

---

**Q49: Design an internal developer platform that enables self-service infrastructure provisioning.**

<details>
<summary>💡 Approach</summary>

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Portal                         │
│                      (Backstage)                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐   ┌──────────────┐   ┌─────────────────┐     │
│  │ Service  │   │  Templates   │   │   Scorecards    │     │
│  │ Catalog  │   │  (Scaffolder)│   │   (Tech Radar)  │     │
│  └──────────┘   └──────────────┘   └─────────────────┘     │
│                                                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   GitOps Repository                          │
├─────────────────────────────────────────────────────────────┤
│  environments/                                               │
│  ├── dev/                                                   │
│  │   └── team-alpha-app/    ◄── Generated by template      │
│  └── prod/                                                  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Control Plane (Kubernetes)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐   ┌──────────────┐   ┌─────────────────┐     │
│  │ Crossplane│   │   ArgoCD    │   │    Kyverno      │     │
│  │ (Infra)  │   │  (Apps)      │   │   (Policy)      │     │
│  └──────────┘   └──────────────┘   └─────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

**Self-service flow:**
```yaml
# 1. Developer fills form in Backstage
# Template: "Create new microservice"
parameters:
  - name: serviceName
    title: Service Name
  - name: owner
    title: Team
  - name: database
    title: Database Type
    enum: [postgres, mysql, none]

# 2. Template generates PR
steps:
  - id: create-repo
    action: github:repo:create
  - id: create-infra
    action: fetch:template
    input:
      url: ./skeleton/crossplane
  - id: create-argocd-app
    action: argocd:create-application

# 3. PR triggers ArgoCD sync
# 4. Crossplane provisions resources
```

**Guardrails:**
```yaml
# Kyverno policy - enforce resource limits
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resources
spec:
  validationFailureAction: enforce
  rules:
    - name: check-resources
      match:
        resources:
          kinds: ["Deployment"]
      validate:
        message: "CPU/memory limits are required"
        pattern:
          spec:
            template:
              spec:
                containers:
                  - resources:
                      limits:
                        memory: "?*"
                        cpu: "?*"
```
</details>

---

## 15. SRE Concepts

### Medium Level

**Q50: Explain how you would implement SLOs for a payment processing service.**

<details>
<summary>💡 Approach</summary>

**Step 1: Define SLIs (Service Level Indicators)**
```yaml
SLIs:
  availability:
    description: "Percentage of successful payment requests"
    formula: "successful_payments / total_payments * 100"
    
  latency:
    description: "Payment processing time at p99"
    formula: "histogram_quantile(0.99, payment_duration_seconds)"
    
  correctness:
    description: "Payments processed without reconciliation errors"
    formula: "payments_without_errors / total_payments * 100"
```

**Step 2: Set SLOs (Service Level Objectives)**
```yaml
SLOs:
  availability:
    target: 99.95%  # 21.6 minutes downtime/month
    window: 30 days
    
  latency_p99:
    target: 500ms
    window: 30 days
    
  correctness:
    target: 99.99%  # Financial accuracy is critical
    window: 30 days
```

**Step 3: Calculate error budgets**
```
Monthly error budget for 99.95% availability:
- Total minutes: 43,200 (30 days)
- Error budget: 43,200 × 0.0005 = 21.6 minutes

If we've used 15 minutes this month:
- Remaining: 6.6 minutes
- Burn rate: 15/21.6 = 69.4%
```

**Step 4: Implement alerting on burn rate**
```yaml
# Alert on fast burn (page)
- alert: HighErrorBudgetBurn
  expr: |
    (
      sum(rate(payment_errors_total[1h])) 
      / sum(rate(payment_requests_total[1h]))
    ) > 14.4 * 0.0005  # 14.4x burn rate
  for: 5m
  labels:
    severity: page
  annotations:
    summary: "Burning through monthly error budget in < 2 days"

# Alert on slow burn (ticket)
- alert: SlowErrorBudgetBurn  
  expr: |
    (
      sum(rate(payment_errors_total[6h])) 
      / sum(rate(payment_requests_total[6h]))
    ) > 3 * 0.0005  # 3x burn rate
  for: 30m
  labels:
    severity: ticket
```

**Step 5: Operational decisions**
```
IF error_budget_remaining < 20%:
  - Freeze non-critical deployments
  - Focus on reliability improvements
  - Review recent changes for issues

IF error_budget_remaining < 5%:
  - Freeze all deployments
  - All-hands on reliability
  - Executive escalation
```
</details>

---

**Q51: A service is consistently burning through its error budget. What systematic approach would you take to improve reliability?**

<details>
<summary>💡 Approach</summary>

**Phase 1: Analysis (Week 1)**
```bash
# Identify top error contributors
Top 5 error sources:
1. Database timeouts (35%)
2. Third-party API failures (25%)
3. Memory pressure crashes (20%)
4. Bad deployments (15%)
5. Network issues (5%)
```

**Phase 2: Prioritize by impact × effort**

| Issue | Impact | Effort | Priority |
|-------|--------|--------|----------|
| DB timeouts | High | Medium | P1 |
| API failures | High | Low | P1 |
| Memory crashes | Medium | Medium | P2 |
| Bad deployments | Medium | Low | P2 |

**Phase 3: Implement improvements**

```yaml
# P1: Database timeouts
Solutions:
  - Add connection pooling
  - Implement circuit breaker
  - Add read replicas
  - Query optimization

# P1: Third-party API failures
Solutions:
  - Implement retry with exponential backoff
  - Add circuit breaker (Hystrix/Resilience4j)
  - Cache responses where possible
  - Add fallback behavior

# P2: Memory pressure
Solutions:
  - Right-size containers (VPA recommendations)
  - Fix memory leaks (profiling)
  - Add memory limits with proper buffer
```

**Phase 4: Verify improvements**
```promql
# Track error budget consumption rate
# Before: 15% of budget consumed/week
# Target: <5% of budget consumed/week

sum(increase(errors_total[7d])) 
/ sum(increase(requests_total[7d]))
/ 0.0005  # SLO target
* 100  # Percentage of weekly budget
```

**Phase 5: Prevent regression**
```yaml
# Add reliability tests to CI
- name: Chaos test
  run: |
    chaos-mesh apply network-delay.yaml
    # Verify service still meets SLOs
    
- name: Load test
  run: |
    k6 run --vus 100 load-test.js
    # Verify p99 latency < 500ms
```
</details>

---

## 🎯 Final Tips for DevOps Interviews

### Technical Interview Strategy

1. **Think out loud**: Explain your reasoning as you work through problems
2. **Start with questions**: Clarify requirements before jumping to solutions
3. **Trade-offs matter**: Discuss pros/cons of different approaches
4. **Real examples win**: Reference your actual experience when possible
5. **It's okay to say "I don't know"**: But explain how you'd find out

### Common Mistakes to Avoid

```
❌ Jumping straight to tools without understanding the problem
❌ Giving textbook answers without practical context
❌ Ignoring security, monitoring, or cost considerations
❌ Not asking clarifying questions
❌ Overcomplicating solutions
```

### Questions to Ask Your Interviewers

```
✓ What does your deployment pipeline look like?
✓ How do you handle incidents and on-call?
✓ What's your biggest technical challenge right now?
✓ How do you measure success for this role?
✓ What does the team structure look like?
```

---

## 📚 Additional Resources

- **[Kubernetes Interview Questions - Comprehensive Guide](https://github.com/JustInCache/kubernetes-interview-questions)** 🔗
- Google SRE Books (free online)
- AWS Well-Architected Framework
- The Phoenix Project / The DevOps Handbook

---

## Contributing

Found an error or want to add more questions? PRs welcome!

## ☕ Support the Project

If this repo saved you time or sparked an idea, consider buying me a coffee to keep the prompts flowing. ❤️ 

<a href="https://buymeacoffee.com/connectankush">
  <img src="img/bmc.png" alt="Buy Me a Coffee" width="150">
</a>

[https://buymeacoffee.com/connectankush](https://buymeacoffee.com/connectankush)

Your support and feedback are valuable in maintaining and improving the extension.

<a href="https://buymeacoffee.com/connectankush">
  <img src="img/bmc-qr-code.png" alt="Buy Me a Coffee QR" width="150">
</a>

## License

MIT License - Feel free to use for interview preparation

---

⭐ **Star this repo if it helped you prepare for your interview!**