---
name: local-dev-evaluator
description: Evaluate local development experience for stack candidates. Use as part of stack-evaluation workflow.
---

# Local Dev Experience Evaluator

Evaluate and recommend the best local development approach for each stack candidate.

## Assumptions

- Docker Desktop is available
- Local Kubernetes is available (Docker Desktop K8s, minikube, kind, or k3s)

## Available Approaches

| Approach | When to Use |
|----------|-------------|
| `docker-compose` | Simple apps, all deps containerized, no K8s features needed |
| `local-k8s` | K8s-native apps, need K8s features (services, ingress, configmaps) |
| `hybrid` | Complex apps: deps in compose, services in k8s |
| `aspire` | .NET distributed apps needing orchestration + observability |

## Evaluation Criteria

### Time to First Run

| Rating | Time | Examples |
|--------|------|----------|
| Excellent | < 5 min | Single docker-compose up, pre-built images |
| Good | 5-15 min | Multiple containers, some build time |
| Fair | 15-30 min | Complex setup, many dependencies |
| Poor | > 30 min | Manual steps, external dependencies |

### Dependency Complexity

- **Low**: 1-3 containers, standard images (postgres, redis)
- **Medium**: 4-6 containers, some custom config
- **High**: 7+ containers, complex networking, custom builds

### Prod Parity

| Area | Full Parity | Partial | None |
|------|-------------|---------|------|
| **Auth** | Same provider/flow | Mock/simplified | Disabled |
| **Data** | Same DB, migrations | Same DB, seed data | Different DB |
| **Async** | Same queue/events | Local queue | Sync only |
| **Config** | Same config system | .env files | Hardcoded |

## Output Contract

```yaml
local_dev_evaluation:
  candidate_id: "<id>"
  recommended_approach: "<docker-compose|local-k8s|hybrid|aspire>"
  time_to_first_run: "<excellent|good|fair|poor>"
  dependency_complexity: "<low|medium|high>"
  prod_parity:
    auth: "<full|partial|none>"
    data: "<full|partial|none>"
    async: "<full|partial|none>"
    config: "<full|partial|none>"
  debugging_support: "<excellent|good|fair|poor>"
  dev_observability: "<excellent|good|fair|poor>"
  dx_score: <1-5>
  notes: "<special considerations>"
```

## Scoring Guide

| DX Score | Criteria |
|----------|----------|
| 5 | < 5 min setup, low complexity, excellent parity, great debugging |
| 4 | 5-15 min setup, low-medium complexity, good parity |
| 3 | 15-30 min setup, medium complexity, partial parity |
| 2 | > 30 min setup, high complexity, limited parity |
| 1 | Complex manual setup, poor parity, difficult debugging |
