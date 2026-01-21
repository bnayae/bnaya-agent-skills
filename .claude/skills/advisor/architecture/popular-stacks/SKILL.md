---
name: popular-stacks
description: Recommend popular, pre-validated architecture stack combinations for TypeScript and .NET projects. Use when user asks for stack recommendations, "what stack should I use", or needs a complete architecture suggestion.
color: indigo
context: fork
agent: Explore
---

# Popular Stacks Recommender

Recommend pre-validated, commonly used architecture combinations for TypeScript and .NET backend projects.

## Language Constraints

- **Frontend**: Always TypeScript (Next.js, Remix, Nuxt, SvelteKit, React)
- **Backend**: TypeScript or .NET (ASP.NET Core) - never Blazor for frontend

## Standalone Usage

Can be invoked directly:
- "What stack should I use for a SaaS app?"
- "Recommend a stack for my startup"
- "Best stack for a .NET backend with React frontend?"
- "What's a good TypeScript full-stack setup?"

## TypeScript Full-Stack Stacks

### Vercel Full-Stack (Recommended for MVPs)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js (App Router) |
| Backend | Next.js Server Actions / API Routes |
| Database | Vercel Postgres or Neon |
| Auth | NextAuth.js or Clerk |
| Hosting | Vercel |

**Best for**: Rapid MVP development, SaaS applications, startups
**Pros**: Fast deployment, excellent DX, integrated platform
**Cons**: Vercel lock-in, costs at scale

```bash
# Quick start
npx create-next-app@latest my-app --typescript
cd my-app
vercel
```

### Supabase Stack (Recommended for Real-time)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js or SvelteKit |
| Backend | Supabase Edge Functions |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Hosting | Vercel / Netlify |

**Best for**: Real-time apps, startups, rapid prototyping
**Pros**: Built-in auth/storage/realtime, generous free tier
**Cons**: Limited compute, PostgreSQL only

```bash
# Quick start
npx create-next-app@latest my-app --typescript
npm install @supabase/supabase-js
npx supabase init
```

### Firebase Stack (Recommended for Mobile-first)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js or React |
| Backend | Firebase Cloud Functions |
| Database | Firestore |
| Auth | Firebase Auth |
| Hosting | Firebase Hosting |

**Best for**: Mobile-first apps, rapid prototyping, real-time sync
**Pros**: Real-time sync, offline support, Google ecosystem
**Cons**: NoSQL limitations, vendor lock-in

### T3 Stack (Recommended for Type-safety)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js |
| Backend | tRPC |
| Database | PostgreSQL + Prisma |
| Auth | NextAuth.js |
| Hosting | Vercel |

**Best for**: Type-safe full-stack, teams valuing DX
**Pros**: End-to-end type safety, great DX
**Cons**: Learning curve for tRPC

```bash
# Quick start
npm create t3-app@latest
```

### Remix + Fly (Recommended for Forms-heavy)

| Component | Technology |
|-----------|------------|
| Frontend | Remix |
| Backend | Remix loaders/actions |
| Database | PostgreSQL (Fly or Neon) |
| Auth | Remix Auth |
| Hosting | Fly.io |

**Best for**: Form-heavy applications, global edge deployment
**Pros**: Progressive enhancement, web standards, global edge
**Cons**: Smaller ecosystem than Next.js

### AWS Serverless TypeScript

| Component | Technology |
|-----------|------------|
| Frontend | Next.js or React |
| Backend | Lambda + API Gateway |
| Database | Aurora Serverless / DynamoDB |
| Auth | Cognito |
| Hosting | CloudFront + S3 |

**Best for**: Enterprise AWS shops, variable traffic
**Pros**: Scalable, pay-per-use, AWS ecosystem
**Cons**: Complexity, cold starts

### GCP Serverless TypeScript

| Component | Technology |
|-----------|------------|
| Frontend | Next.js or React |
| Backend | Cloud Run / Cloud Functions |
| Database | Cloud SQL / Firestore |
| Auth | Firebase Auth / Identity Platform |
| Hosting | Cloud Run / Firebase Hosting |

**Best for**: Google ecosystem, AI/ML integration
**Pros**: Good AI integration, Cloud Run simplicity
**Cons**: Smaller community than AWS

### Cloudflare Stack (Recommended for Edge-first)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js / Remix / SvelteKit |
| Backend | Cloudflare Workers |
| Database | D1 / Turso |
| Auth | Cloudflare Access |
| Hosting | Cloudflare Pages |

**Best for**: Global edge deployment, cost-conscious
**Pros**: Fast globally, cheap, simple
**Cons**: Edge runtime limitations

## .NET Backend Stacks

Note: .NET is used for **backend only**. Frontend always uses TypeScript frameworks.

### Azure + Next.js (Recommended for Enterprise)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js (TypeScript) |
| Backend | ASP.NET Core |
| Database | Azure SQL / Cosmos DB |
| Auth | Azure AD B2C |
| Hosting | Azure App Service + Vercel |

**Best for**: Enterprise Microsoft shops
**Pros**: Enterprise features, Azure integration
**Cons**: Complexity, costs

### .NET Aspire + Next.js (Recommended for Distributed)

| Component | Technology |
|-----------|------------|
| Frontend | Next.js / React (TypeScript) |
| Backend | ASP.NET Core + Aspire |
| Database | PostgreSQL / SQL Server |
| Auth | Identity Server / Azure AD |
| Hosting | Azure Container Apps |

