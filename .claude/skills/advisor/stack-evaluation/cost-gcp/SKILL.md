---
name: cost-gcp
description: Estimate GCP infrastructure costs for any stack. Use when user asks about Google Cloud pricing, cost estimation for Cloud Run, GKE, Cloud SQL, Compute Engine, or any GCP service combination.
---

# GCP Cost & Ops Evaluator

Estimate costs and operational overhead for GCP-based infrastructure.

## Standalone Usage

Can be invoked directly for cost estimation:
- "How much would Cloud Run + Cloud SQL cost?"
- "Estimate GCP costs for a Python API"
- "Compare GKE Autopilot vs Standard pricing"

## Common GCP Services Pricing

### Compute

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Cloud Functions | Free: 2M req/mo | $0.40/1M + duration | Low |
| Cloud Run | Pay per use | $30-200/mo | Low |
| GKE Autopilot | ~$70/mo minimum | $150-1000/mo | Medium |
| GKE Standard | ~$70/mo + nodes | $200-2000/mo | High |
| Compute Engine (e2-micro) | ~$6/mo | $30-300/mo | High |
| Compute Engine (e2-small) | ~$12/mo | $60-600/mo | High |

### Database

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Cloud SQL (db-f1-micro) | ~$8/mo | $50-300/mo | Medium |
| Cloud SQL (db-g1-small) | ~$25/mo | $100-500/mo | Medium |
| AlloyDB | ~$200/mo minimum | $500-3000/mo | Low |
| Cloud Spanner | ~$65/mo (100 PU) | $300-3000/mo | Low |
| Firestore | Pay per use | Variable | Low |
| Memorystore Redis | ~$35/mo (1GB) | $100-500/mo | Medium |

### Storage & CDN

| Service | Baseline | At Scale |
|---------|----------|----------|
| Cloud Storage | ~$0.020/GB | Variable |
| Cloud Storage (typical) | ~$1-5/mo | $10-100/mo |
| Cloud CDN | ~$5-15/mo | $50-500/mo |
| Persistent Disk | ~$0.04/GB/mo (standard) | Variable |

### Other Services

| Service | Cost | Notes |
|---------|------|-------|
| Identity Platform | Free < 50k MAU | $0.0055/MAU after |
| Pub/Sub | ~$40/TiB | Usually < $5/mo |
| Cloud Tasks | $0.40/1M operations | Usually < $5/mo |
| Secret Manager | $0.06/secret/mo | Much cheaper than AWS |
| Cloud Load Balancing | ~$18/mo + data | Per rule |
| Cloud NAT | ~$0.045/hr + data | ~$32/mo |
| Cloud DNS | $0.20/zone/mo | + queries |

## Example Calculations

### Basic API + Database
```
Cloud Run (1 vCPU, 512MB): ~$20/mo
Cloud SQL (db-f1-micro): $8/mo
Cloud Load Balancing: $18/mo
Cloud Storage (10GB): $1/mo
Cloud Logging: $5/mo
─────────────────────────────
Total: ~$52/mo baseline
```

### Serverless Stack
```
Cloud Functions (1M requests): Free tier
Cloud SQL (db-f1-micro): $8/mo
Cloud Storage + CDN: $10/mo
Identity Platform (10k MAU): Free
─────────────────────────────
Total: ~$20/mo baseline
```

### GKE + Cloud SQL + CDN
```
GKE Autopilot cluster: $72/mo
Autopilot pods (2 vCPU): $60/mo
Cloud SQL (db-g1-small): $25/mo
Cloud CDN: $15/mo
Cloud Load Balancing: $18/mo
Cloud Storage: $5/mo
Secret Manager (5 secrets): $0.30/mo
─────────────────────────────
Total: ~$195/mo baseline
```

## Output Contract

```yaml
gcp_cost_estimate:
  description: "<what's being estimated>"

  components:
    - service: "<service name>"
      config: "<configuration>"
      monthly_cost: "<$X>"
      notes: "<any notes>"

  baseline_monthly:
    compute: "<$X>"
    database: "<$X>"
    storage: "<$X>"
    networking: "<$X>"
    other: "<$X>"
    total: "<$X>"

  at_10x_scale:
    compute: "<$X>"
    database: "<$X>"
    networking: "<$X>"
    total: "<$X>"

  cost_drivers:
    - "<primary cost driver>"

  ops_overhead: "<low|medium|high>"
  required_gcp_knowledge: "<basic|intermediate|advanced>"

  optimization_tips:
    - "<tip 1>"
```

## GCP vs AWS Cost Notes

- **Cloud Run** often cheaper than App Runner for variable traffic
- **Cloud SQL** slightly cheaper than RDS at baseline
- **Sustained use discounts** apply automatically (up to 30% off)
- **Committed use discounts** save 57-70% (1-3 year)
- **Secret Manager** significantly cheaper than AWS Secrets Manager
- **Cloud Functions** has higher free tier than Lambda

## Cost Optimization Tips

1. **Sustained Use**: Automatic discounts for consistent usage
2. **Committed Use**: Save 57%+ with 1-3 year commitments
3. **Preemptible VMs**: Up to 80% off for fault-tolerant workloads
4. **Cloud Run min instances=0**: Only pay when handling requests
5. **Regional vs Multi-regional**: Choose based on actual needs
6. **Coldline/Archive Storage**: For infrequently accessed data
