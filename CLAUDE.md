# Claude Context - Hackers App Project

> **ALWAYS commit and push code changes immediately after making them. No exceptions.**
> **Do NOT start Playwright/browser automation unless explicitly asked.**
> **Do NOT close Playwright browsers unless explicitly asked.**

## Release Orchestration - How It Should Work

**Current state**: We built a per-component CI/CD shortcut where j-hackers-web's build workflow embeds deploy jobs. This only handles ONE service.

**Proper CloudBees Release Orchestration** uses j-hackers-app to coordinate ALL services:

```
┌─────────────────────────────────────────────────────────────────┐
│  CI PIPELINES (produce artifacts independently)                 │
│                                                                 │
│  j-hackers-web  → CloudBees build → registers artifact (UUID)  │
│  j-hackers-api  → Jenkins build   → registers artifact (UUID)  │
│  j-hackers-auth → GHA build       → registers artifact (UUID)  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                    Artifacts registered in CloudBees
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  RELEASE WORKFLOW (j-hackers-app/deployer.yaml)                 │
│                                                                 │
│  1. Picks up latest artifacts for ALL components                │
│  2. Creates a "manifest" with versions/UUIDs to deploy          │
│  3. Deploys ALL services to staging                             │
│  4. Approval gate                                               │
│  5. Deploys ALL services to production                          │
└─────────────────────────────────────────────────────────────────┘
```

### The Manifest Pattern
The `deployer.yaml` takes a manifest JSON specifying what to deploy:
```json
{
  "hackers-api": {"jamesalts/hackers-api": {"deploy": true, "version": "1.0-X", "id": "uuid"}},
  "hackers-auth": {"jamesalts/hackers-auth": {"deploy": true, "version": "1.0-X", "id": "uuid"}},
  "hackers-web": {"jamesalts/hackers-web": {"deploy": true, "version": "1.0-X", "id": "uuid"}}
}
```

### What's Next
- Set up j-hackers-app release workflow to work with our environments
- Ensure all 3 CI pipelines register artifacts properly
- Use release workflow to deploy all services together through staging → approval → prod

---

## True Enterprise Flow (Target State)

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
| Security scans | Partial | Full suite | Future |
| Quality gates (blocking) | ✗ | ✓ | Future |
| SBOM generation | ✗ | ✓ | Future |
| Artifact signing | ✗ | ✓ | Future |
| Multi-environment deploy | ✓ (k3s) | ✓ | ✅ Done |
| Approval gates | ✓ | ✓ | ✅ Done |
| ServiceNow integration | ✗ | ✓ | Phase 6 |
| Progressive rollout | ✓ (via FM) | ✓ | ✅ Done |
| Feature flags | ✓ | ✓ | ✅ Done |
| Observability | ✗ | ✓ | Future |

---

## Current State (Jan 26, 2026)

### Environments
| Environment | Namespace | URL | Purpose |
|------------|-----------|-----|---------|
| **Production** | `3demo` | http://hackers-web.54.201.69.176.nip.io | All 3 services |
| **Staging** | `3demo-staging` | http://staging-hackers-web.54.201.69.176.nip.io | Web only (currently) |

### j-hackers-web Current Flow (Simplified CI/CD)
```
Push to main → test → build → deploy-staging → APPROVAL → deploy (prod)
```
- Staging deploys automatically after build
- Manual approval gate required
- Production deploys only after approval
- Artifact UUID passed correctly for registration

### Build Systems
| Component | CI | Artifact Registration | Notes |
|-----------|----|-----------------------|-------|
| j-hackers-web | CloudBees Unify | ✅ Working | Kaniko extracts UUID |
| j-hackers-api | Jenkins | ⚠️ Check | May need UUID extraction |
| j-hackers-auth | GitHub Actions | ⚠️ Check | May need UUID extraction |

---

## Known Bugs

### GitHub App Integration (Jan 15, 2026)
GHA workflows with CloudBees actions failing with `404 Not Found : unable to retrieve run details`:
- **Cause**: Inherited integration from parent org doesn't work for GHA OIDC
- **Fix**: Reinstall GitHub App directly, recreate components
- **Integration**: `j-hackers` / Endpoint ID: `953edc92-9c1a-48c9-9329-412cf8083a5c`

### Workflow Dispatch UI Delay
Runs triggered via `workflow_dispatch` don't appear in Runs list immediately (search index delay). Runs work and can be accessed via direct URL.

### Jenkins CloudBees Integration Broken - CloudBees API 500 Errors (Jan 26-27, 2026)
**Status: ROOT CAUSE FOUND - Jenkins internal URL not updated**