**Best for**: Distributed .NET applications
**Pros**: Unified observability, local dev story
**Cons**: Azure focus

### AWS .NET + Next.js

| Component | Technology |
|-----------|------------|
| Frontend | Next.js / React (TypeScript) |
| Backend | ASP.NET Core on ECS/Lambda |
| Database | Aurora / DynamoDB |
| Auth | Cognito |
| Hosting | ECS Fargate + Vercel/CloudFront |

**Best for**: .NET shops on AWS
**Pros**: AWS ecosystem, scalable
**Cons**: Complexity

### GCP .NET + Next.js

| Component | Technology |
|-----------|------------|
| Frontend | Next.js / React (TypeScript) |
| Backend | ASP.NET Core on Cloud Run |
| Database | Cloud SQL / AlloyDB |
| Auth | Identity Platform |
| Hosting | Cloud Run + Vercel |

**Best for**: .NET shops on GCP
**Pros**: Cloud Run simplicity, good .NET support
**Cons**: Smaller .NET community on GCP

### Supabase + .NET API

| Component | Technology |
|-----------|------------|
| Frontend | Next.js / SvelteKit (TypeScript) |
| Backend | ASP.NET Core API |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Hosting | Vercel + Azure/AWS |

**Best for**: .NET API with modern frontend
**Pros**: Best of both worlds
**Cons**: Two hosting platforms

### .NET + Remix

| Component | Technology |
|-----------|------------|
| Frontend | Remix (TypeScript) |
| Backend | ASP.NET Core API |
| Database | PostgreSQL (any) |
| Auth | ASP.NET Identity |
| Hosting | Fly.io + Azure/AWS |

**Best for**: Forms-heavy .NET applications
**Pros**: Progressive enhancement, .NET backend
**Cons**: Two deployments

## Stack Selection Guide

### By Use Case

| Use Case | Recommended Stack |
|----------|-------------------|
| MVP / Startup | Vercel Full-Stack or Supabase |
| SaaS | T3 Stack or Vercel Full-Stack |
| E-commerce | Next.js + Vercel + external DB |
| Real-time app | Supabase or Firebase |
| Forms-heavy | Remix + Fly |
| Mobile-first | Firebase |
| Enterprise (Microsoft) | Azure + Next.js |
| Enterprise (AWS) | AWS Serverless |
| Global edge | Cloudflare Stack |
| .NET backend | Azure + Next.js or Supabase + .NET |

### By Team Expertise

| Team Background | Recommended |
|-----------------|-------------|
| React developers | Next.js stacks |
| Vue developers | Nuxt + Supabase |
| Svelte developers | SvelteKit + Supabase |
| .NET developers | .NET + Next.js/React |
| Full-stack JS | T3 Stack |
| DevOps heavy | AWS/GCP Serverless |
| Minimal ops | Vercel/Supabase |

### By Budget

| Budget | Recommended |
|--------|-------------|
| Free tier / Bootstrap | Supabase + Vercel free |
| Startup ($100-500/mo) | Vercel Pro + Supabase Pro |
| Growth ($500-2000/mo) | Dedicated infrastructure |
| Enterprise | Cloud provider managed |

## Selection Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Team expertise | High | Match stack to existing skills |
| Time to market | High | BaaS stacks fastest |
| Scale requirements | Medium | Consider growth trajectory |
| Vendor preference | Medium | Enterprise often has mandates |
| Cost sensitivity | Medium | Serverless vs provisioned |
| Compliance needs | High | May dictate cloud choices |

## Output Contract

```yaml
popular_stack_recommendation:
  language: "<typescript|dotnet>"
  selected_stack: "<stack name>"

  components:
    frontend:
      technology: "<framework>"
      language: "TypeScript"
    backend:
      technology: "<framework/platform>"
      language: "<TypeScript|C#>"
    database:
      primary: "<database>"
      type: "<relational|document|graph>"
    auth:
      provider: "<auth provider>"
    hosting:
      frontend: "<platform>"
      backend: "<platform>"
      regions: []

  rationale:
    - "<why this combination>"

  trade_offs:
    pros:
      - "<advantage>"
    cons:
      - "<disadvantage>"

  alternatives:
    - stack: "<alternative stack>"
      when: "<when to consider>"

  getting_started:
    - step: "<step description>"
      command: "<command if applicable>"

  estimated_costs:
    free_tier: "<what's free>"
    starter: "<$/month for starter>"
    growth: "<$/month at scale>"

  reference_repos:
    - name: "<repo name>"
      url: "<github url>"
      description: "<what it demonstrates>"
```

## Quick Comparison Table

| Stack | DX | Scale | Cost | Complexity | Best For |
|-------|-------|-------|------|------------|----------|
| Vercel Full-Stack | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | $$$ | Low | MVPs, SaaS |
| Supabase | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | $$ | Low | Real-time |
| Firebase | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | $$ | Medium | Mobile-first |
| T3 Stack | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | $$ | Medium | Type-safety |
| AWS Serverless | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | $-$$$ | High | Enterprise |
| Azure + .NET | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | $$$ | High | MS Enterprise |
| Cloudflare | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | $ | Medium | Edge-first |

## Integration Points

- Use individual **arch-*** skills for deep dives
- Use **cost-*** skills for detailed pricing
- Feeds into **implementation-plan** for execution
