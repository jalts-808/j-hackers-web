# Claude Context - Hackers App Project

> **ALWAYS commit and push code changes immediately after making them. No exceptions.**
> **Do NOT start Playwright/browser automation unless explicitly asked.**

## WHAT'S NEXT (Jan 15, 2026)

**Jenkins build #8 passed!** Now deploy j-hackers-api to k3s:

1. Go to: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/components/0c2fe8b7-cc02-48da-a571-b042ad85cbbb/workflows
2. Run the "deploy" workflow with:
   - `artifact-id`: `manual`
   - `artifactVersion`: `3.0-8`
   - `environment`: `j-hackers-k3s`
3. Once deployed, test: http://hackers-api.54.189.62.135.nip.io
4. Verify web app works: http://hackers-web.54.189.62.135.nip.io (should show stories instead of "API Currently Unavailable")

After API is deployed, deploy auth service (j-hackers-auth) the same way, then move to Phase 5 (Feature Management).

## Project Owner
James Altheide (jalts-808 on GitHub) - Growth Product Manager at CloudBees

## What This Project Is
Learning project to understand CloudBees Unify by building and deploying a 3-component application.

## End Goal
Run a Feature Management example against hackers-web, with everything deployed to a personal k3s instance on AWS.

## Current State (Last Updated: Jan 14, 2026)

### ACTIVE WORK: Jenkins → Unify Integration

#### ⚠️ CORRECTED INFO (Jan 14, 2026)
Previous notes had incorrect plugin names. Here's what the docs actually say:

#### Official 5-Step Integration Process (from CloudBees docs)
| Step | Location | Action | Status |
|------|----------|--------|--------|
| 1 | Jenkins | Create **Multibranch Pipeline** (NOT regular Pipeline!) with GitHub branch source | ❌ TODO |
| 2 | Jenkins | Install `CloudBees Platform Integration plugin::Controllers` | ⚠️ Verify |
| 3 | Unify | Create CI controller integration | ✅ Done (`3demo-k3s-jenkins`) |
| 4 | Unify | Create SCM integration for the repo | ⚠️ Verify |
| 5 | Unify | Create component linked to integrated source | ✅ Done (`j-hackers-api`) |

#### Critical Requirements (from docs)
- **Job Type**: Must be **Multibranch Pipeline** or **Organization Folder** - other job types NOT supported!
- **Plugin**: `CloudBees Platform Integration plugin::Controllers` (ID: `cloudbees-cbp-unify-integration-plugin`)
- **Jenkins Version**: 2.504.2 or later required
- **SCM**: GitHub, Bitbucket, or GitLab branch source only
- **Important**: Only builds executed AFTER integration will appear in Unify

#### What Was Previously Done (may be incorrect)
1. ✅ Created Jenkins controller integration in Unify named `3demo-k3s-jenkins`
2. ⚠️ Installed `CloudBees Installation` plugin - **WRONG PLUGIN?** Should be `CloudBees Platform Integration plugin::Controllers`
3. ⚠️ Installed `CloudBees Platform Insights Plugin` - **Different feature** (CI Insights, not integration)
4. ⚠️ Configured plugin with auth code - this may be for CI Insights, not the integration plugin
5. ✅ Controller appears in Unify → Jenkins Management

#### The Real Problem
Jenkins has NO JOBS, but more importantly:
- We need a **Multibranch Pipeline** (not a regular Pipeline job)
- The correct plugin may not be installed

#### Next Steps (CORRECTED)
1. **Check Jenkins plugins** - verify `CloudBees Platform Integration plugin::Controllers` is installed
2. **Create Multibranch Pipeline job** in Jenkins:
   - New Item → Multibranch Pipeline
   - Add GitHub branch source pointing to `jalts-808/j-hackers-api`
   - It will auto-discover branches with Jenkinsfile
3. **Verify SCM integration** exists in Unify for `jalts-808/j-hackers-api`
4. **Trigger a build** and verify it appears in Unify → Runs

