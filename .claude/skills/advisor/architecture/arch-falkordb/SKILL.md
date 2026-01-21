---
name: arch-falkordb
description: Evaluate FalkorDB graph database architecture decisions including graph modeling, Cypher queries, Redis integration, and use cases like social graphs, recommendations, and knowledge graphs. Use when user needs guidance on graph database architecture.
context: fork
agent: Explore
---

# FalkorDB Architecture Decision Evaluator

Evaluate and recommend FalkorDB configurations, graph modeling patterns, and query strategies for relationship-heavy applications.

## Standalone Usage

Can be invoked directly:
- "Should I use FalkorDB or a relational database?"
- "How do I model a social graph in FalkorDB?"
- "FalkorDB vs Neo4j for my use case?"
- "Best practices for Cypher queries in FalkorDB"

## Before Answering FalkorDB Architecture Questions

### Step 1: Check Available Tools

**Redis CLI with FalkorDB**:
```bash
# Connect to FalkorDB
redis-cli

# Run a Cypher query
GRAPH.QUERY social "MATCH (u:User) RETURN u LIMIT 10"
```

### Step 2: If Tools Are Unavailable

Help the user set up FalkorDB:

```bash
# Docker (recommended)
docker run -p 6379:6379 -it --rm falkordb/falkordb

# Or with Redis Stack
docker run -p 6379:6379 redis/redis-stack
```

### Step 3: Fallback to Web Search

Use web search to fetch current information from:
- https://docs.falkordb.com/
- https://redis.io/docs/stack/graph/

## When to Use Graph Databases

### Graph vs Relational Decision

| Data Pattern | Graph DB | Relational |
|--------------|----------|------------|
| Deep relationships (4+ hops) | ✅ Better | ❌ Slow joins |
| Relationship queries | ✅ Native | ❌ Complex joins |
| Schema flexibility | ✅ Easy | ❌ Migrations |
| Aggregations | ❌ Limited | ✅ Better |
| ACID transactions | ⚠️ Limited | ✅ Full |
| Tabular reports | ❌ Not ideal | ✅ Better |

### Ideal Use Cases for FalkorDB

| Use Case | Why Graph |
|----------|-----------|
| Social networks | Friend-of-friend queries |
| Recommendation engines | Pattern matching |
| Fraud detection | Circular relationships |
| Knowledge graphs | Entity relationships |
| Network topology | Path finding |
| Access control | Permission inheritance |
| Supply chain | Multi-hop dependencies |

## Graph Modeling Patterns

### Node and Relationship Design

```cypher
// Nodes represent entities
CREATE (:User {id: 'u1', name: 'Alice', email: 'alice@example.com'})
CREATE (:Product {id: 'p1', name: 'Widget', price: 99.99})
CREATE (:Category {id: 'c1', name: 'Electronics'})

// Relationships connect entities
MATCH (u:User {id: 'u1'}), (p:Product {id: 'p1'})
CREATE (u)-[:PURCHASED {date: '2024-01-15', quantity: 2}]->(p)
```

### Common Modeling Patterns

| Pattern | Structure | Use Case |
|---------|-----------|----------|
| **Hub and spoke** | Central node with many connections | User with orders |
| **Linked list** | Sequential relationships | History, versioning |
| **Tree** | Hierarchical relationships | Categories, org chart |
| **Bipartite** | Two node types connected | Users-Products |
| **Hypergraph** | Intermediate nodes | Complex relationships |

### Property Guidelines

| Location | Store Here | Example |
|----------|------------|---------|
| Node properties | Entity attributes | name, email, created_at |
| Relationship properties | Interaction metadata | timestamp, weight, type |
| Separate nodes | Shared/complex data | Address, metadata |

## Cypher Query Patterns

### Basic Patterns

```cypher
// Find all friends
MATCH (u:User {name: 'Alice'})-[:FRIEND]->(friend)
RETURN friend.name

// Find friends of friends
MATCH (u:User {name: 'Alice'})-[:FRIEND*2]->(fof)
RETURN DISTINCT fof.name

// Find shortest path
MATCH path = shortestPath(
  (a:User {name: 'Alice'})-[:FRIEND*]-(b:User {name: 'Bob'})
)
RETURN path
```

### Recommendation Queries

```cypher
// Collaborative filtering: users who bought X also bought
MATCH (u:User)-[:PURCHASED]->(p:Product {name: 'Widget'})
MATCH (u)-[:PURCHASED]->(other:Product)
WHERE other.name <> 'Widget'
RETURN other.name, count(*) as freq
ORDER BY freq DESC
LIMIT 5

// Content-based: similar products
MATCH (p:Product {name: 'Widget'})-[:IN_CATEGORY]->(c:Category)
MATCH (similar:Product)-[:IN_CATEGORY]->(c)
WHERE similar <> p
RETURN similar.name
```

### Fraud Detection Queries

