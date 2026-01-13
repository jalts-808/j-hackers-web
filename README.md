# Hackers App - CloudBees Unify Learning Project

A multi-repo application for learning CloudBees Unify, Kubernetes (k3s), and Feature Management.

## Project Goal

Reverse-engineer and understand CloudBees Unify by:
1. Running a 3-component app locally and on k3s (AWS EC2)
2. Understanding CloudBees workflow YAML files
3. Implementing Feature Management against the web UI
4. Being able to explain these concepts in customer demos

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    hackers-app (orchestrator)               │
│         Release workflows, deployment coordination          │
└─────────────────────┬───────────────────────────────────────┘
                      │ coordinates
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ hackers-web  │ │ hackers-api  │ │ hackers-auth │
│   (Vue.js)   │ │    (Go)      │ │    (Go)      │
│   Frontend   │ │   Backend    │ │ Auth Service │
└──────────────┘ └──────────────┘ └──────────────┘
```

## Repositories

| Repo | Description | Tech |
|------|-------------|------|
| `hackers-web` | Frontend UI | Vue.js |
| `hackers-api` | Backend API | Go |
| `hackers-auth` | Auth service | Go |
| `hackers-app` | Release orchestration | CloudBees workflows |

## Deployment Target

- **Infrastructure**: Single EC2 instance with k3s (lightweight Kubernetes)
- **Cost**: ~$8-15/month
- **Why k3s**: Existing Helm charts work as-is, representative of real deployments

## CloudBees Unify Concepts

### Hierarchy
```
Organization
  └── Application (groups components)
        ├── Components: hackers-web, hackers-api, hackers-auth
        └── Releases (manifest with specific versions)
```

### Key Files
- `.cloudbees/workflows/build.yaml` - CI pipeline (test, build, push image)
- `.cloudbees/workflows/deploy.yaml` - CD pipeline (Helm install)
- `hackers-app/.cloudbees/workflows/full-release-snow.yaml` - Release orchestration
- `chart/` - Helm charts for Kubernetes deployment

### Workflow YAML Structure
- `metadata.stages` - Visual layout for Workflow Composer
- `jobs:` - Actual job definitions with steps
- Jobs reference CloudBees actions (e.g., `cloudbees-io/kaniko`, `cloudbees-io/helm-install`)

## Feature Management

The web UI (`hackers-web`) has feature flag integration:
- `VUE_APP_FM_KEY` environment variable
- Release workflows include `enable-feature-flags-*` and `roll-out-feature-flags` jobs
- Uses `fm-update-flag` action to update flag configurations

## Getting Started

```bash
# All repos are siblings in this directory
ls -la
# hackers-web/
# hackers-api/
# hackers-auth/
# hackers-app/
```

## TODO

- [ ] Set up AWS EC2 with k3s
- [ ] Configure DockerHub for image storage
- [ ] Set up CloudBees components in personal sub-org
- [ ] Create Application and Release
- [ ] Implement Feature Management example
