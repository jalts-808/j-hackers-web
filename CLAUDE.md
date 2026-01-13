# Claude Context - Hackers App Project

## Project Owner
James Altheide (jalts-808 on GitHub) - Growth Product Manager at CloudBees

## What This Project Is
Learning project to understand CloudBees Unify by building and deploying a 3-component application. James learns by doing - he needs to create things himself to understand them.

## End Goal
Run a Feature Management example against the hackers-web service, with everything deployed to a personal k3s instance on AWS.

## Repository Structure
```
3demo/
├── hackers-web/     # Vue.js frontend (forked from cloudbees-days)
├── hackers-api/     # Go backend API
├── hackers-auth/    # Go auth service
├── hackers-app/     # Release orchestration workflows
├── README.md
└── CLAUDE.md
```

## Key Decisions Made
- **Deployment**: k3s on single EC2 (~$8-15/month) - keeps Helm charts working
- **Feature Management**: Target hackers-web for FM example
- **Forked repos**: All 4 repos forked to jalts-808 GitHub account

## CloudBees Unify Concepts Understood
1. **Component** = Created via UI by connecting a repo
2. **Application** = Groups components together
3. **Release** = UI-created entity with workflow + manifest (artifact versions)
4. **Workflow** = YAML file in `.cloudbees/workflows/`, Composer is visual representation

## Workflow YAML Pattern
```yaml
metadata:
  stages/v1alpha1:
    - name: Stage Name
      jobs:
        - job-name-1    # References job definition below

jobs:
  job-name-1:
    steps:
      - uses: some-action
```

## What Needs to Be Done
1. Set up AWS EC2 with k3s
2. Configure secrets (DockerHub, kubeconfig)
3. Set up CloudBees components in James's sub-org
4. Modify workflows to point to jalts-808 repos/images
5. Create Application and Release
6. Implement Feature Management example on hackers-web

## Important Notes
- Do NOT do anything unless James explicitly asks
- James is learning - explain concepts clearly
- The original repos are from `cloudbees-days` org (demo team)
- James has access to CloudBees Unify UI, GitHub CLI, and Playwright MCP

## Upstream Repos (for reference)
- https://github.com/cloudbees-days/hackers-web
- https://github.com/cloudbees-days/hackers-api
- https://github.com/cloudbees-days/hackers-auth
- https://github.com/cloudbees-days/hackers-app
