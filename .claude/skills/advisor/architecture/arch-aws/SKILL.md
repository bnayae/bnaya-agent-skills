---
name: arch-aws
description: Evaluate AWS architecture decisions including compute (EC2 vs Lambda vs ECS vs EKS), database (Aurora vs RDS vs DynamoDB), networking, and messaging. Use when user needs to choose between AWS services for their architecture.
color: blue
context: fork
agent: Explore
---

# AWS Architecture Decision Evaluator

Evaluate and recommend AWS service selections for compute, database, networking, and messaging based on requirements.

## Standalone Usage

Can be invoked directly:
- "Should I use Lambda or ECS for my API?"
- "Compare Aurora vs DynamoDB for my application"
- "What AWS compute option fits my workload?"
- "EKS vs ECS for my microservices"

## Before Answering AWS Architecture Questions

### Step 1: Check Available Tools

**AWS MCP Tools** (preferred):
- `mcp__aws-mcp__aws___search_documentation` - Search AWS docs
- `mcp__aws-mcp__aws___read_documentation` - Read specific service pages

**AWS CLI** (alternative):
- `aws describe-*` commands for service capabilities
- `aws pricing get-products` for cost considerations

### Step 2: If Tools Are Unavailable

Help the user set up AWS access:

```bash
# Install AWS CLI
brew install awscli  # macOS
# or: pip install awscli

# Configure credentials
aws configure
```

### Step 3: Fallback to Web Search

Use web search to fetch current information from:
- https://aws.amazon.com/documentation/
- AWS Architecture Center: https://aws.amazon.com/architecture/

## Compute Decision Matrix

| Service | Best For | Cold Start | Scaling | Ops Overhead | Cost Model |
|---------|----------|------------|---------|--------------|------------|
| **Lambda** | Event-driven, spiky traffic | 100ms-1s | Instant (1000 concurrent) | Minimal | Per invocation |
| **ECS Fargate** | Containers, steady traffic | None (running) | Minutes | Low | Per vCPU/memory |
| **ECS EC2** | Cost optimization, GPUs | None (running) | Minutes | Medium | EC2 + container |
| **EKS** | K8s ecosystem, portability | None (running) | Minutes | High | Cluster + nodes |
| **App Runner** | Simple containers | Seconds | Automatic | Minimal | Per vCPU/memory |
| **EC2** | Full control, specialized | None | Minutes (ASG) | High | Per instance |

### When to Use Each Compute Option

| Workload Type | Recommended | Why |
|---------------|-------------|-----|
| API with spiky traffic | Lambda + API Gateway | Pay-per-use, instant scale |
| Steady-state API | ECS Fargate | Predictable cost, no cold starts |
| Microservices (K8s team) | EKS | K8s features, team expertise |
| Microservices (simple) | ECS Fargate | Simpler than K8s |
| ML/GPU workloads | EC2 or EKS | GPU instance types |
| Simple web app | App Runner | Minimal config |
| Background jobs | Lambda or ECS | Depends on duration |
| Long-running processes | ECS or EC2 | Lambda 15min limit |

## Database Decision Matrix

| Service | Type | Best For | Scaling | Consistency | Ops Overhead |
|---------|------|----------|---------|-------------|--------------|
| **Aurora PostgreSQL** | Relational | OLTP, complex queries | Read replicas, Serverless v2 | Strong | Low |
| **Aurora MySQL** | Relational | MySQL compatibility | Read replicas, Serverless v2 | Strong | Low |
| **RDS PostgreSQL/MySQL** | Relational | Standard workloads | Read replicas | Strong | Low |
| **DynamoDB** | Key-Value/Document | High throughput, simple access | Unlimited | Eventual/Strong | Minimal |
| **DocumentDB** | Document | MongoDB compatibility | Read replicas | Strong | Low |
| **ElastiCache Redis** | Cache/Session | Caching, real-time | Cluster mode | Eventual | Low |
| **ElastiCache Memcached** | Cache | Simple caching | Horizontal | None | Low |
| **Neptune** | Graph | Relationships | Read replicas | Strong | Medium |
| **Timestream** | Time-series | IoT, metrics | Automatic | Strong | Minimal |
| **Keyspaces** | Wide-column | Cassandra workloads | Automatic | Eventual | Minimal |

### When to Use Each Database

| Use Case | Recommended | Why |
|----------|-------------|-----|
| General OLTP | Aurora PostgreSQL | Best price/performance |
| Simple key-value | DynamoDB | Serverless, unlimited scale |
| Complex queries + joins | Aurora/RDS PostgreSQL | SQL power |
| Caching layer | ElastiCache Redis | Low latency, data structures |
| MongoDB migration | DocumentDB | Compatibility |
| Graph relationships | Neptune | Native graph queries |
| Time-series data | Timestream | Optimized for time data |
| Session storage | ElastiCache or DynamoDB | Low latency |

## Networking & Edge Decision Matrix

