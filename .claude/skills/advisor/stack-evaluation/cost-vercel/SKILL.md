---
name: cost-vercel
description: Estimate Vercel costs for any project. Use when user asks about Vercel pricing, Next.js hosting costs, or edge deployment costs.
---

# Vercel Cost Evaluator

Estimate costs for Vercel-based projects.

## Standalone Usage

Can be invoked directly for cost estimation:
- "How much does Vercel Pro cost for 5 developers?"
- "Estimate Vercel costs for a Next.js app"
- "What's included in Vercel free tier?"

## Pricing Tiers

### Hobby ($0/mo)
| Resource | Limit |
|----------|-------|
| Bandwidth | 100 GB |
| Serverless Function Execution | 100 GB-hrs |
| Edge Function Invocations | 1M |
| Image Optimization | 1k source images |
| Build Minutes | 6,000/mo |
| Deployments | Unlimited |
| Preview Deployments | Yes |
| Team Members | 1 (personal only) |
| Commercial Use | **No** |

**Best for**: Personal projects, learning, non-commercial

### Pro ($20/user/month)
| Resource | Included | Overage |
|----------|----------|---------|
| Bandwidth | 1 TB | $0.15/GB |
| Serverless Execution | 1,000 GB-hrs | $0.18/GB-hr |
| Edge Functions | 1M invocations | $2/1M |
| Edge Middleware | 1M invocations | $0.65/1M |
| Image Optimization | 5k source images | $5/1k |
| Build Minutes | Unlimited | - |
| Preview Deployments | Unlimited | - |
| Password Protection | Yes | - |
| Web Analytics | 25k events/mo | Custom |

**Best for**: Professional teams, production apps

### Enterprise (Custom)
- SLA
- SSO/SAML
- Advanced security
- Dedicated support
- Custom limits

## Add-on Services

| Service | Free Tier | Pro Cost |
|---------|-----------|----------|
| Vercel Postgres | 256 MB | From $20/mo |
| Vercel KV (Redis) | 256 MB | From $10/mo |
| Vercel Blob | 250 MB | $0.15/GB |
| Vercel Edge Config | 8 KB | $0.10/read |
| Web Analytics | - | Included in Pro |
| Speed Insights | 10k data points | From $10/mo |

## Example Calculations

### Solo Developer (Hobby)
```
Hobby tier: $0
Bandwidth (50GB): Included
Functions (50 GB-hrs): Included
─────────────────────────────
Total: $0/mo (non-commercial only)
```

### Small Team (3 devs)
```
Pro tier (3 seats): $60/mo
Bandwidth (500GB): Included
Functions (500 GB-hrs): Included
Vercel Postgres: $20/mo
─────────────────────────────
Total: ~$80/mo
```

### Medium Team (10 devs)
```
Pro tier (10 seats): $200/mo
Bandwidth (2TB, 1TB extra): $150/mo
Functions (2k GB-hrs, 1k extra): $180/mo
Vercel Postgres (Pro): $60/mo
Vercel KV: $30/mo
─────────────────────────────
Total: ~$620/mo
```

### Large Scale
```
Pro tier (25 seats): $500/mo
Bandwidth (10TB): $1,350/mo
Functions (10k GB-hrs): $1,620/mo
Vercel Postgres: $200/mo
Edge Functions (50M): $100/mo
─────────────────────────────
Total: ~$3,770/mo
(Consider Enterprise tier)
```

## Output Contract

```yaml
vercel_cost_estimate:
  description: "<what's being estimated>"

  recommended_tier: "<hobby|pro|enterprise>"
  team_size: <N>

  baseline_monthly:
    seats: "<$X>"
    bandwidth_included: "<X TB>"
    bandwidth_overage: "<$X>"
    functions_included: "<X GB-hrs>"
    functions_overage: "<$X>"
    edge_functions: "<$X>"
    storage_services: "<$X>"
    total: "<$X>"

  at_10x_scale:
    seats: "<$X>"
    overages: "<$X>"
    total: "<$X>"

  included_features:
    - "Global edge network (CDN)"
    - "Automatic HTTPS"
    - "Preview deployments"
    - "Serverless functions"
    - "Edge functions"
    - "Git integration"
    - "Instant rollbacks"
    - "Analytics (Pro)"

  notes:
    - "<important note>"

  optimization_tips:
    - "<tip 1>"
```

## Vercel Considerations

**Strengths**:
- Zero-config deployment for Next.js
- Global edge network
- Preview deployments for every PR
- Excellent DX

**Watch out for**:
- Per-seat pricing adds up for larger teams
- Bandwidth overages can be expensive
- Function execution time affects cost
- Hobby tier is non-commercial only

## Cost Optimization Tips

1. **Static where possible**: Static pages = no function cost
2. **Use ISR**: Incremental Static Regeneration reduces functions
3. **Edge functions**: Cheaper than serverless for simple logic
4. **Optimize images**: Use next/image, it's cached
5. **External DB**: Vercel DB is convenient but pricier than alternatives
6. **Caching headers**: Reduce bandwidth with proper caching
7. **Consider alternatives**: For large teams, compare total cost with AWS/GCP
