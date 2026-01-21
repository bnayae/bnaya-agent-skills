---
name: arch-mongodb
description: Evaluate MongoDB Atlas architecture decisions including deployment options (Atlas dedicated vs serverless vs shared), cluster sizing, Atlas Search, Atlas Device Sync (Realm), and document modeling patterns. Use when user needs guidance on MongoDB architecture choices.
color: green
context: fork
agent: Explore
---

# MongoDB Atlas Architecture Decision Evaluator

Evaluate and recommend MongoDB Atlas configurations, deployment options, and document modeling patterns.

## Standalone Usage

Can be invoked directly:
- "Should I use MongoDB Atlas dedicated or serverless?"
- "How should I model my data in MongoDB?"
- "Atlas Search vs Elasticsearch for my use case?"
- "Do I need sharding for my MongoDB cluster?"

## Before Answering MongoDB Architecture Questions

### Step 1: Check Available Tools

**MongoDB Atlas CLI**:
```bash
# Install Atlas CLI
brew install mongodb-atlas-cli

# Login and check clusters
atlas auth login
atlas clusters list
```

**MongoDB Shell**:
```bash
# Check cluster stats
mongosh "mongodb+srv://..." --eval "db.serverStatus()"
```

### Step 2: If Tools Are Unavailable

Help the user set up MongoDB access:

```bash
# Install Atlas CLI
brew install mongodb-atlas-cli

# Or via npm
npm install -g mongodb-atlas-cli

# Authenticate
atlas auth login
```

### Step 3: Fallback to Web Search

Use web search to fetch current information from:
- https://www.mongodb.com/docs/atlas/
- https://www.mongodb.com/docs/manual/

## Deployment Options

| Tier | Best For | Auto-scaling | Backup | Price Range |
|------|----------|--------------|--------|-------------|
| **M0 (Free)** | Learning, prototypes | No | No | Free |
| **M2/M5 (Shared)** | Small apps, dev | No | Daily snapshots | $9-25/mo |
| **M10+ (Dedicated)** | Production | Yes | Continuous | $60+/mo |
| **Serverless** | Variable traffic | Yes (automatic) | Continuous | $0.10/million reads |

### When to Use Each Deployment

| Use Case | Recommended | Why |
|----------|-------------|-----|
| Learning/POC | M0 Free | No cost, basic features |
| Development | M2/M5 Shared | Low cost, sufficient for dev |
| Small production | M10 Dedicated | Dedicated resources, backups |
| Standard production | M30+ Dedicated | Performance, HA |
| Variable/spiky traffic | Serverless | Auto-scale, pay-per-use |
| High throughput | M40+ with NVMe | IOPS, low latency |
| Global users | Multi-region | Local reads, disaster recovery |

### Dedicated vs Serverless

| Aspect | Dedicated (M10+) | Serverless |
|--------|------------------|------------|
| Pricing | Per-hour | Per-operation |
| Scaling | Manual or auto-scale | Automatic |
| Max connections | Tier-dependent | 500 concurrent |
| Sharding | Yes | No |
| Multi-region | Yes | Coming |
| Triggers | Yes | Yes |
| Search | Yes | Yes |
| Best for | Predictable workloads | Variable traffic |

## Cluster Sizing Guide

| Tier | RAM | Storage | Use Case |
|------|-----|---------|----------|
| **M10** | 2 GB | 10-128 GB | Small production, low traffic |
| **M20** | 4 GB | 10-256 GB | Medium traffic, small datasets |
| **M30** | 8 GB | 10-512 GB | Standard production |
| **M40** | 16 GB | 10 GB-1 TB | High traffic, larger datasets |
| **M50** | 32 GB | 10 GB-4 TB | Large datasets, analytics |
| **M60+** | 64+ GB | Up to 4 TB | Enterprise, high performance |

### Sizing Recommendations

| Metric | Rule of Thumb |
|--------|---------------|
| Working set | Should fit in RAM |
| Storage | 2-3x current data for growth |
| IOPS | M40+ NVMe for high IOPS needs |
| Connections | ~100 per GB RAM |

## Atlas Features Decision Matrix

| Feature | Best For | Cost Impact |
|---------|----------|-------------|
| **Atlas Search** | Full-text search, facets | Additional compute |
| **Atlas Data Lake** | Analytics on S3 | Query-based |
| **Atlas Charts** | Visualizations | Free (included) |
| **Atlas Device Sync** | Mobile offline | Per-sync operation |
| **Atlas Triggers** | Event-driven logic | Per-execution |
| **Atlas Data Federation** | Query across sources | Query-based |

### Atlas Search vs Elasticsearch

| Aspect | Atlas Search | Elasticsearch |
|--------|--------------|---------------|
| Integration | Native (same cluster) | Separate service |
| Sync | Automatic | Manual/connector |
| Query language | $search aggregation | Elasticsearch DSL |
| Facets | Yes | Yes |
| Autocomplete | Yes | Yes |
| Ops overhead | None (managed) | High (if self-hosted) |
| Cost | Included in cluster | Separate infrastructure |
| Best for | MongoDB-native apps | Multi-source search |

