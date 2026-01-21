---
name: cost-mongodb
description: Estimate MongoDB Atlas costs including cluster tiers, serverless pricing, Atlas Search, Device Sync, and data transfer. Use when user asks about MongoDB Atlas pricing or cost estimation.
context: fork
agent: Explore
---

# MongoDB Atlas Cost Evaluator

Estimate costs for MongoDB Atlas deployments including dedicated clusters, serverless instances, and add-on features.

## Standalone Usage

Can be invoked directly:
- "How much does MongoDB Atlas M30 cost?"
- "Estimate costs for MongoDB serverless"
- "Atlas Search pricing for my use case"
- "Compare M10 vs M30 costs"

## Before Answering MongoDB Cost Questions

### Step 1: Check Available Tools

**MongoDB Atlas CLI**:
```bash
# Check current cluster costs
atlas clusters describe CLUSTER_NAME
```

**Atlas UI**: Check billing section for current usage

### Step 2: Fallback to Web Search

Use web search to fetch current pricing from:
- https://www.mongodb.com/pricing
- https://www.mongodb.com/docs/atlas/billing/

## Dedicated Cluster Pricing (Pay-as-you-go)

### Cluster Tiers (AWS us-east-1, 3-node replica set)

| Tier | RAM | Storage | Monthly Cost | Hourly Cost |
|------|-----|---------|--------------|-------------|
| **M10** | 2 GB | 10 GB | ~$60 | ~$0.08 |
| **M20** | 4 GB | 20 GB | ~$140 | ~$0.19 |
| **M30** | 8 GB | 40 GB | ~$280 | ~$0.39 |
| **M40** | 16 GB | 80 GB | ~$560 | ~$0.78 |
| **M50** | 32 GB | 160 GB | ~$1,100 | ~$1.53 |
| **M60** | 64 GB | 320 GB | ~$2,200 | ~$3.06 |
| **M80** | 128 GB | 750 GB | ~$4,400 | ~$6.11 |

### Storage Costs (Additional)

| Storage Type | Price per GB/month |
|--------------|-------------------|
| Standard | $0.25 |
| NVMe SSD | Included in tier price |

### Data Transfer

| Transfer Type | Price |
|---------------|-------|
| Same region | Free |
| Cross-region | $0.02/GB |
| Internet egress | $0.09/GB |

## Serverless Pricing

| Resource | Price |
|----------|-------|
| Read Processing Units (RPU) | $0.10 per million |
| Write Processing Units (WPU) | $1.00 per million |
| Storage | $0.25/GB/month |
| Data Transfer (out) | $0.015/GB |

### Serverless Cost Examples

| Workload | Reads/mo | Writes/mo | Storage | Monthly Cost |
|----------|----------|-----------|---------|--------------|
| Light | 1M | 100K | 5 GB | ~$2.50 |
| Medium | 10M | 1M | 50 GB | ~$15 |
| Heavy | 100M | 10M | 500 GB | ~$150 |

### When Serverless vs Dedicated

| Workload Pattern | Recommendation | Why |
|------------------|----------------|-----|
| < $50/mo usage | Serverless | Lower minimum |
| Variable traffic | Serverless | Auto-scale |
| Predictable high traffic | Dedicated | More cost-effective |
| Need sharding | Dedicated | Not available in serverless |
| Multi-region | Dedicated | Not available in serverless |

## Atlas Search Pricing

Atlas Search runs on the same cluster - costs are included in cluster tier.

| Consideration | Impact |
|---------------|--------|
| Index size | Increases storage |
| Search queries | CPU usage |
| Dedicated search nodes | Additional cost (M30+) |

### Dedicated Search Nodes (Optional)

| Node Size | Monthly Cost |
|-----------|--------------|
| S20 | ~$75 |
| S30 | ~$150 |
| S50 | ~$300 |
| S80 | ~$600 |

## Atlas Device Sync (Realm) Pricing

| Resource | Free Tier | Price Above |
|----------|-----------|-------------|
| Sync operations | 1M/month | $0.04/1000 |
| Data transfer | 10 GB | $0.12/GB |
| Requests | 3M/month | $1/100K (functions) |

## Backup Pricing

| Backup Type | Price |
|-------------|-------|
| Cloud Backup (continuous) | 2.5% of cluster cost |
| Legacy Backup | Included |
| Point-in-time restore | Included in continuous |

## Multi-Region Pricing

| Configuration | Additional Cost |
|---------------|-----------------|
| Cross-region replica | ~50% of primary |
| Multi-region cluster | 2-3x single region |
| Global write | Premium pricing |

## Cost Estimation Examples

### Small Production App

```yaml
configuration:
  cluster: M10
  storage: 20 GB
  region: us-east-1
  backup: cloud backup

monthly_cost:
  cluster: $60
  storage: $2.50 (10 GB additional)
  backup: $1.50
  total: ~$65/month
```

### Medium Production App

```yaml
configuration:
  cluster: M30
  storage: 100 GB
  region: us-east-1
  atlas_search: yes
  backup: cloud backup

monthly_cost:
  cluster: $280
  storage: $15 (60 GB additional)
  backup: $7
  total: ~$300/month
```

### Large Production App

```yaml
configuration:
  cluster: M50
  storage: 500 GB
  regions: [us-east-1, eu-west-1]
  atlas_search: dedicated S30
  backup: cloud backup

monthly_cost:
  primary_cluster: $1,100
  replica_region: $550
  storage: $85
  search_nodes: $150
  backup: $40
  total: ~$1,925/month
```

### Serverless App

```yaml
configuration:
  type: serverless
  storage: 10 GB
  reads: 5M/month
  writes: 500K/month

monthly_cost:
  rpus: $0.50
  wpus: $0.50
  storage: $2.50
  total: ~$4/month
```

## Cost Optimization Strategies

### Right-sizing

| Strategy | Savings Potential |
|----------|-------------------|
| Monitor cluster utilization | 20-50% |
| Use auto-scaling | Variable |
| Archive cold data | 30-70% |
| Optimize indexes | 10-30% |

### Reserved Capacity

| Commitment | Discount |
|------------|----------|
| 1 year | ~30% |
| 3 year | ~50% |

### Data Management

| Strategy | Impact |
|----------|--------|
| TTL indexes | Automatic cleanup |
| Archive to S3 | Cheaper storage |
| Compression | 50-90% storage reduction |

## Output Contract

```yaml
mongodb_cost_estimate:
  description: "<what's being estimated>"
  configuration:
    tier: "<M10|M30|serverless|etc>"
    storage_gb: <number>
    regions: []
    features:
      - "<atlas_search|device_sync|etc>"

  monthly_costs:
    cluster: "<$X>"
    storage: "<$X>"
    data_transfer: "<$X>"
    backup: "<$X>"
    features: "<$X>"
    total: "<$X>"

  scaling_projection:
    at_2x: "<$X>"
    at_5x: "<$X>"
    at_10x: "<$X>"

  cost_drivers:
    - "<primary cost driver>"
    - "<secondary driver>"

  optimization_opportunities:
    - type: "<reserved|right-sizing|archival>"
      potential_savings: "<X%>"
      trade_off: "<description>"

  comparison:
    vs_self_hosted: "<comparison>"
    vs_alternatives: "<comparison to other DBs>"
```

## Pricing Notes

- Prices vary by cloud provider (AWS, GCP, Azure)
- Regional pricing differences exist
- Always verify current pricing at mongodb.com/pricing
- Shared tiers (M0, M2, M5) have fixed pricing
- Data transfer costs often overlooked