When the EC2 instance restarted and got a new IP (54.201.69.176 instead of 54.189.62.135), the Jenkins CloudBees integration broke.

**Symptoms:**
- Jenkins shows error banner: "The instance is experiencing errors sending data to CloudBees Platform"
- Jenkins logs show `500` errors on `POST https://api.cloudbees.io/token-exchange/external-access-token-exchange/challenge`
- CloudBees integration shows "disconnected" status
- Builds complete but don't appear in CloudBees Unify Runs
- CloudBees logs show: "unable to find endpoint metadata for JENKINS"

**What We Tried:**
1. ✅ Updated k3s ingress to use new IP
2. ✅ Updated CloudBees integration URL to `http://jenkins.54.201.69.176.nip.io/`
3. ✅ Deleted and recreated CloudBees integration (multiple times)
4. ✅ Tried Automatic connection (recommended) - gets 500 errors from CloudBees API
5. ✅ Tried Manual connection - PEM download fails with "unexpected error"
6. ✅ Cleared stale Private key credential in Jenkins via Script Console
7. ✅ Created fresh integration `j-hackers-jenkins-2` - still blocked by API errors

**Root Cause (identified by Antonio Muniz Martin via Datadog logs):**
Jenkins internal URL setting was NOT updated. While we updated:
- k3s ingress ✅
- CloudBees integration URL ✅

We did NOT update Jenkins' own URL in **Manage Jenkins → System → Jenkins Location → Jenkins URL**.
Jenkins was still sending the OLD URL (`http://jenkins.54.189.62.135.nip.io/`) in requests to CloudBees, which didn't match the endpoint created with the new URL → "endpoint metadata not found" → 500.

**Note on API behavior:** The 500 error should arguably be a 4xx with a helpful message (per Ben Walding). The API returning 500 instead of a proper error made debugging harder.

### Fix Plan (Jan 27, 2026)

**Step 1: Update Jenkins Internal URL**
1. Navigate to: http://jenkins.54.201.69.176.nip.io/manage/configure
2. Find "Jenkins Location" section
3. Update "Jenkins URL" field:
   - FROM: `http://jenkins.54.189.62.135.nip.io/`
   - TO: `http://jenkins.54.201.69.176.nip.io/`
4. Click Save

**Step 2: Retry CloudBees Integration Connection**
1. Navigate to CloudBees integrations: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/configurations/integrations
2. Find `j-hackers-jenkins-2` integration
3. Click to edit
4. Click "Go to controller" to initiate handshake
5. In Jenkins, approve the connection if prompted

**Step 3: Verify**
1. CloudBees integration status shows "connected" (not "disconnected" or "pending")
2. Jenkins error banner disappears
3. Trigger a test build on j-hackers-api
4. Verify build appears in CloudBees Unify Runs within ~1 minute

**Key URLs:**
| Resource | URL |
|----------|-----|
| Jenkins System Config | http://jenkins.54.201.69.176.nip.io/manage/configure |
| CloudBees Integrations | https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/configurations/integrations |
| CloudBees Runs | https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/runs |

**Current Configuration:**
- Jenkins URL (should be): `http://jenkins.54.201.69.176.nip.io/`
- CloudBees Integration: `j-hackers-jenkins-2`
- Endpoint ID: `cb9e0f9e-393e-4bba-98b7-4284f8642e36`
- CloudBees Endpoint: `https://api.cloudbees.io/`
- Org ID: `eb3ae95d-a459-4f0a-ac58-57d752e4a373`

**Notes:**
- The 500 error should arguably be a 4xx with a helpful message (per Ben Walding)
- This is a common scenario when servers restart with new IPs - useful feedback for CloudBees engineering

---

## Bug Report for CloudBees Support

**Copy the section below to submit to CloudBees Support:**

---

### Bug Report: Jenkins Controller Integration - HTTP 500 Errors on Connection Endpoints

**Summary:**
Unable to connect Jenkins controller to CloudBees Platform. Both Automatic and Manual connection methods fail with HTTP 500 errors from CloudBees API endpoints.

**Environment:**
- CloudBees Organization: `j-hackers`
- Organization ID: `eb3ae95d-a459-4f0a-ac58-57d752e4a373`
- Integration Name: `j-hackers-jenkins-2`
- Jenkins URL: `http://jenkins.54.201.69.176.nip.io/`
- Jenkins Version: 2.528.3
- CloudBees Platform Integration Plugin: Installed and configured
- CloudBees API Endpoint: `https://api.cloudbees.io/`

