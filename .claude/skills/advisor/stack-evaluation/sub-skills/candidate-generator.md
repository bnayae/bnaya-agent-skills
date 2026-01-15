---
name: candidate-generator
description: Generate coherent stack candidates based on architecture requirements. Use as part of stack-evaluation workflow.
---

# Candidate Pattern Generator

Generate 2-5 coherent stack candidates based on the architecture brief.

## Example Patterns

| Pattern | Best For |
|---------|----------|
| React + AWS Amplify + Aurora Serverless | Rapid development, serverless, AWS-native |
| React + AWS App Runner + RDS/Aurora | Container-based, balanced control/convenience |
| React + Supabase | PostgreSQL-centric, open-source friendly |
| React + Firebase | Google ecosystem, real-time features |
| React + Cloud Run + Cloud SQL | GCP-native, container-based |
| Next.js + Vercel | Frontend-heavy, edge deployment |
| .NET + Azure App Service + Azure SQL | Microsoft ecosystem, enterprise |
| Node.js + ECS Fargate + RDS | AWS containers, moderate ops |

## Required Components

Each candidate must specify:

1. **Frontend stack** - framework/library and hosting
2. **Backend/API stack** - framework/runtime and hosting
3. **Database solution** - technology and managed/self-hosted
4. **Authentication approach** - provider/solution
5. **Hosting/deployment model** - platform and strategy
6. **Local development story** - how a dev runs this end-to-end locally

## Output Contract

```yaml
candidate:
  id: "<unique_id>"
  name: "<descriptive name>"
  summary: "<one-line summary>"
  components:
    frontend:
      technology: "<framework/library>"
      hosting: "<where it runs>"
    backend:
      technology: "<framework/runtime>"
      hosting: "<where it runs>"
    database:
      technology: "<database>"
      hosting: "<managed|self-hosted>"
    auth:
      provider: "<auth solution>"
    async:
      technology: "<queues/events if needed>"
    storage:
      technology: "<object storage if needed>"
  local_dev_story: |
    <Detailed description of how a developer runs this
    end-to-end on a laptop with Docker Desktop>
```

## Selection Criteria

When generating candidates, consider:

- **Budget constraints** - prefer serverless for minimal, containers for growth+
- **Ops maturity** - low maturity prefers fully managed services
- **Team size** - small teams need simpler stacks
- **Cloud preference** - stick to preferred cloud when specified
- **Compliance** - some providers have better compliance certifications
- **Local dev expectations** - favor stacks with good local tooling
