# Claude Context - Hackers App Project

## Project Owner
James Altheide (jalts-808 on GitHub) - Growth Product Manager at CloudBees

## What This Project Is
Learning project to understand CloudBees Unify by building and deploying a 3-component application. James learns by doing - he needs to create things himself to understand them.

## End Goal
Run a Feature Management example against the j-hackers-web service, with everything deployed to a personal k3s instance on AWS.

## Current State (Last Updated: Jan 12, 2025)

### Completed
- Forked all 4 repos from cloudbees-days to jalts-808
- Renamed repos with `j-` prefix (j-hackers-web, j-hackers-api, j-hackers-auth, j-hackers-app)
- Cloned all repos locally to `/Users/jalts/Desktop/3demo/`
- Created README.md and CLAUDE.md in j-hackers-web
- Components created in CloudBees Unify (demo1.cloudbees.io) - needs verification via Playwright

### Next Steps
- Verify components visible in Unify UI (Playwright session needs login)
- Decide Path A (AWS/k3s first) vs Path B (CloudBees config first) - James leaning toward understanding CloudBees side first
- Set up AWS EC2 with k3s
- Configure secrets and workflows

## Repository Structure
```
3demo/
├── j-hackers-web/   # Vue.js frontend - THIS IS THE MAIN REPO
├── j-hackers-api/   # Go backend API
├── j-hackers-auth/  # Go auth service
└── j-hackers-app/   # Release orchestration workflows
```

## GitHub Repos
- https://github.com/jalts-808/j-hackers-web
- https://github.com/jalts-808/j-hackers-api
- https://github.com/jalts-808/j-hackers-auth
- https://github.com/jalts-808/j-hackers-app

## CloudBees Unify URLs
- Components: https://ui.demo1.cloudbees.io/demo/6ca23bee-6bff-4637-9be0-824e2e4fb261/components?q=j-
- Note: This is demo1.cloudbees.io (different instance than cloudbees.io used earlier)

## Key Decisions Made
- **Deployment**: k3s on single EC2 (~$8-15/month) - keeps Helm charts working
- **Feature Management**: Target j-hackers-web for FM example
- **Repo naming**: j-* prefix to distinguish from cloudbees-days originals

## CloudBees Unify Concepts Understood
1. **Component** = Created via UI by connecting a repo
2. **Application** = Groups components together
3. **Release** = UI-created entity with workflow + manifest (artifact versions)
4. **Workflow** = YAML file in `.cloudbees/workflows/`, Composer is visual representation
5. **Manifest** = Found in Release Definition page, lists artifact versions grouped together

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

## Important Notes
- Do NOT do anything unless James explicitly asks
- James is learning - explain concepts clearly
- James prefers small, iterative commits
- The original repos are from `cloudbees-days` org (demo team)

## Upstream Repos (for reference)
- https://github.com/cloudbees-days/hackers-web
- https://github.com/cloudbees-days/hackers-api
- https://github.com/cloudbees-days/hackers-auth
- https://github.com/cloudbees-days/hackers-app