## Document Modeling Patterns

### Embedding vs Referencing

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Embed** | 1:1, 1:few relationships | User + address |
| **Reference** | 1:many, many:many | User → orders |
| **Hybrid** | Mixed access patterns | User with recent orders embedded |

### Common Patterns

| Pattern | Use Case | Structure |
|---------|----------|-----------|
| **Attribute** | Variable attributes | `{ attributes: [{k, v}] }` |
| **Bucket** | Time-series | Group by hour/day |
| **Computed** | Aggregation results | Pre-compute counts |
| **Extended Reference** | Avoid joins | Embed frequently accessed fields |
| **Outlier** | Handle large arrays | Overflow documents |
| **Polymorphic** | Different types | Discriminator field |
| **Schema Versioning** | Evolution | Version field + migration |
| **Subset** | Large documents | Separate hot/cold data |

### Schema Design Checklist

```yaml
document_design:
  questions:
    - "What queries will run most frequently?"
    - "What is the read/write ratio?"
    - "How will data grow over time?"
    - "What are the cardinality relationships?"
    - "Do we need atomic updates across fields?"

  guidelines:
    - "Data that is accessed together should be stored together"
    - "Avoid unbounded arrays (max ~1000 elements)"
    - "Consider document size limit (16MB)"
    - "Index fields used in queries"
```

## Sharding Strategies

### When to Shard

| Indicator | Threshold | Action |
|-----------|-----------|--------|
| Data size | > 2 TB | Consider sharding |
| Write throughput | > 3000 ops/sec | Consider sharding |
| Working set | > available RAM | Consider sharding |
| Geographic distribution | Multi-region writes | Zone sharding |

### Shard Key Selection

| Shard Key Type | Pros | Cons | Example |
|----------------|------|------|---------|
| **Hashed** | Even distribution | No range queries | `_id` |
| **Ranged** | Range queries work | Hotspots possible | `timestamp` |
| **Compound** | Balanced | More complex | `{tenant, timestamp}` |
| **Zone** | Data locality | Complex setup | Geographic data |

### Shard Key Best Practices

```yaml
shard_key_selection:
  must_have:
    - High cardinality (many unique values)
    - Even distribution of writes
    - Present in most queries

  avoid:
    - Monotonically increasing values alone (timestamp, ObjectId)
    - Low cardinality fields (status, type)
    - Fields that change frequently
```

## Multi-Region Deployments

| Configuration | Use Case | Latency | Cost |
|---------------|----------|---------|------|
| **Single region** | Local users | Low | $ |
| **Cross-region replicas** | Read scaling | Low reads | $$ |
| **Multi-region cluster** | Global users | Low everywhere | $$$ |
| **Global write** | Write anywhere | Variable | $$$$ |

### Multi-Region Patterns

| Pattern | Description | Trade-off |
|---------|-------------|-----------|
| **Read from secondary** | Local reads | Eventual consistency |
| **Write to primary** | All writes to one region | Write latency for remote users |
| **Zone sharding** | Data locality | Complex management |

## Atlas Device Sync (Realm)

### When to Use

| Use Case | Recommendation |
|----------|----------------|
| Mobile offline-first | Yes |
| Real-time sync | Yes |
| Complex conflict resolution | Yes (custom handlers) |
| Simple mobile data | Consider direct API |

### Sync Patterns

| Pattern | Description | Best For |
|---------|-------------|----------|
| **Partition-based** | Sync subset by key | Multi-tenant |
| **Flexible sync** | Query-based sync | Complex filtering |
| **Asymmetric sync** | Write-only sync | IoT, logging |

## Output Contract

```yaml
architecture_decision:
  vendor: "mongodb"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<deployment|modeling|features|scaling>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        cost_profile: "<low|medium|high|variable>"
        ops_complexity: "<minimal|low|medium|high>"
        scalability: "<limited|horizontal|auto-scaling|global>"
        vendor_lock_in: "<low|medium|high>"
      score: <1-5>
      score_rationale: "<why>"

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"
    alternative_rationale: "<when to use>"

  schema_recommendations:
    collections:
      - name: "<collection>"
        pattern: "<embedding|referencing|hybrid>"
        indexes:
          - "<index definition>"
        sharding: "<shard key if needed>"

  implementation_guidance:
    getting_started:
      - "<step>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Unbounded arrays | Performance, 16MB limit | Reference or bucket pattern |
| No indexes | Slow queries | Index query fields |
| Over-embedding | Large documents | Reference pattern |
| ObjectId shard key alone | Monotonic hotspot | Hash or compound key |
| Ignoring working set | Poor performance | Size RAM appropriately |
| $lookup heavy | Slow joins | Embed or denormalize |

## Integration Points

- Use **cost-mongodb** to estimate Atlas costs
- Use **local-dev-evaluator** for local MongoDB setup
- Feeds into **implementation-plan** for deployment
