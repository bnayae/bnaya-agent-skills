---
name: arch-cockroachdb
description: Evaluate CockroachDB architecture decisions including deployment options (Serverless vs Dedicated vs Self-hosted), multi-region patterns, consistency models, and migration from PostgreSQL. Use when user needs guidance on CockroachDB architecture choices.
context: fork
agent: Explore
---

# CockroachDB Architecture Decision Evaluator

Evaluate and recommend CockroachDB configurations, deployment options, and distributed database patterns for globally distributed applications.

## Standalone Usage

Can be invoked directly:
- "Should I use CockroachDB Serverless or Dedicated?"
- "How do I design multi-region tables in CockroachDB?"
- "CockroachDB vs PostgreSQL for my global app?"
- "What's the best topology for my CockroachDB cluster?"

## Before Answering CockroachDB Architecture Questions

### Step 1: Check Available Tools

**CockroachDB CLI**:
```bash
# Connect to cluster
cockroach sql --url "postgresql://..."

# Check cluster status
cockroach node status
```

### Step 2: If Tools Are Unavailable

Help the user set up CockroachDB access:

```bash
# Install cockroach CLI
brew install cockroachdb/tap/cockroach

# Or download from cockroachlabs.com
```

### Step 3: Fallback to Web Search

Use web search to fetch current information from:
- https://www.cockroachlabs.com/docs/
- https://www.cockroachlabs.com/docs/stable/architecture/overview.html

## Deployment Options

| Option | Best For | Scaling | Ops Overhead | Cost Model |
|--------|----------|---------|--------------|------------|
| **Serverless** | Variable traffic, dev | Automatic | None | Pay-per-use (RUs) |
| **Dedicated** | Production, predictable | Manual + auto | Low | Per vCPU/hour |
| **Self-hosted** | Full control, compliance | Manual | High | Infrastructure |

### When to Use Each Deployment

| Use Case | Recommended | Why |
|----------|-------------|-----|
| Development/testing | Serverless | Free tier, instant start |
| Variable/spiky traffic | Serverless | Auto-scale, pay-per-use |
| Predictable production | Dedicated | Better price at scale |
| Compliance requirements | Self-hosted or Dedicated | Data control |
| Multi-region production | Dedicated | Full topology control |
| Cost optimization at scale | Dedicated or Self-hosted | Predictable pricing |

### Serverless vs Dedicated

| Aspect | Serverless | Dedicated |
|--------|------------|-----------|
| Pricing | Request Units (RUs) | Per vCPU/hour |
| Min cost | $0 (free tier) | ~$300/mo (3 nodes) |
| Scaling | Automatic | Manual + auto-scale |
| Multi-region | Limited | Full support |
| Max storage | 50 GB (basic) | Unlimited |
| SLA | 99.9% | 99.99% |
| Best for | Dev, variable loads | Production |

## Multi-Region Patterns

### Topology Options

| Topology | Latency | Availability | Use Case |
|----------|---------|--------------|----------|
| **Single region** | Low | Regional | Local users |
| **Regional tables** | Low per region | Regional | Data locality |
| **Global tables** | Higher writes | Global | Reference data |
| **Follower reads** | Low reads | Regional | Read-heavy |

### Regional vs Global Tables

| Table Type | Write Latency | Read Latency | Consistency |
|------------|---------------|--------------|-------------|
| **Regional** | Low (single region) | Low | Strong |
| **Regional by row** | Low (home region) | Low (home) | Strong |
| **Global** | Higher (consensus) | Low (anywhere) | Strong |

### Multi-Region Configuration

```sql
-- Create a multi-region database
ALTER DATABASE mydb PRIMARY REGION "us-east1";
ALTER DATABASE mydb ADD REGION "us-west1";
ALTER DATABASE mydb ADD REGION "eu-west1";

-- Regional table (data stays in primary region)
ALTER TABLE users SET LOCALITY REGIONAL BY TABLE IN PRIMARY REGION;

-- Regional by row (each row has a home region)
ALTER TABLE orders SET LOCALITY REGIONAL BY ROW;

-- Global table (replicated everywhere, low-latency reads)
ALTER TABLE config SET LOCALITY GLOBAL;
```

### When to Use Each Pattern

| Pattern | Use When | Example |
|---------|----------|---------|
| **Regional by table** | All data for table in one region | Regional config |
| **Regional by row** | Data has natural region affinity | User data by location |
| **Global** | Small, read-heavy, rarely updated | Reference data, config |