#### Docs Reference
- https://docs.cloudbees.com/docs/cloudbees-platform/latest/continuous-integration/ci-getting-started

**Unify URLs:**
- Jenkins Management: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/jenkins-management/jenkins-controllers
- Runs: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/runs

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
- [x] Added secrets: `kubeconfig` (BASE64 ENCODED!), `DOCKERHUB_USER` (jamesalts)
- [x] Added variable: `namespace` (3demo)
- [x] Updated all deploy.yaml workflows (web, api, auth) to use:
  - Repos: `jalts-808/j-hackers-*` instead of `cloudbees-days/hackers-*`
  - Hostnames: `*.54.189.62.135.nip.io`
  - Ingress: `traefik` instead of `nginx`
- [x] Updated `deployer.yaml` in j-hackers-app to reference correct repos/images
- [x] Committed and pushed all changes to all 4 repos
- [x] Fixed kubeconfig encoding (must be BASE64 - `cloudbees-days/setup-kubeconfig` decodes it)
- [x] Replaced ARM64 instance with x86_64 (Docker images built for x86_64)
- [x] **j-hackers-web deployed with auto-deploy and artifact tracking** (see below)

**REMAINING:**
- [ ] Deploy j-hackers-api and j-hackers-auth (manual deploy with `artifact-id: manual`)
- [ ] Verify all 3 services running: `KUBECONFIG=~/.kube/config-3demo kubectl get pods -n 3demo`
- [ ] Test endpoints:
  - http://hackers-web.54.189.62.135.nip.io ✅ (deployed)
  - http://hackers-api.54.189.62.135.nip.io
  - http://hackers-auth.54.189.62.135.nip.io

### How to Trigger Deploy (Manual - for api/auth)
1. Go to component → Workflows tab → click "deploy" → Run
2. Fill in: `artifact-id: manual`, `artifactVersion: <version>`, `environment: j-hackers-k3s`
   - j-hackers-api: version `3.0-3`
   - j-hackers-auth: version `2.0-4`

Note: Using `artifact-id: manual` skips artifact registration (step is conditional). This is fine for manual deploys but means no artifact tracking in Unify.

### How to Update Kubeconfig in Unify
The kubeconfig MUST be BASE64 encoded (the `cloudbees-days/setup-kubeconfig` action decodes it).
```bash
cat ~/.kube/config-3demo | base64 | pbcopy
```
Then paste into: Configurations → Environments → j-hackers-k3s → Edit → kubeconfig secret

---

## Artifact Registration & Build→Deploy Flow (IMPORTANT)

### Why Artifact Tracking Matters
For **Release Orchestration** and **Feature Management** to work properly, CloudBees Unify needs to track:
1. Which artifact (by UUID) was built
2. Which environment it was deployed to
3. When and by whom

