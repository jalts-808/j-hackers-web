# Claude Context - Hackers App Project

> **ALWAYS commit and push code changes immediately after making them. No exceptions.**

## Project Owner
James Altheide (jalts-808 on GitHub) - Growth Product Manager at CloudBees

## What This Project Is
Learning project to understand CloudBees Unify by building and deploying a 3-component application.

## End Goal
Run a Feature Management example against hackers-web, with everything deployed to a personal k3s instance on AWS.

## Current State (Last Updated: Jan 14, 2026)

### Build Systems - All Working
| Service | CI/CD | Image | Status |
|---------|-------|-------|--------|
| j-hackers-api | Jenkins | `jamesalts/hackers-api` | Builds working |
| j-hackers-auth | GitHub Actions | `jamesalts/hackers-auth` | Builds working |
| j-hackers-web | CloudBees Unify | `jamesalts/hackers-web` | Builds working |

All three services are building and pushing Docker images to DockerHub.

## Project Roadmap

| Phase | Description | Status |
|-------|-------------|--------|
| **1** | AWS Infrastructure - EC2 + k3s setup | **Complete** |
| **2** | Jenkins on k3s | **Complete** |
| **3** | CloudBees Unify builds | **Complete** |
| **4** | Deploy all 3 services to k3s | **IN PROGRESS** |
| **5** | Feature Management setup | Pending |
| **6** | Release orchestration *(optional)* | Pending |

### Phase 4: Deploy to k3s (IN PROGRESS)

**COMPLETED:**
- [x] Created `j-hackers-k3s` environment in Unify (Configurations → Environments)
- [x] Added secrets: `kubeconfig`, `DOCKERHUB_USER` (jamesalts)
- [x] Added variable: `namespace` (3demo)
- [x] Updated all deploy.yaml workflows (web, api, auth) to use:
  - Repos: `jalts-808/j-hackers-*` instead of `cloudbees-days/hackers-*`
  - Hostnames: `*.54.214.225.132.nip.io` instead of `*.preview.cb-demos.io`
  - Ingress: `traefik` instead of `nginx`
- [x] Updated `deployer.yaml` in j-hackers-app to reference correct repos/images
- [x] Committed and pushed all changes to all 4 repos

**BLOCKED - FIX NEEDED:**
- [ ] First deploy of j-hackers-web FAILED with kubeconfig error:
  ```
  error loading config file: yaml: invalid leading UTF-8 octet
  ```
  **Fix:** Re-enter kubeconfig in environment settings (copy/paste corrupted it)
  - Go to: Configurations → Environments → search "j-hackers-k3s" → Edit
  - Clear and re-paste kubeconfig from `~/.kube/config-3demo`

**REMAINING:**
- [ ] Fix kubeconfig and retry j-hackers-web deploy
- [ ] Deploy j-hackers-api and j-hackers-auth
- [ ] Verify all 3 services running: `KUBECONFIG=~/.kube/config-3demo kubectl get pods -n 3demo`
- [ ] Test endpoints:
  - http://hackers-web.54.214.225.132.nip.io
  - http://hackers-api.54.214.225.132.nip.io
  - http://hackers-auth.54.214.225.132.nip.io

### How to Trigger Deploy (Manual)
1. Go to component → Workflows tab → click "deploy" → Run
2. Fill in: `artifact-id: test-deploy`, `artifactVersion: latest`, `environment: j-hackers-k3s`

### Phase 5: Feature Management
- [ ] Create FM application in Unify
- [ ] Create feature flags
- [ ] Integrate SDK into hackers-web
- [ ] Test flag toggling

## Repository Structure
```
3demo/
├── j-hackers-web/   # Vue.js frontend (CloudBees Unify)
├── j-hackers-api/   # Go backend API (Jenkins)
├── j-hackers-auth/  # Go auth service (GitHub Actions)
└── j-hackers-app/   # Release orchestration workflows
```

