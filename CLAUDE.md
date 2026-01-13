# Claude Context - Hackers App Project

## Project Owner
James Altheide (jalts-808 on GitHub) - Growth Product Manager at CloudBees

## What This Project Is
Learning project to understand CloudBees Unify by building and deploying a 3-component application. James learns by doing - he needs to create things himself to understand them.

## End Goal
Run a Feature Management example against the j-hackers-web service, with everything deployed to a personal k3s instance on AWS.

## Current State (Last Updated: Jan 13, 2025)

### Completed
- Forked all 4 repos from cloudbees-days to jalts-808
- Renamed repos with `j-` prefix (j-hackers-web, j-hackers-api, j-hackers-auth, j-hackers-app)
- Cloned all repos locally to `/Users/jalts/Desktop/3demo/`
- Created README.md and CLAUDE.md in j-hackers-web
- Components created in CloudBees Unify
- Pushed test commits to all 4 repos (fork attribution)
- Created ROADMAP.md with full project plan

### In Progress
- Phase 1: AWS Infrastructure setup

## Project Roadmap

| Phase | Description | Status |
|-------|-------------|--------|
| **1** | AWS Infrastructure - EC2 + k3s setup | Pending |
| **2** | Jenkins on k3s *(optional)* | Pending |
| **3** | CloudBees Unify config - secrets, variables | Pending |
| **4** | Build & Deploy all 3 services to k3s | Pending |
| **5** | Feature Management setup and integration | Pending |
| **6** | Release orchestration *(optional)* | Pending |

### Phase 1: AWS Infrastructure
- [ ] Launch EC2 instance (t3.medium or t3.large)
- [ ] Install k3s: `curl -sfL https://get.k3s.io | sh -`
- [ ] Configure security groups (22, 80, 443, 6443)
- [ ] Set up DNS (own domain or nip.io)

### Phase 2: Jenkins on k3s (Optional)
- [ ] Install Jenkins via Helm
- [ ] Configure Kubernetes cloud for pod agents
- [ ] Test j-hackers-api Jenkinsfile build

### Phase 3: CloudBees Unify Configuration
- [ ] Configure secrets: `DOCKERHUB_USER`, `DOCKERHUB_TOKEN`, `PAT`, `kubeconfig`
- [ ] Configure variables: `namespace`
- [ ] Create k8s namespace: `kubectl create namespace 3demo`

### Phase 4: Build & Deploy
- [ ] Trigger builds for all 3 services
- [ ] Deploy to k3s via Helm
- [ ] Configure Ingress routing
- [ ] Verify application works end-to-end

### Phase 5: Feature Management
- [ ] Create FM application in Unify
- [ ] Create feature flags
- [ ] Integrate SDK into j-hackers-web
- [ ] Test flag toggling

### Phase 6: Release Orchestration (Optional)
- [ ] Configure j-hackers-app release workflow
- [ ] Test coordinated multi-service deployment

## Repository Structure
```
3demo/
├── j-hackers-web/   # Vue.js frontend - THIS IS THE MAIN REPO
├── j-hackers-api/   # Go backend API (has Jenkinsfile)
├── j-hackers-auth/  # Go auth service
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
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Jenkins (optional) + ephemeral build pods              │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

## GitHub Repos
- https://github.com/jalts-808/j-hackers-web
- https://github.com/jalts-808/j-hackers-api
- https://github.com/jalts-808/j-hackers-auth
- https://github.com/jalts-808/j-hackers-app

## CloudBees Unify URLs
- Components: https://cloudbees.io/cloudbees/eb3ae95d-a459-4f0a-ac58-57d752e4a373/components

## Key Decisions Made
- **Deployment**: k3s on single EC2 (~$8-30/month depending on size)
- **Feature Management**: Target j-hackers-web for FM example
- **Repo naming**: j-* prefix to distinguish from cloudbees-days originals
- **Build system**: CloudBees Unify workflows (Jenkins optional for learning)

## Workflow Triggers
| Repo | Trigger | Notes |
|------|---------|-------|
| j-hackers-web | `on: push` + `workflow_dispatch` | Auto-builds on push |
| j-hackers-api | `workflow_dispatch` only | Manual trigger |
| j-hackers-auth | `workflow_dispatch` only | Manual trigger |
| j-hackers-app | `workflow_dispatch` only | Manual trigger |

## CloudBees Unify Concepts
1. **Component** = Created via UI by connecting a repo
2. **Application** = Groups components together
3. **Release** = UI-created entity with workflow + manifest (artifact versions)
4. **Workflow** = YAML file in `.cloudbees/workflows/`, Composer is visual representation
5. **Manifest** = Found in Release Definition page, lists artifact versions grouped together

## Important Notes
- Do NOT do anything unless James explicitly asks
- James is learning - explain concepts clearly
- James prefers small, iterative commits
- The original repos are from `cloudbees-days` org (demo team)
- Fork relationship is safe - changes won't leak upstream

## Upstream Repos (for reference)
- https://github.com/cloudbees-days/hackers-web
- https://github.com/cloudbees-days/hackers-api
- https://github.com/cloudbees-days/hackers-auth
- https://github.com/cloudbees-days/hackers-app
