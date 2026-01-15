---
name: cicd-phase
description: Detailed template for Phase 5 CI/CD Pipeline. Referenced by implementation-plan skill.
---

# Phase 5: CI/CD Pipeline

**Goals**:
- Automate build and test
- Enable continuous deployment to all environments
- Enforce quality and security gates

## Step Template

```yaml
phase_5_cicd:
  goals:
    - "Automate build and test"
    - "Enable continuous deployment"
    - "Enforce quality gates"

  steps:
    - id: "5.1"
      name: "Repository Strategy"
      decisions:
        - "Monorepo vs multi-repo"
        - "Repository naming convention"
      outputs:
        - "Repository structure document"

    - id: "5.2"
      name: "Branching Strategy"
      model: "<trunk-based|gitflow|github-flow>"
      branches:
        - name: "main"
          purpose: "Production-ready code"
          protection:
            - "Require PR"
            - "Require reviews"
            - "Require CI pass"
        - name: "feature/*"
          purpose: "Feature development"

    - id: "5.3"
      name: "CI Pipeline"
      platform: "<github-actions|gitlab-ci|azure-devops>"
      stages:
        - name: "Build"
          steps:
            - "Install dependencies"
            - "Compile/transpile"
            - "Generate artifacts"
        - name: "Test"
          steps:
            - "Unit tests"
            - "Integration tests"
            - "Coverage report"
        - name: "Lint"
          steps:
            - "Code linting"
            - "Format check"
        - name: "Security"
          steps:
            - "Dependency scan"
            - "SAST"
            - "Secrets detection"

    - id: "5.4"
      name: "Artifact Management"
      registry: "<ECR|GCR|ACR|Docker Hub>"
      tagging: "<git-sha|semver|timestamp>"

    - id: "5.5"
      name: "CD Pipeline"
      environments:
        - name: "dev"
          trigger: "push to main"
          approval: "none"
          strategy: "rolling"
        - name: "staging"
          trigger: "after dev success"
          approval: "none"
          strategy: "rolling"
        - name: "production"
          trigger: "manual or tag"
          approval: "required"
          strategy: "<rolling|blue-green|canary>"

    - id: "5.6"
      name: "Secrets Injection"
      approach: "<CI secrets|external secrets|vault>"

    - id: "5.7"
      name: "Quality Gates"
      gates:
        - "All tests pass"
        - "Coverage > X%"
        - "No critical vulnerabilities"
        - "Approval (production)"

    - id: "5.8"
      name: "Rollback Strategy"
      approach: "<automatic|manual>"
      procedure: "<steps>"

    - id: "5.9"
      name: "Preview Environments"
      enabled: true|false
      configuration: "<how they work>"
```

## GitHub Actions Example

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  deploy-dev:
    needs: [build, security]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Deploy to dev
        run: echo "Deploy to dev environment"
```

## CD Pipeline Example

```yaml
# .github/workflows/cd.yml
name: CD

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]

jobs:
  deploy-staging:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to staging
        run: echo "Deploy to staging"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - name: Deploy to production
        run: echo "Deploy to production"
```

## Quality Gate Configuration

```yaml
# Example: SonarQube quality gate
quality_gate:
  conditions:
    - metric: coverage
      operator: LT
      threshold: 80
      status: ERROR
    - metric: duplicated_lines_density
      operator: GT
      threshold: 3
      status: ERROR
    - metric: security_rating
      operator: GT
      threshold: 1  # A rating
      status: ERROR
```
