---
name: gcp-cost-evaluator
description: Estimate GCP infrastructure costs for stack candidates. Use as part of stack-evaluation for GCP-based stacks.
---

# GCP Cost & Ops Evaluator

Estimate costs and ops overhead for GCP-based stack candidates.

## Common GCP Services Pricing

### Compute

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Cloud Functions | Free tier: 2M req/mo | $0.40/1M req + duration | Low |
| Cloud Run | Pay per use | $50-300/mo | Low |
| GKE Autopilot | ~$70/mo minimum | $150-1000/mo | Medium |
| Compute Engine | ~$10/mo (e2-micro) | $100-1000/mo | High |

### Database

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Cloud SQL (db-f1-micro) | ~$10/mo | $100-500/mo | Medium |
| Cloud Spanner | ~$65/mo minimum | $300-3000/mo | Low |
| Firestore | Pay per use | Variable | Low |

### Other Services

| Service | Baseline | Notes |
|---------|----------|-------|
| Cloud Storage | ~$1-5/mo | Storage + egress |
| Cloud CDN | ~$5-20/mo | CDN |
| Identity Platform | Free < 50k MAU | Auth |
| Pub/Sub | ~$1-5/mo | Messaging |
| Secret Manager | $0.06/secret/mo | Secrets |

## Cost Estimation Template

```yaml
gcp_cost_estimate:
  candidate_id: "<id>"
  baseline_monthly:
    compute: "<$X>"
    database: "<$X>"
    storage: "<$X>"
    networking: "<$X>"
    total: "<$X>"
  at_10x_scale:
    compute: "<$X>"
    database: "<$X>"
    total: "<$X>"
  cost_drivers:
    - "<primary cost driver>"
  ops_overhead: "<low|medium|high>"
  required_gcp_knowledge: "<basic|intermediate|advanced>"
```

## GCP vs AWS Notes

- Cloud Run often cheaper than App Runner for variable traffic
- Cloud SQL slightly cheaper than RDS at baseline
- GCP sustained use discounts apply automatically
