---
name: cost-cockroachdb
description: Estimate CockroachDB costs including Serverless (Request Units), Dedicated clusters, multi-region deployments, and storage. Use when user asks about CockroachDB pricing or cost estimation.
color: yellow
context: fork
agent: Explore
---

# CockroachDB Cost Evaluator

Estimate costs for CockroachDB deployments including Serverless and Dedicated clusters.

## Standalone Usage

Can be invoked directly:
- "How much does CockroachDB Serverless cost?"
- "Estimate CockroachDB Dedicated cluster costs"
- "Compare Serverless vs Dedicated pricing"
- "Multi-region CockroachDB cost estimate"

## Before Answering CockroachDB Cost Questions

### Step 1: Check Available Tools

**CockroachDB Cloud Console**: Check usage and billing in the cloud console.

### Step 2: Fallback to Web Search

Use web search to fetch current pricing from:
- https://www.cockroachlabs.com/pricing/
- https://www.cockroachlabs.com/docs/cockroachcloud/

## Serverless Pricing

### Free Tier

| Resource | Free Allowance |
|----------|----------------|
| Request Units | 10M RUs/month |
| Storage | 5 GiB |

### Paid Usage

| Resource | Price |
|----------|-------|
| Request Units (RUs) | $1.00 per million |
| Storage | $1.00 per GiB/month |

### What Consumes Request Units?

| Operation | Approximate RUs |
|-----------|-----------------|
| Point read | 1 RU |
| Range scan (per row) | 0.25 RU |
| Write (per row) | 5 RUs |
| SQL CPU (per ms) | 0.33 RU |

### Serverless Cost Examples

| Workload | Reads/mo | Writes/mo | Storage | Monthly Cost |
|----------|----------|-----------|---------|--------------|
| Free tier | 10M | 2M | 5 GB | $0 |
| Light | 50M | 5M | 20 GB | ~$50 |
| Medium | 200M | 20M | 100 GB | ~$250 |
| Heavy | 1B | 100M | 500 GB | ~$1,500 |

## Dedicated Pricing

### Cluster Configurations (AWS)

| Config | vCPUs | RAM | Storage | Monthly Cost |
|--------|-------|-----|---------|--------------|
| **Basic (3-node)** | 2 per node | 8 GB | 75 GB | ~$300/mo |
| **Standard (3-node)** | 4 per node | 16 GB | 150 GB | ~$600/mo |
| **Performance (3-node)** | 8 per node | 32 GB | 300 GB | ~$1,200/mo |
| **High-memory (3-node)** | 16 per node | 64 GB | 600 GB | ~$2,400/mo |

### Per-vCPU Pricing

| Cloud Provider | Price per vCPU/hour |
|----------------|---------------------|
| AWS | ~$0.14 |
| GCP | ~$0.13 |
| Azure | ~$0.14 |

### Storage Pricing

| Storage Type | Price per GiB/month |
|--------------|---------------------|
| Standard | $0.50 |

### Network Transfer

| Transfer Type | Price |
|---------------|-------|
| Same region | Free |
| Cross-region | $0.01/GB |
| Internet egress | ~$0.05-0.12/GB |

## Multi-Region Pricing

Multi-region deployments multiply costs based on regions:

| Configuration | Cost Multiplier |
|---------------|-----------------|
| Single region | 1x |
| 2 regions | ~2x |
| 3 regions | ~3x |

### Multi-Region Example (3 regions, standard nodes)

```yaml
configuration:
  regions: [us-east1, us-west1, eu-west1]
  nodes_per_region: 3
  vcpus_per_node: 4
  storage_per_node: 150 GB

monthly_cost:
  compute: $600 x 3 regions = $1,800
  storage: 450 GB x $0.50 x 3 = $675
  network: ~$100 (estimated cross-region)
  total: ~$2,575/mo
```

## Cost Comparison: Serverless vs Dedicated

| Monthly Workload | Serverless Cost | Dedicated Cost | Recommendation |
|------------------|-----------------|----------------|----------------|
| < 50M RUs | ~$50 | ~$300 | Serverless |
| 50-200M RUs | ~$200 | ~$300 | Either |
| 200M-500M RUs | ~$500 | ~$300 | Dedicated |
| > 500M RUs | ~$1,000+ | ~$600+ | Dedicated |

### Breakeven Analysis

Serverless becomes more expensive than basic Dedicated at approximately:
- **300 million RUs/month** (~10M reads + 1M writes/day)
- **50+ GB storage**

## Enterprise Features (Additional Cost)

| Feature | Included In |
|---------|-------------|
| SSO/SCIM | Enterprise |
| Audit logging | Enterprise |
| Private endpoints | Dedicated + Enterprise |
| PCI compliance | Enterprise |
| 99.999% SLA | Enterprise |

## Cost Optimization Strategies

### Serverless Optimization

| Strategy | Savings |
|----------|---------|
| Batch writes | 20-50% RU reduction |
| Use indexes effectively | 50-80% RU reduction |
| Avoid SELECT * | 10-30% RU reduction |
| Connection pooling | Lower latency, fewer retries |

### Dedicated Optimization

| Strategy | Savings |
|----------|---------|
| Right-size cluster | 20-50% |
| Reserved capacity (committed use) | 30-50% |
| Use follower reads | Reduce cross-region traffic |
| Archive old data | Storage savings |

## Cost Estimation Examples

### Startup SaaS (Serverless)

```yaml
configuration:
  type: serverless
  storage: 10 GB
  monthly_reads: 100M
  monthly_writes: 10M

monthly_cost:
  rus: ~$100
  storage: $10
  total: ~$110/mo
```

### Growth Stage App (Dedicated)

```yaml
configuration:
  type: dedicated
  nodes: 3
  vcpus_per_node: 4
  storage: 300 GB
  region: us-east1

monthly_cost:
  compute: $600
  storage: $150
  total: ~$750/mo
```

### Global Enterprise App (Multi-Region Dedicated)

```yaml
configuration:
  type: dedicated_multi_region
  regions: [us-east1, eu-west1, ap-southeast1]
  nodes_per_region: 3
  vcpus_per_node: 8
  storage: 1 TB per region

monthly_cost:
  compute: $1,200 x 3 = $3,600
  storage: 1000 GB x $0.50 x 3 = $1,500
  network: ~$300
  support: (enterprise tier)
  total: ~$5,400+/mo
```

## Output Contract

```yaml
cockroachdb_cost_estimate:
  description: "<what's being estimated>"
  deployment_type: "<serverless|dedicated|self-hosted>"

  configuration:
    regions: []
    nodes_per_region: <number>
    vcpus_per_node: <number>
    storage_gb: <number>

  monthly_costs:
    compute: "<$X>"
    storage: "<$X>"
    network: "<$X>"
    support: "<$X>"
    total: "<$X>"

  serverless_metrics:
    estimated_rus: <number>
    ru_breakdown:
      reads: "<$X>"
      writes: "<$X>"
      cpu: "<$X>"

  scaling_projection:
    at_2x: "<$X>"
    at_5x: "<$X>"
    at_10x: "<$X>"

  optimization_opportunities:
    - type: "<right-sizing|reserved|batching>"
      potential_savings: "<X%>"
      trade_off: "<description>"

  vs_alternatives:
    vs_postgresql: "<comparison>"
    vs_spanner: "<comparison>"
    vs_aurora: "<comparison>"
```

## Pricing Notes

- Prices vary by cloud provider and region
- Reserved capacity discounts available for committed use
- Enterprise tier pricing is custom
- Always verify current pricing at cockroachlabs.com/pricing
- Network costs can be significant for multi-region