| Service | Best For | Features | Cost Consideration |
|---------|----------|----------|-------------------|
| **CloudFront** | Global CDN | Edge caching, Lambda@Edge | Per request + transfer |
| **API Gateway REST** | REST APIs | Full features, caching | Per request |
| **API Gateway HTTP** | Simple APIs | Lower latency, cheaper | Per request (70% less) |
| **ALB** | Load balancing | Path routing, WebSocket | Per hour + LCU |
| **NLB** | TCP/UDP, extreme perf | Static IP, low latency | Per hour + LCU |
| **Global Accelerator** | Global TCP/UDP | Anycast IPs | Per hour + transfer |

### API Gateway: REST vs HTTP

| Aspect | REST API | HTTP API |
|--------|----------|----------|
| Cost | $3.50/million | $1.00/million |
| Latency | Higher | Lower |
| Features | Full (caching, WAF) | Basic |
| Auth | Cognito, IAM, Lambda | Cognito, IAM, JWT |
| Use When | Need caching/WAF | Simple APIs |

## Messaging & Events Decision Matrix

| Service | Pattern | Best For | Ordering | Retention |
|---------|---------|----------|----------|-----------|
| **SQS Standard** | Queue | Decoupling, buffering | Best-effort | 14 days |
| **SQS FIFO** | Queue | Ordered processing | Strict FIFO | 14 days |
| **SNS** | Pub/Sub | Fan-out, notifications | None | None |
| **EventBridge** | Event bus | Event-driven arch | Best-effort | 24 hours replay |
| **Kinesis Data Streams** | Streaming | Real-time analytics | Per shard | 7 days (extended: 365) |
| **MSK (Kafka)** | Streaming | Kafka workloads | Per partition | Configurable |
| **Step Functions** | Orchestration | Workflows | Sequential | N/A |

### When to Use Each Messaging Service

| Pattern | Recommended | Why |
|---------|-------------|-----|
| Simple async processing | SQS Standard | Easy, reliable |
| Ordered processing | SQS FIFO | Guaranteed order |
| Multiple subscribers | SNS + SQS | Fan-out pattern |
| Event-driven architecture | EventBridge | Schema registry, rules |
| Real-time streaming | Kinesis | High throughput |
| Complex workflows | Step Functions | Visual, error handling |

## Common Architecture Patterns

### Serverless API

```
CloudFront → API Gateway HTTP → Lambda → DynamoDB
                                      → Aurora Serverless v2
```

**Best for**: Variable traffic, low ops overhead
**Cost**: Pay-per-use

### Container-based API

```
CloudFront → ALB → ECS Fargate → Aurora PostgreSQL
                              → ElastiCache Redis
```

**Best for**: Steady traffic, predictable costs
**Cost**: Provisioned capacity

### Event-Driven Microservices

```
API Gateway → Lambda → EventBridge → Lambda (multiple)
                                   → SQS → ECS
                                   → SNS → External
```

**Best for**: Loosely coupled, scalable
**Cost**: Per-event

### Real-Time Analytics

```
Kinesis Data Streams → Kinesis Data Analytics → S3
                     → Lambda                  → Redshift
                     → Kinesis Firehose        → OpenSearch
```

**Best for**: IoT, clickstream, logs
**Cost**: Per shard + storage

## Output Contract

```yaml
architecture_decision:
  vendor: "aws"
  context: "<what was evaluated>"
  decision_point: "<specific decision>"
  decision_category: "<compute|database|networking|messaging>"

  options_evaluated:
    - id: "<service-id>"
      name: "<service name>"
      best_for:
        - "<use case>"
      avoid_when:
        - "<anti-pattern>"
      characteristics:
        cost_profile: "<low|medium|high|variable>"
        ops_complexity: "<minimal|low|medium|high>"
        scalability: "<limited|horizontal|auto-scaling|global>"
        cold_start: "<none|minimal|moderate|significant>"
        vendor_lock_in: "<low|medium|high>"
      score: <1-5>
      score_rationale: "<why>"

  recommendation:
    primary: "<service-id>"
    primary_rationale: "<why this service>"
    alternative: "<service-id>"
    alternative_rationale: "<when to use alternative>"

  decision_factors:
    - factor: "<factor name>"
      weight: "<high|medium|low>"
      notes: "<explanation>"

  implementation_guidance:
    getting_started:
      - "<step 1>"
    terraform_modules:
      - "<module reference>"
    common_pitfalls:
      - "<pitfall to avoid>"

  cost_considerations:
    baseline_estimate: "<$/month>"
    scaling_model: "<linear|step|logarithmic>"
    hidden_costs:
      - "<often overlooked cost>"

  related_decisions:
    - "<other decisions this affects>"
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Lambda for steady traffic | Expensive at scale | ECS Fargate |
| DynamoDB for complex queries | Limited query patterns | Aurora PostgreSQL |
| Overusing Step Functions | Expensive for simple flows | Direct Lambda calls |
| NAT Gateway for high egress | $0.045/GB adds up | VPC endpoints, architecture |
| Single AZ deployment | No fault tolerance | Multi-AZ always |
| Ignoring data transfer costs | Surprise bills | Architect for locality |

## Integration Points

- Use **cost-aws** to estimate costs for architecture choices
- Use **local-dev-evaluator** for local development setup
- Feeds into **implementation-plan** for deployment strategy
