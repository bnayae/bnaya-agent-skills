---
name: vercel-cost-evaluator
description: Estimate Vercel costs for stack candidates. Use as part of stack-evaluation for Vercel-based stacks.
---

# Vercel Cost Evaluator

Estimate costs for Vercel-based stack candidates.

## Pricing Tiers

### Hobby (Free)
- Personal, non-commercial use only
- 100 GB bandwidth
- 100 GB-hours serverless functions
- 6k build minutes
- **Best for**: Personal projects, learning

### Pro ($20/user/month)
- Commercial use allowed
- 1 TB bandwidth
- 1000 GB-hours serverless functions
- Unlimited build minutes
- Preview deployments
- Password protection
- **Best for**: Professional teams

### Enterprise (Custom)
- SLA
- SSO/SAML
- Advanced security
- Dedicated support

## Usage-Based Pricing (Pro+)

| Resource | Included | Additional |
|----------|----------|------------|
| Bandwidth | 1 TB | $0.15/GB |
| Serverless Functions | 1000 GB-hrs | $0.18/GB-hr |
| Edge Functions | 1M invocations | $2/1M |
| Image Optimization | 5k source images | $5/1k |
| Analytics | 25k events/mo | Custom |

## Common Add-ons

| Add-on | Cost |
|--------|------|
| Vercel Postgres | $20+/mo |
| Vercel KV (Redis) | $10+/mo |
| Vercel Blob | $0.15/GB |
| Web Analytics | Included in Pro |

## Cost Estimation Template

```yaml
vercel_cost_estimate:
  candidate_id: "<id>"
  recommended_tier: "<hobby|pro|enterprise>"
  team_size: <N>
  baseline_monthly:
    seats: "<$X>"
    bandwidth: "<$X>"
    functions: "<$X>"
    storage: "<$X>"
    total: "<$X>"
  at_10x_scale:
    seats: "<$X>"
    overages: "<$X>"
    total: "<$X>"
  included_features:
    - "Edge network (CDN)"
    - "Automatic HTTPS"
    - "Preview deployments"
    - "Serverless functions"
```

## Vercel Considerations

- Per-seat pricing adds up for larger teams
- Excellent for frontend-heavy apps
- Pairs well with external databases (Supabase, PlanetScale)
- Edge Functions for low-latency global APIs