Without proper artifact tracking, you can't use:
- Environment Inventory (see what's deployed where)
- Release pipelines with artifact promotion
- Deployment history and audit trails

### How Artifact Registration Works

The `cloudbees-io/register-deployed-artifact@v2` action requires a **valid UUID** for the `artifact-id` parameter - NOT a placeholder like "test-deploy" or "manual".

**Where does the UUID come from?**
The `cloudbees-io/kaniko@v1` action (used in build) registers the artifact and outputs its UUID.

### The Problem We Solved

**Kaniko outputs `artifact-ids` as JSON:**
```json
{"docker.io/jamesalts/hackers-web:1.0-0.0.31": "45d69219-332b-4a50-930f-cca5f8caa59d"}
```

The deploy workflow needs just the UUID: `45d69219-332b-4a50-930f-cca5f8caa59d`

### The Solution: Auto-Deploy with Artifact ID Extraction

We modified `j-hackers-web/build.yaml` to:

1. **Add `id: kaniko` to the Kaniko step** (to access outputs)
2. **Add an extraction step** that parses the UUID from the JSON:
```yaml
- name: Extract artifact ID
  id: extract-artifact-id
  uses: docker://alpine:latest
  run: |
    ARTIFACT_IDS='${{ steps.kaniko.outputs.artifact-ids }}'
    # Extract UUID using regex pattern
    ARTIFACT_ID=$(echo "$ARTIFACT_IDS" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}')
    echo -n "$ARTIFACT_ID" >> $CLOUDBEES_OUTPUTS/artifact-id
```

3. **Add job outputs:**
```yaml
build:
  outputs:
    artifact-id: ${{ steps.extract-artifact-id.outputs.artifact-id }}
    artifact-version: 1.0-${{ cloudbees.version }}
```

4. **Add a deploy job that runs after build:**
```yaml
deploy:
  needs: build
  if: ${{ cloudbees.scm.branch == 'main' }}
  uses: jalts-808/j-hackers-web/.cloudbees/workflows/deploy.yaml
  with:
    artifact-id: ${{ needs.build.outputs.artifact-id }}
    artifactVersion: ${{ needs.build.outputs.artifact-version }}
    environment: j-hackers-k3s
  secrets: inherit
  vars: inherit
```

### The Result

Now when you push to main:
```
test → build → deploy (automatic)
         ↓
   Kaniko builds image
   Registers artifact (gets UUID)
   Extract step parses UUID
   Deploy receives real UUID
   Artifact registration succeeds!
```

**Unify now tracks:**
- Artifact: `docker.io/jamesalts/hackers-web:1.0-0.0.31`
- Environment: `j-hackers-k3s`
- Deployed by: build workflow
- Full audit trail

### Why Auto-Deploy Makes Sense for Feature Management

**Deploy ≠ Release** with feature flags:
1. Push code with new feature behind a flag (flag OFF by default)
2. Auto-deploy to k3s - code is deployed but feature is hidden
3. Toggle flag in Unify UI - feature goes live instantly
4. Problem? Toggle OFF immediately - instant rollback

This enables rapid iteration without risky deployments.

### Manual Deploy Option (for api/auth)

The deploy.yaml workflows have a conditional that skips artifact registration for manual deploys:
```yaml
- name: Register deployed artifact
  if: ${{ inputs.artifact-id != 'test-deploy' && inputs.artifact-id != 'manual' && inputs.artifact-id != '' }}
  uses: cloudbees-io/register-deployed-artifact@v2
```

Use `artifact-id: manual` for manual deploys when you don't have the real UUID.

---

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

### EC2 Instance (Current - x86_64)
- **Instance ID**: `i-001c9ceb643b1d99a`
- **Name**: `3demo-k3s-x86`
- **Public IP**: `54.189.62.135`
- **Instance Type**: `t3.medium` (x86_64)
- **Region**: `us-west-2`

### Old EC2 Instance (Stopped - ARM64, DO NOT USE)
- **Instance ID**: `i-0cb966d64870f9575`
- **Name**: `3demo-k3s`
- **Instance Type**: `t4g.medium` (ARM64) - incompatible with x86_64 Docker images

### SSH Access
```bash
ssh -i ~/.ssh/3demo-key.pem ubuntu@54.189.62.135
```

### k3s
- **Kubeconfig**: `~/.kube/config-3demo`
- **Namespace**: `3demo`

```bash
KUBECONFIG=~/.kube/config-3demo kubectl get nodes
```

### Jenkins on k3s
- **URL**: http://jenkins.54.189.62.135.nip.io (needs reinstall on new instance)
- **Username**: `admin`
- **Password**: (stored in k3s secret)

### AWS Commands
```bash
# Check instance status
aws ec2 describe-instances --instance-ids i-001c9ceb643b1d99a --query 'Reservations[0].Instances[0].[State.Name,PublicIpAddress]' --output text

# Start/Stop instance
aws ec2 start-instances --instance-ids i-001c9ceb643b1d99a
aws ec2 stop-instances --instance-ids i-001c9ceb643b1d99a
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
