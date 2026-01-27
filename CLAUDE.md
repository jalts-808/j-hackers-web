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

## Enterprise Pipeline Patterns: Team Autonomy vs Coordinated Releases

When you have multiple teams (each owning their own service with their own CI system), there are two fundamental approaches to deploying to staging/production.

### Pattern A: Continuous Deployment (Team Autonomy)
```
Team A pushes → CI builds → Automatically deploys to Staging
Team B pushes → CI builds → Automatically deploys to Staging
Team C pushes → CI builds → Automatically deploys to Staging
```
Each team owns their service end-to-end. They can deploy to staging whenever they want.

**Pros:** Fast feedback, team autonomy, less coordination overhead
**Cons:** Potential "integration hell" - Team A deploys, breaks Team B's integration

### Pattern B: Coordinated Releases (Release Train)
```
Team A pushes → CI builds → Artifact sits in registry (holding zone)
Team B pushes → CI builds → Artifact sits in registry (holding zone)
Team C pushes → CI builds → Artifact sits in registry (holding zone)
                              ↓
              Release Orchestration picks up all 3
                              ↓
                    Deploys ALL to Staging together
```
A release manager or automated process deploys services together.

**Pros:** All services deploy together (same versions), clear release cadence, better for compliance
**Cons:** Slower iteration, teams wait for each other

### What We Have (Hybrid)

Our setup supports **both patterns**:

| Pattern | How to Use | When to Use |
|---------|-----------|-------------|
| **Team Autonomy** | Each service's `build.yaml` auto-deploys to staging | Day-to-day development, fast iteration |
| **Coordinated Release** | `simplified-release.yaml` deploys all 3 together | Production releases, coordinated changes |

### The "Holding Zone" Concept

The artifact registry (Docker Hub) **is** the holding zone:
```
┌─────────────────────────────────────────────────────────────┐
│  CI PIPELINES (Build + Register Artifact)                   │
│                                                             │
│  Team A → Jenkins → Builds hackers-api:3.0-19              │
│  Team B → GHA → Builds hackers-auth:2.0-19                 │
│  Team C → CloudBees → Builds hackers-web:1.0-42            │
│                                                             │
│  Artifacts exist in Docker Hub / Artifact Registry         │
│  They're "built" but not "deployed" anywhere yet           │
└─────────────────────────────────────────────────────────────┘
                              ↓
                    This IS the holding zone
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  RELEASE ORCHESTRATION (simplified-release.yaml)            │
│                                                             │
│  DevOps/Release Manager triggers with manifest:             │
│  "Deploy api:3.0-19, auth:2.0-19, web:1.0-42 to staging"   │
└─────────────────────────────────────────────────────────────┘
```

### Typical Enterprise Environment Ownership

| Environment | Who Controls Deploy? | Why? |
|-------------|---------------------|------|
| **Dev/Feature** | Individual teams, automatic | Fast feedback, experimentation |
| **Staging** | Often automatic, sometimes gated | Integration validation |
| **Production** | Release team, manual approval | Risk management, compliance |

### Mitigating "Integration Hell"

When teams deploy independently, one team's change can break another's integration:
```
Monday 9am:  Team A deploys v2.0 of API to staging (works fine)
Monday 10am: Team B deploys v1.5 of Auth to staging (breaks API v2.0!)
Monday 11am: Team A: "WTF, staging is broken, I didn't change anything!"
```

**Enterprise mitigations:**
1. **Contract Testing** - Services define API contracts, CI fails if you break them (Pact)
2. **Feature Flags** - New functionality is "dark" until all services are ready
3. **API Versioning** - v1 endpoints stay working while v2 rolls out
4. **Coordinated Release Windows** - "Staging deploys happen at 2pm daily"
5. **Automated Integration Tests** - Run after every staging deploy, alert if broken

### Recommended Approach for This Project

**For day-to-day development:** Let teams deploy to staging independently via their own CI.
```
Push to main → Build → Auto-deploy to staging
```
Fast feedback. If something breaks, the team that broke it fixes it.

**For production releases:** Use coordinated releases via `simplified-release.yaml`.
```
Pick all 3 artifact versions → Deploy to staging together → Approve → Production
```
This ensures you know exactly what versions are in production together.

---

## Current State (Jan 27, 2026)

### Environments
| Environment | Namespace | URL | Purpose | CloudBees Env Name |
|------------|-----------|-----|---------|-------------------|
| **Production** | `3demo` | http://hackers-web.54.201.69.176.nip.io | All 3 services | `hackers-prod` |
| **Staging** | `3demo-staging` | http://staging-hackers-web.54.201.69.176.nip.io | All 3 services | `hackers-staging` |

### CloudBees Environment Configuration
Both environments require these variables/secrets:

| Variable/Secret | Description | Example |
|----------------|-------------|---------|
| `namespace` | k8s namespace | `3demo` or `3demo-staging` |
| `host_prefix` | URL prefix | `staging-` or empty |
| `ORG_ID` | CloudBees org ID | `eb3ae95d-a459-4f0a-ac58-57d752e4a373` |
| `FM_TOKEN` (secret) | Feature Management token | - |
| `kubeconfig` (secret) | k3s cluster access | - |
| `DOCKERHUB_USER` (secret) | Registry username | - |
| `DOCKERHUB_TOKEN` (secret) | Registry password | - |