## Architecture
```
┌──────────────────────────────────────────────────────────────┐
│  AWS EC2 + k3s                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Traefik Ingress (built into k3s)                       │ │
│  └────────┬──────────────────┬──────────────────┬──────────┘ │
│           ▼                  ▼                  ▼            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│  │ hackers-web │    │ hackers-api │    │ hackers-auth│       │
│  │   (Vue.js)  │───▶│    (Go)     │    │    (Go)     │       │
│  └─────────────┘    └─────────────┘    └─────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

## AWS Infrastructure

### EC2 Instance
- **Instance ID**: `i-0cb966d64870f9575`
- **Name**: `3demo-k3s`
- **Public IP**: `54.214.225.132` (changes when stopped/started)
- **Instance Type**: `t4g.medium` (ARM64)
- **Region**: `us-west-2`

### SSH Access
```bash
ssh -i ~/.ssh/3demo-key.pem ubuntu@54.214.225.132
```

### k3s
- **Kubeconfig**: `~/.kube/config-3demo`
- **Namespace**: `3demo`

```bash
KUBECONFIG=~/.kube/config-3demo kubectl get nodes
```

### Jenkins on k3s
- **URL**: http://jenkins.54.214.225.132.nip.io
- **Username**: `admin`
- **Password**: (stored in k3s secret)

### AWS Commands
```bash
# Check instance status
aws ec2 describe-instances --instance-ids i-0cb966d64870f9575 --query 'Reservations[0].Instances[0].[State.Name,PublicIpAddress]' --output text