```cypher
// Find circular money transfers
MATCH path = (a:Account)-[:TRANSFER*3..6]->(a)
WHERE ALL(r IN relationships(path) WHERE r.amount > 10000)
RETURN path

// Find accounts with suspicious connections
MATCH (a:Account)-[:TRANSFER]->(b:Account)-[:TRANSFER]->(c:Account)
WHERE a.flagged = true OR c.flagged = true
RETURN DISTINCT b
```

## Performance Optimization

### Indexing Strategies

```cypher
// Create index on frequently queried property
GRAPH.QUERY social "CREATE INDEX FOR (u:User) ON (u.email)"

// Create index on relationship property
GRAPH.QUERY social "CREATE INDEX FOR ()-[p:PURCHASED]-() ON (p.date)"
```

### Index Types

| Index Type | Use Case | Syntax |
|------------|----------|--------|
| **Exact match** | Equality lookups | `CREATE INDEX FOR (n:Label) ON (n.prop)` |
| **Full-text** | Text search | `CREATE FULLTEXT INDEX ...` |

### Query Optimization Tips

| Tip | Why |
|-----|-----|
| Use indexes | Fast node lookup |
| Limit early | Reduce working set |
| Specify direction | Faster traversal |
| Avoid Cartesian products | Combinatorial explosion |
| Use PROFILE | Understand query plan |

```cypher
// Profile query to see execution plan
GRAPH.PROFILE social "MATCH (u:User)-[:FRIEND]->(f) RETURN f"
```

## FalkorDB vs Neo4j

| Aspect | FalkorDB | Neo4j |
|--------|----------|-------|
| Storage | Redis (in-memory) | Disk + cache |
| Performance | Very fast (memory) | Fast |
| Persistence | Redis persistence | Native |
| Clustering | Redis Cluster | Neo4j Cluster |
| Query language | Cypher subset | Full Cypher |
| ACID | Limited | Full |
| Cost | Open source | Enterprise license |
| Best for | Real-time, caching | Enterprise, full features |

### When to Choose FalkorDB

| Scenario | FalkorDB | Neo4j |
|----------|----------|-------|
| Real-time queries | ✅ | ⚠️ |
| Sub-millisecond latency | ✅ | ❌ |
| Already using Redis | ✅ | ❌ |
| Complex transactions | ❌ | ✅ |
| Large graphs (100M+ nodes) | ⚠️ | ✅ |
| Enterprise support needed | ❌ | ✅ |

## Deployment Patterns

### Standalone (Development)

```bash
docker run -p 6379:6379 falkordb/falkordb
```

### Redis Cluster (Production)

```yaml
# Multiple nodes for HA and scale
architecture:
  type: redis_cluster
  nodes: 6  # 3 masters, 3 replicas
  persistence: AOF
```

### Hybrid Architecture

```yaml
# Combine with relational for best of both
architecture:
  primary_db: PostgreSQL  # ACID, aggregations
  graph_db: FalkorDB     # Relationships, real-time
  sync: Change Data Capture
```

## Integration Patterns

### With TypeScript/Node.js

```typescript
import { createClient } from 'redis';
import { Graph } from 'redisgraph.js';

const client = createClient();
await client.connect();

const graph = new Graph(client, 'social');

// Run query
const result = await graph.query(
  'MATCH (u:User {name: $name})-[:FRIEND]->(f) RETURN f.name',
  { name: 'Alice' }
);
```

### With .NET

```csharp
using NRedisStack;
using NRedisStack.Graph;

var redis = ConnectionMultiplexer.Connect("localhost");
var db = redis.GetDatabase();
var graph = db.Graph();

var result = graph.Query("social",
    "MATCH (u:User {name: 'Alice'})-[:FRIEND]->(f) RETURN f.name");
```

## Output Contract

```yaml
architecture_decision:
  vendor: "falkordb"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<modeling|queries|deployment|integration>"

  options_evaluated:
    - id: "<option-id>"
      name: "<option name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        latency: "<sub-ms|low|medium|high>"
        scalability: "<limited|horizontal|cluster>"
        consistency: "<eventual|strong>"
        memory_usage: "<low|medium|high>"
      score: <1-5>

  recommendation:
    primary: "<option-id>"
    primary_rationale: "<why>"
    alternative: "<option-id>"

  graph_model:
    nodes:
      - label: "<label>"
        properties: []
        indexes: []
    relationships:
      - type: "<type>"
        from: "<label>"
        to: "<label>"
        properties: []

  implementation_guidance:
    getting_started:
      - "<step>"
    sample_queries:
      - "<query>"
    common_pitfalls:
      - "<pitfall>"

  related_decisions:
    - "<other decisions>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| No indexes | Full graph scans | Index lookup properties |
| Deep unbounded traversals | Memory exhaustion | Limit depth |
| Storing large blobs | Memory waste | Reference external storage |
| Over-modeling | Complexity | Keep it simple |
| Ignoring cardinality | Query performance | Model for query patterns |

## Integration Points

- Use **arch-supabase** or **arch-firebase** for primary data store
- Combine with relational DB for ACID transactions
- Feeds into **implementation-plan** for deployment
