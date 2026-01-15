---
name: supabase-cost-evaluator
description: Estimate Supabase costs for stack candidates. Use as part of stack-evaluation for Supabase-based stacks.
---

# Supabase Cost Evaluator

Estimate costs for Supabase-based stack candidates.

## Pricing Tiers

### Free Tier
- 500 MB database
- 1 GB file storage
- 2 GB bandwidth
- 50k monthly active users (auth)
- Shared compute
- **Best for**: Development, prototypes

### Pro Tier ($25/mo)
- 8 GB database
- 100 GB file storage
- 50 GB bandwidth
- 100k monthly active users
- Dedicated compute (2 vCPU)
- Daily backups
- **Best for**: Production apps, small-medium scale

### Team Tier ($599/mo)
- Everything in Pro
- SOC2 compliance
- Priority support
- SSO
- **Best for**: Teams needing compliance

### Enterprise (Custom)
- Custom limits
- SLA
- Dedicated support

## Add-on Costs

| Add-on | Cost |
|--------|------|
| Compute (per GB RAM) | $10/mo |
| Database (per GB) | $0.125/GB |
| Storage (per GB) | $0.021/GB |
| Bandwidth (per GB) | $0.09/GB |

## Cost Estimation Template

```yaml
supabase_cost_estimate:
  candidate_id: "<id>"
  recommended_tier: "<free|pro|team|enterprise>"
  baseline_monthly:
    tier: "<$X>"
    compute_addon: "<$X>"
    storage_addon: "<$X>"
    total: "<$X>"
  at_10x_scale:
    tier: "<$X>"
    addons: "<$X>"
    total: "<$X>"
  included_features:
    - "PostgreSQL database"
    - "Auth (email, OAuth, magic link)"
    - "Realtime subscriptions"
    - "Edge Functions"
    - "Storage"
    - "Vector embeddings"
```

## Supabase Advantages

- All-in-one platform (DB + Auth + Storage + Functions)
- PostgreSQL with full SQL access
- Open source (can self-host)
- Generous free tier for development
