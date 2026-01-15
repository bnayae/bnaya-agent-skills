---
name: cost-supabase
description: Estimate Supabase costs for any project. Use when user asks about Supabase pricing, BaaS costs, or PostgreSQL-based backend costs.
---

# Supabase Cost Evaluator

Estimate costs for Supabase-based projects.

## Standalone Usage

Can be invoked directly for cost estimation:
- "How much does Supabase Pro cost?"
- "Estimate Supabase costs for 50k users"
- "What's included in Supabase free tier?"

## Pricing Tiers

### Free Tier ($0/mo)
| Resource | Limit |
|----------|-------|
| Database | 500 MB |
| File Storage | 1 GB |
| Bandwidth | 2 GB |
| Monthly Active Users | 50,000 |
| Edge Functions | 500k invocations |
| Realtime | 200 concurrent |
| Compute | Shared |

**Best for**: Development, prototypes, hobby projects

### Pro Tier ($25/mo)
| Resource | Limit |
|----------|-------|
| Database | 8 GB |
| File Storage | 100 GB |
| Bandwidth | 50 GB |
| Monthly Active Users | 100,000 |
| Edge Functions | 2M invocations |
| Realtime | 500 concurrent |
| Compute | Dedicated (2 vCPU) |
| Backups | Daily |

**Best for**: Production apps, small-medium scale

### Team Tier ($599/mo)
Everything in Pro, plus:
- SOC2 compliance
- SSO/SAML
- Priority support
- 28-day log retention

**Best for**: Teams needing compliance

### Enterprise (Custom)
- Custom limits
- SLA
- Dedicated support
- Custom security

## Add-on Pricing (Pro+)

| Add-on | Cost |
|--------|------|
| Compute (per vCPU) | ~$25/mo |
| Database (per GB) | $0.125/GB |
| Storage (per GB) | $0.021/GB |
| Bandwidth (per GB) | $0.09/GB |
| Point-in-time Recovery | $100/mo |
| Custom domains | $10/mo |

## Example Calculations

### Small Production App
```
Pro tier base: $25/mo
Database (8GB included): $0
Storage (20GB, 19 extra): $0.40/mo
Bandwidth (30GB included): $0
─────────────────────────────
Total: ~$26/mo
```

### Medium Scale App (100k MAU)
```
Pro tier base: $25/mo
Extra compute (+2 vCPU): $50/mo
Database (20GB, 12 extra): $1.50/mo
Storage (50GB, 50 extra): $1.05/mo
Bandwidth (100GB, 50 extra): $4.50/mo
─────────────────────────────
Total: ~$82/mo
```

### Large Scale App (500k MAU)
```
Pro tier base: $25/mo
Extra compute (+6 vCPU): $150/mo
Database (100GB): $12.50/mo
Storage (200GB): $4.20/mo
Bandwidth (500GB): $45/mo
Point-in-time Recovery: $100/mo
─────────────────────────────
Total: ~$337/mo
```

## Output Contract

```yaml
supabase_cost_estimate:
  description: "<what's being estimated>"

  recommended_tier: "<free|pro|team|enterprise>"

  baseline_monthly:
    tier_base: "<$X>"
    compute_addon: "<$X>"
    database_addon: "<$X>"
    storage_addon: "<$X>"
    bandwidth_addon: "<$X>"
    other_addons: "<$X>"
    total: "<$X>"

  at_10x_scale:
    tier: "<$X>"
    addons: "<$X>"
    total: "<$X>"

  included_features:
    - "PostgreSQL database"
    - "Auth (email, OAuth, magic link, phone)"
    - "Realtime subscriptions"
    - "Edge Functions (Deno)"
    - "Storage (S3-compatible)"
    - "Vector embeddings (pgvector)"
    - "Database webhooks"
    - "Auto-generated APIs"

  limitations:
    - "<limitation 1>"

  optimization_tips:
    - "<tip 1>"
```

## Supabase Advantages

- **All-in-one**: DB + Auth + Storage + Functions + Realtime
- **PostgreSQL**: Full SQL access, extensions, triggers
- **Open source**: Can self-host if needed
- **Generous free tier**: Great for development
- **Simple pricing**: Predictable costs
- **Built-in Auth**: No separate auth service needed

## Cost Optimization Tips

1. **Use free tier for dev**: Create separate projects for dev/prod
2. **Optimize queries**: Reduce database compute needs
3. **Use storage policies**: Prevent abuse
4. **Cache at edge**: Reduce bandwidth with CDN
5. **Connection pooling**: Use PgBouncer (built-in)
6. **Self-host for scale**: Consider self-hosting at very large scale