# Start/Stop instance
aws ec2 start-instances --instance-ids i-0cb966d64870f9575
aws ec2 stop-instances --instance-ids i-0cb966d64870f9575
```

## GitHub Repos
- https://github.com/jalts-808/j-hackers-web
- https://github.com/jalts-808/j-hackers-api
- https://github.com/jalts-808/j-hackers-auth
- https://github.com/jalts-808/j-hackers-app

## CloudBees Unify
- **Organization**: j-hackers
- **Org ID**: `eb3ae95d-a459-4f0a-ac58-57d752e4a373`
- **Components URL**: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/components

### Component IDs
| Component | ID |
|-----------|-----|
| j-hackers-web | `25798b54-86df-4e9d-a89c-75fe3faebe25` |
| j-hackers-api | `0c2fe8b7-cc02-48da-a571-b042ad85cbbb` |
| j-hackers-auth | `9a1504a9-ccde-48df-bad0-0a21e27161a1` |
| j-hackers-app | `f5cceab1-2bd2-4048-88a4-08ff753688a0` |

## Phase 6 Notes: Release Orchestration (j-hackers-app)

The `j-hackers-app` repo is a **Release Orchestration Component** - no application code, only CloudBees Unify workflows that coordinate deployments across all three services.

### Key Workflows
| Workflow | Purpose |
|----------|---------|
| `deployer.yaml` | Core dispatcher - calls each component's deploy.yaml based on manifest |
| `release.yaml` | Simple BAU release: pre-prod → approval → prod |
| `full-release.yaml` | Enterprise 4-stage pipeline (QA → Staging → Prod) |
| `snow.yaml` | Same as full-release with ServiceNow integration |

### How It Works
Uses a **manifest JSON** to specify which components/versions to deploy:
```json
{
  "hackers-web": { "deploy": true, "version": "1.0-0.0.69" },
  "hackers-api": { "deploy": true, "version": "..." },
  "hackers-auth": { "deploy": false }
}
```

### Full Release Pipeline Stages
1. **Release Readiness** - Jira issue, quality checks, approval gate
2. **QA** - Deploy, enable feature flags, run tests, exit criteria
3. **Staging** - Release manager approval, E2E tests, risk assessment, ServiceNow CR
4. **Production** - Deploy, smoke tests, progressive flag rollout (10% → 50% → 100%)

### To Customize for This Project
The workflows reference CloudBees demo environment. To use:
- Change `cloudbees-days/hackers-*` → `jalts-808/j-hackers-*`
- Change `ldonleycb/*` Docker images → `jamesalts/*`
- Update environment names to match k3s setup

## Pipeline Comparison: Current vs Enterprise

### Current Flow (This Project)
```
commit
   ↓
build → save artifact
   │   ✓ j-hackers-api: Jenkins (Buildah)
   │   ✓ j-hackers-auth: GitHub Actions (docker/build-push-action)
   │   ✓ j-hackers-web: CloudBees Unify (Kaniko + Skopeo)
   ↓
security scan
   │   ✓ j-hackers-api: Trivy + Grype (Jenkins)
   │   ⚠ j-hackers-web: Unify implicit scans
   │   ✗ j-hackers-auth: None
   ↓
push to DockerHub
   │   ✓ All 3 services
   ↓
deploy to k3s                           ✗ Phase 4 (TODO)
   ↓
feature flags                           ✗ Phase 5 (TODO)
   ↓
release orchestration                   ✗ Phase 6 (TODO)
```

### True Enterprise Flow (Target State)
```
commit
   ↓
build → save artifact
   │   • Artifact stored in registry (Unify, Artifactory, etc.)
   │   • SBOM generated (Software Bill of Materials)
   │   • Artifact signed (Sigstore, Cosign)
   ↓
security scan (SAST, SCA, container scan)
   │   • SAST: Static code analysis (SonarQube, Checkmarx)
   │   • SCA: Dependency vulnerabilities (Snyk, Grype, Trivy)
   │   • Container: Image vulnerabilities (Trivy, Grype)
   │   • Secrets: Leaked credentials (GitLeaks, TruffleHog)
   ↓
quality gate (BLOCKS if failed)
   │   • Tests pass? Coverage threshold met?
   │   • No critical/high CVEs?
   │   • Code quality score acceptable?
   ↓
push to DEV registry
   │   • Separate registry per environment
   │   • Or same registry with environment tags
   ↓
deploy to DEV → run smoke tests
   │   • Automated deployment
   │   • Basic health checks
   ↓
approval gate (QA sign-off)
   │   • Manual approval required
   │   • QA team verifies functionality
   ↓
deploy to STAGING → run E2E tests
   │   • Full integration tests
   │   • Performance tests
   │   • Chaos engineering (optional)
   ↓
approval gate (Release Manager)
   │   • Business sign-off
   │   • Release notes reviewed
   ↓
change request (ServiceNow/Jira)
   │   • Audit trail created
   │   • Change window scheduled
   ↓
approval gate (CAB - Change Advisory Board)
   │   • Final approval for production
   │   • Risk assessment complete
   ↓
deploy to PROD (maintenance window)
   │   • Blue/green or canary deployment
   │   • Automated rollback ready
   ↓
smoke tests → progressive rollout
   │   • 10% traffic → monitor → 50% → monitor → 100%
   │   • Automatic rollback on error spike
   ↓
feature flags enabled gradually
   │   • Features enabled per user segment
   │   • A/B testing capabilities
   │   • Kill switch available
   ↓
observability & feedback
   │   • Metrics, logs, traces (Prometheus, Grafana, Jaeger)
   │   • Error tracking (Sentry)
   │   • User feedback loops
```

### Gap Analysis: What's Missing
| Capability | Current | Enterprise | Phase to Add |
|------------|---------|------------|--------------|
| Build artifacts | ✓ | ✓ | - |
| Security scans | Partial | Full suite | Phase 4+ |
| Quality gates (blocking) | ✗ | ✓ | Phase 4+ |
| SBOM generation | ✗ | ✓ | Future |
| Artifact signing | ✗ | ✓ | Future |
| Multi-environment deploy | ✗ | ✓ | Phase 4 |
| Approval gates | ✗ | ✓ | Phase 6 |
| ServiceNow integration | ✗ | ✓ | Phase 6 |
| Progressive rollout | ✗ | ✓ | Phase 5/6 |
| Feature flags | ✗ | ✓ | Phase 5 |
| Observability | ✗ | ✓ | Future |

## Important Notes
- Do NOT do anything unless James explicitly asks
- James is learning - explain concepts clearly
- James prefers small, iterative commits
