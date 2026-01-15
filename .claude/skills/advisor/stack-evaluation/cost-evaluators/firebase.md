---
name: firebase-cost-evaluator
description: Estimate Firebase costs for stack candidates. Use as part of stack-evaluation for Firebase-based stacks.
---

# Firebase Cost Evaluator

Estimate costs for Firebase-based stack candidates.

## Pricing Plans

### Spark Plan (Free)
- Firestore: 1 GB storage, 50k reads/day, 20k writes/day
- Realtime DB: 1 GB storage, 10 GB/mo download
- Storage: 5 GB, 1 GB/day download
- Functions: Not available
- Auth: 10k/mo phone auth
- **Best for**: Development, small apps

### Blaze Plan (Pay as you go)
- All Spark limits removed
- Pay per use
- Free tier included

## Blaze Plan Pricing

### Firestore
| Operation | Cost |
|-----------|------|
| Document reads | $0.036/100k |
| Document writes | $0.108/100k |
| Document deletes | $0.012/100k |
| Storage | $0.108/GB |

### Cloud Functions
| Resource | Cost |
|----------|------|
| Invocations | $0.40/1M |
| GB-seconds | $0.0000025 |
| CPU-seconds | $0.00001 |

### Storage
| Resource | Cost |
|----------|------|
| Storage | $0.026/GB |
| Downloads | $0.12/GB |

### Auth
| Type | Cost |
|------|------|
| Email/password | Free |
| Phone auth | $0.01-0.06/verification |
| SAML/OIDC | $0.015/MAU after 50 free |

## Cost Estimation Template

```yaml
firebase_cost_estimate:
  candidate_id: "<id>"
  recommended_plan: "<spark|blaze>"
  baseline_monthly:
    firestore: "<$X>"
    functions: "<$X>"
    storage: "<$X>"
    auth: "<$X>"
    hosting: "<$X>"
    total: "<$X>"
  at_10x_scale:
    firestore: "<$X>"
    functions: "<$X>"
    total: "<$X>"
  cost_drivers:
    - "<primary driver - usually Firestore reads>"
```

## Firebase Cost Considerations

- Firestore reads can be expensive at scale
- Optimize with caching and denormalization
- Functions cold starts add latency
- Consider Firebase + Cloud SQL for relational data