## Consistency Models

### Default: Serializable Isolation

CockroachDB provides serializable isolation by default - the strongest level.

| Isolation Level | CockroachDB Support | Use Case |
|-----------------|---------------------|----------|
| **Serializable** | Default | All transactions |
| **Read committed** | Available | Performance trade-off |

### Read Patterns

| Pattern | Freshness | Latency | Use Case |
|---------|-----------|---------|----------|
| **Strong reads** | Current | Higher | Default, critical data |
| **Follower reads** | Slightly stale | Lower | Analytics, dashboards |
| **AS OF SYSTEM TIME** | Historical | Lowest | Reports, backups |

```sql
-- Follower read (bounded staleness)
SELECT * FROM users AS OF SYSTEM TIME follower_read_timestamp();

-- Historical query
SELECT * FROM orders AS OF SYSTEM TIME '-5m';
```

## Migration from PostgreSQL

### Compatibility

| Feature | Compatibility |
|---------|---------------|
| SQL syntax | High (PostgreSQL wire protocol) |
| Data types | Most supported |
| Indexes | B-tree, GIN, partial |
| Transactions | Full ACID |
| Stored procedures | Limited (PLpgSQL coming) |
| Extensions | Limited |

### Migration Steps

1. **Schema migration**: Export and adapt schema
2. **Data migration**: Use IMPORT or CDC
3. **Application changes**: Minimal (retry logic)
4. **Testing**: Verify query compatibility

### Key Differences from PostgreSQL

| Aspect | PostgreSQL | CockroachDB |
|--------|------------|-------------|
| Primary key | Optional | Required (or ROWID) |
| Sequences | Auto-increment | UUID preferred |
| Serial columns | Supported | Use UUID instead |
| Joins | Hash, merge, nested | Distributed joins |
| Transactions | Local | Distributed |

### Schema Best Practices for CockroachDB

```sql
-- Use UUID for primary keys (better distribution)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email STRING NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
);

-- Add indexes for query patterns
CREATE INDEX ON users (email);

-- For multi-region, add region column
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    region crdb_internal_region NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
) LOCALITY REGIONAL BY ROW;
```

## Performance Patterns

### Indexing Strategies

| Index Type | Use Case |
|------------|----------|
| **Primary key** | Required, use UUID |
| **Secondary** | Query patterns |
| **Covering** | Avoid table lookups |
| **Partial** | Filtered queries |
| **GIN** | JSON, arrays |

### Query Optimization

| Pattern | Benefit |
|---------|---------|
| **Limit scans** | Use WHERE, LIMIT |
| **Batch writes** | Reduce round trips |
| **Prepared statements** | Query plan caching |
| **Connection pooling** | Reduce connection overhead |

## Multi-Tenancy Patterns

| Pattern | Implementation | Pros | Cons |
|---------|----------------|------|------|
| **Column** | tenant_id column | Simple | Query complexity |
| **Schema** | Separate schemas | Isolation | Management overhead |
| **Database** | Separate databases | Full isolation | Resource overhead |
| **Row-level security** | Policies | Transparent | Performance |

### Recommended: Column-based with Regional Affinity

```sql
CREATE TABLE tenant_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    region crdb_internal_region NOT NULL,
    data JSONB,
    INDEX (tenant_id)
) LOCALITY REGIONAL BY ROW;
```

## Output Contract

```yaml
architecture_decision:
  vendor: "cockroachdb"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<deployment|topology|consistency|migration>"

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
        latency: "<low|medium|high|variable>"
        consistency: "<eventual|strong|serializable>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  topology_recommendations:
    regions: []
    table_localities:
      - table: "<table name>"
        locality: "<regional|global|regional-by-row>"
        reason: "<why>"

  implementation_guidance:
    getting_started:
      - "<step>"
    schema_changes:
      - "<change>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Auto-increment IDs | Hotspots | Use UUID |
| Single-region for global users | High latency | Multi-region deployment |
| Global tables for large data | Write latency | Regional by row |
| Ignoring connection pooling | Connection exhaustion | Use PgBouncer or built-in |
| Large transactions | Lock contention | Smaller batches |
| SELECT * | Network overhead | Select specific columns |

## Integration Points

- Use **cost-cockroachdb** to estimate costs
- Use **local-dev-evaluator** for local CockroachDB setup
- Feeds into **implementation-plan** for deployment