### j-hackers-web Current Flow (Simplified CI/CD)
```
Push to main → test → build → deploy-staging → APPROVAL → deploy (prod)
```
- Staging deploys automatically after build
- Manual approval gate required
- Production deploys only after approval
- Artifact UUID passed correctly for registration

### Build Systems
| Component | CI | Artifact Registration | Artifact UUID Example |
|-----------|----|-----------------------|----------------------|
| j-hackers-web | CloudBees Unify | ✅ Working | Kaniko extracts UUID automatically |
| j-hackers-api | Jenkins | ✅ Working | `35818d5f-0cd7-485c-9d8d-b1d9751dd592` |
| j-hackers-auth | GitHub Actions | ✅ Working | Extracted from build output |

---

## Known Bugs

### GitHub App Integration (Jan 15, 2026)
GHA workflows with CloudBees actions failing with `404 Not Found : unable to retrieve run details`:
- **Cause**: Inherited integration from parent org doesn't work for GHA OIDC
- **Fix**: Reinstall GitHub App directly, recreate components
- **Integration**: `j-hackers` / Endpoint ID: `953edc92-9c1a-48c9-9329-412cf8083a5c`

### Workflow Dispatch UI Delay
Runs triggered via `workflow_dispatch` don't appear in Runs list immediately (search index delay). Runs work and can be accessed via direct URL.

### Jenkins CloudBees Integration (Jan 26-27, 2026) - ✅ RESOLVED
**Status: FIXED on Jan 27, 2026**

When the EC2 instance restarted and got a new IP (54.201.69.176 instead of 54.189.62.135), the Jenkins CloudBees integration broke.

**Root Cause (identified by Antonio Muniz Martin via Datadog logs):**
Jenkins internal URL setting was NOT updated. Jenkins was sending the OLD URL in requests to CloudBees, causing "endpoint metadata not found" → 500 errors.

**Resolution:**
1. Verified Jenkins URL was already correct at `http://jenkins.54.201.69.176.nip.io/`
2. Deleted old broken integration `j-hackers-jenkins-2`
3. Created fresh integration `j-hackers-jenkins-v3` using Automatic connection
4. Confirmed connection in Jenkins when prompted
5. Verified build #16 on j-hackers-api appears in CloudBees Runs

**Current Working Configuration:**
- Jenkins URL: `http://jenkins.54.201.69.176.nip.io/`
- CloudBees Integration: `j-hackers-jenkins-v3` (status: **connected**)
- Org ID: `eb3ae95d-a459-4f0a-ac58-57d752e4a373`

**Lesson Learned:**
When EC2 IP changes, update ALL of these:
1. k3s ingress
2. CloudBees integration URL
3. Jenkins internal URL (Manage Jenkins → System → Jenkins Location)
4. If integration is corrupted, delete and recreate fresh

**Note:** The CloudBees API returning 500 instead of a helpful 4xx error made debugging harder (feedback sent to CloudBees engineering).

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
| `simplified-release.yaml` | **Recommended** - Staging → Approval → Production (skips QA/ServiceNow) |
| `release.yaml` | BAU release: pre-prod → approval → prod |
| `full-release.yaml` | Enterprise 4-stage pipeline (includes QA and ServiceNow placeholders) |

### Running a Simplified Release
Trigger `simplified-release.yaml` with this manifest format:
```json
{
  "hackers-api": {"jamesalts/hackers-api": {"deploy": true, "version": "3.0-19", "id": "35818d5f-0cd7-485c-9d8d-b1d9751dd592"}},
  "hackers-auth": {"jamesalts/hackers-auth": {"deploy": true, "version": "2.0-19", "id": "manual"}},
  "hackers-web": {"jamesalts/hackers-web": {"deploy": true, "version": "1.0-42", "id": "manual"}}
}
```

Use `"id": "manual"` if you don't have the artifact UUID - this skips artifact registration tracking.

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

### Phase 6 Checklist
- [x] Set up j-hackers-app release workflow for multi-component deploys (`simplified-release.yaml`)
- [x] Verify api artifact registration (Jenkins integration working, UUID: `35818d5f-0cd7-485c-9d8d-b1d9751dd592`)
- [x] Verify auth artifact registration (working)
- [x] Configure CloudBees environments (`hackers-staging`, `hackers-prod`)
- [ ] Test full release: all 3 services → staging → approval → prod
- [ ] Verify staging URLs accessible after deployment
- [ ] Verify production URLs accessible after deployment

---

## Service URLs

### Staging Environment
| Service | URL |
|---------|-----|
| Web | http://staging-hackers-web.54.201.69.176.nip.io |
| API | http://staging-hackers-api.54.201.69.176.nip.io |
| Auth | http://staging-hackers-auth.54.201.69.176.nip.io |

### Production Environment
| Service | URL |
|---------|-----|
| Web | http://hackers-web.54.201.69.176.nip.io |
| API | http://hackers-api.54.201.69.176.nip.io |
| Auth | http://hackers-auth.54.201.69.176.nip.io |

---

## Notes
- James is learning - explain concepts clearly
- James prefers small, iterative commits