**Date/Time of Issue:**
January 26, 2026, starting approximately 14:00 UTC

**Steps to Reproduce:**

1. Navigate to Configurations → Integrations in CloudBees Platform
2. Create new Jenkins Controller integration with:
   - Name: `j-hackers-jenkins-2`
   - Controller URL: `http://jenkins.54.201.69.176.nip.io`
3. Select "Automatic connection" (recommended) and submit
4. Click "Go to controller" button to initiate handshake
5. **Result:** CloudBees UI displays "An unexpected error occurred"

Alternative attempt with Manual connection:
1. Edit integration, select "Manual connection"
2. Click "Download the PEM file" button
3. **Result:** CloudBees UI displays "An unexpected error occurred"

**Error Details:**

Browser console shows:
```
Failed to load resource: the server responded with a status of 500
POST https://api.cloudbees.io/.../endpoints/192a579a-29ca-405e-ae7c-c1cc9e073420
```

Jenkins logs show:
```
500 Internal Server Error on POST https://api.cloudbees.io/token-exchange/external-access-token-exchange/challenge
```

**Expected Behavior:**
- Automatic connection: Handshake completes, integration status changes to "connected"
- Manual connection: PEM file downloads successfully for manual configuration

**Actual Behavior:**
- Both connection methods fail with HTTP 500 errors
- Integration remains in "disconnected" status
- Jenkins shows error banner: "The instance is experiencing errors sending data to CloudBees Platform"

**Troubleshooting Already Performed:**
1. Verified Jenkins URL is correct and accessible
2. Deleted and recreated integration multiple times
3. Cleared stale credentials from Jenkins configuration via Script Console
4. Tried both Automatic and Manual connection types
5. Verified CloudBees Platform Integration Plugin is installed on Jenkins

**Impact:**
- Jenkins builds complete but are not reported to CloudBees Platform
- Unable to use CloudBees Unify for Jenkins CI visibility
- Blocking release orchestration workflow implementation

**Additional Context:**
This issue started after our EC2 instance was restarted and received a new IP address. The Jenkins URL was updated accordingly, but the CloudBees API endpoints are returning 500 errors regardless of the connection method used.

**Request:**
Please investigate the HTTP 500 errors on the Jenkins integration endpoints. Is there a known outage or issue with the token-exchange and PEM generation APIs?

---

## Infrastructure

### AWS EC2 (k3s Server)
- **Public IP**: `54.201.69.176`
- **Instance ID**: `i-001c9ceb643b1d99a`
- **SSH**: `ssh -i ~/.ssh/3demo-key.pem ubuntu@54.201.69.176`

### Namespaces on k3s
```bash
# Check all environments
kubectl get pods -n 3demo          # Production (all 3 services)
kubectl get pods -n 3demo-staging  # Staging (web only)
```

### CloudBees Unify
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

---

## Artifact Registration

For release orchestration to work, artifacts must be registered with UUIDs.

### How It Works (j-hackers-web)
1. Kaniko builds image and outputs `artifact-ids` as JSON
2. Extract step parses UUID from JSON
3. Deploy workflow receives UUID for `register-deployed-artifact`

### For Manual Deploys
Use `artifact-id: manual` to skip registration when you don't have the real UUID.

---

## j-hackers-app Workflows

| Workflow | Purpose |
|----------|---------|
| `deployer.yaml` | Core dispatcher - calls each component's deploy.yaml based on manifest |
| `release.yaml` | BAU release: pre-prod → approval → prod |
| `full-release.yaml` | Enterprise 4-stage pipeline |

---

## GitHub Repos
- https://github.com/jalts-808/j-hackers-web
- https://github.com/jalts-808/j-hackers-api
- https://github.com/jalts-808/j-hackers-auth
- https://github.com/jalts-808/j-hackers-app

## Project Roadmap

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | AWS Infrastructure | ✅ Complete |
| 2 | Jenkins on k3s | ✅ Complete |
| 3 | CloudBees Unify builds | ✅ Complete |
| 4 | Deploy all 3 services | ✅ Complete |
| 5 | Feature Management | ✅ Complete |
| 6 | Release Orchestration | **IN PROGRESS** |

### Phase 6 Remaining
- [ ] Set up j-hackers-app release workflow for multi-component deploys
- [ ] Verify api/auth artifact registration
- [ ] Test full release: all 3 services → staging → approval → prod

---

## Notes
- James is learning - explain concepts clearly
- James prefers small, iterative commits
