---
name: cost-firebase
description: Estimate Firebase costs for any project. Use when user asks about Firebase pricing, Firestore costs, or Google BaaS costs.
---

# Firebase Cost Evaluator

Estimate costs for Firebase-based projects.

## Standalone Usage

Can be invoked directly for cost estimation:
- "How much does Firebase cost for 100k users?"
- "Estimate Firestore costs for my app"
- "What's in Firebase free tier?"

## Pricing Plans

### Spark Plan (Free)
| Resource | Limit |
|----------|-------|
| Firestore Storage | 1 GB |
| Firestore Reads | 50k/day |
| Firestore Writes | 20k/day |
| Firestore Deletes | 20k/day |
| Realtime DB Storage | 1 GB |
| Realtime DB Download | 10 GB/mo |
| Cloud Storage | 5 GB |
| Storage Download | 1 GB/day |
| Hosting Storage | 10 GB |
| Hosting Transfer | 360 MB/day |
| Cloud Functions | Not available |
| Phone Auth | 10k/mo |

**Best for**: Development, small apps, learning

### Blaze Plan (Pay as you go)
- All Spark limits removed
- Free tier usage still included
- Pay only for what you use above free tier

## Blaze Plan Pricing

### Firestore
| Operation | Cost | Free/mo |
|-----------|------|---------|
| Document reads | $0.036/100k | 50k/day |
| Document writes | $0.108/100k | 20k/day |
| Document deletes | $0.012/100k | 20k/day |
| Storage | $0.108/GB | 1 GB |

### Cloud Functions
| Resource | Cost | Free/mo |
|----------|------|---------|
| Invocations | $0.40/1M | 2M |
| GB-seconds | $0.0000025 | 400k |
| CPU-seconds | $0.00001 | 200k |
| Outbound networking | $0.12/GB | 5 GB |

### Cloud Storage
| Resource | Cost |
|----------|------|
| Storage | $0.026/GB |
| Download | $0.12/GB |
| Upload | Free |
| Operations | $0.05/10k |

### Authentication
| Type | Cost |
|------|------|
| Email/password | Free |
| Anonymous | Free |
| Phone (SMS) | $0.01-0.06/verification |
| SAML/OIDC | $0.015/MAU (50 free) |

### Hosting
| Resource | Cost | Free |
|----------|------|------|
| Storage | $0.026/GB | 10 GB |
| Transfer | $0.15/GB | 360 MB/day |

## Example Calculations

### Small App (10k MAU)
```
Firestore reads (500k/mo): ~$15/mo
Firestore writes (100k/mo): ~$8/mo
Firestore storage (2GB): $0.22/mo
Cloud Functions (500k inv): Free tier
Storage (5GB): Free tier
Auth: Free (email/password)
─────────────────────────────
Total: ~$25/mo
```

### Medium App (50k MAU)
```
Firestore reads (5M/mo): $180/mo
Firestore writes (1M/mo): $108/mo
Firestore storage (10GB): $1.08/mo
Cloud Functions (5M inv): $2/mo
Storage (50GB): $1.30/mo
Auth (phone, 10k): $100/mo
─────────────────────────────
Total: ~$392/mo
```

### Large App (200k MAU)
```
Firestore reads (50M/mo): $1,800/mo
Firestore writes (10M/mo): $1,080/mo
Firestore storage (100GB): $10.80/mo
Cloud Functions (20M inv): $8/mo
Storage (500GB): $13/mo
Hosting transfer (1TB): $150/mo
─────────────────────────────
Total: ~$3,062/mo
```

## Output Contract

```yaml
firebase_cost_estimate:
  description: "<what's being estimated>"

  recommended_plan: "<spark|blaze>"

  baseline_monthly:
    firestore_reads: "<$X>"
    firestore_writes: "<$X>"
    firestore_storage: "<$X>"
    cloud_functions: "<$X>"
    storage: "<$X>"
    hosting: "<$X>"
    auth: "<$X>"
    total: "<$X>"

  at_10x_scale:
    firestore: "<$X>"
    functions: "<$X>"
    total: "<$X>"

  cost_drivers:
    - "<primary driver - usually Firestore reads>"

  included_features:
    - "Firestore (NoSQL database)"
    - "Realtime Database"
    - "Authentication"
    - "Cloud Storage"
    - "Hosting (static + SSR)"
    - "Cloud Functions"
    - "Analytics"
    - "Crashlytics"
    - "Remote Config"

  warnings:
    - "<cost warning if applicable>"

  optimization_tips:
    - "<tip 1>"
```

## Firebase Cost Considerations

**Watch out for**:
- Firestore reads scale quickly (every listener = reads)
- Realtime listeners multiply read costs
- Cloud Functions cold starts (latency, not cost)
- Phone auth can get expensive

**Cost traps**:
- Listening to large collections
- Not using pagination
- Fetching entire documents for small fields
- Not caching on client

## Cost Optimization Tips

1. **Use caching**: Persist data locally to reduce reads
2. **Denormalize data**: Reduce reads with flatter structure
3. **Use pagination**: Don't load entire collections
4. **Composite indexes**: Reduce query complexity
5. **Use subcollections**: For large nested data
6. **Batch writes**: Reduce write operations
7. **Use Firebase Extensions**: Pre-built, optimized functions
8. **Consider hybrid**: Firebase for some features, SQL for others
