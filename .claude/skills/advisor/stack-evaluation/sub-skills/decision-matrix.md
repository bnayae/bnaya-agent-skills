---
name: decision-matrix
description: Build a weighted decision matrix to compare stack candidates. Use as part of stack-evaluation workflow.
---

# Decision Matrix Builder

Build a weighted decision matrix comparing all stack candidates.

## Criteria

| Criterion | Description |
|-----------|-------------|
| **cost** | Monthly infrastructure cost |
| **ops_overhead** | Operational complexity and maintenance burden |
| **reliability** | Uptime, failover, disaster recovery capabilities |
| **scalability** | Ability to handle growth and traffic spikes |
| **security** | Security features, compliance, access controls |
| **vendor_lock_in** | Difficulty of migrating away from the stack |
| **local_dev_dx** | Local development experience quality |

## Scoring Guide (1-5)

| Score | Cost | Ops Overhead | Other Criteria |
|-------|------|--------------|----------------|
| 1 | Very expensive | Very high ops burden | Poor |
| 2 | Expensive | High ops burden | Below average |
| 3 | Moderate | Moderate ops | Average |
| 4 | Affordable | Low ops burden | Good |
| 5 | Very cheap | Minimal ops | Excellent |

## Process

1. **Gather weights** - Ask user to assign importance (1-5) to each criterion
2. **Score candidates** - Rate each candidate on each criterion
3. **Calculate weighted scores** - `sum(score * weight) / sum(weights)`
4. **Rank candidates** - Order by weighted total

## Output Contract

```yaml
decision_matrix:
  criteria_weights:
    cost: <1-5>
    ops_overhead: <1-5>
    reliability: <1-5>
    scalability: <1-5>
    security: <1-5>
    vendor_lock_in: <1-5>
    local_dev_dx: <1-5>
  options:
    - candidate_id: "<id>"
      scores:
        cost: <1-5>
        ops_overhead: <1-5>
        reliability: <1-5>
        scalability: <1-5>
        security: <1-5>
        vendor_lock_in: <1-5>
        local_dev_dx: <1-5>
      weighted_total: <calculated>
      rank: <1-N>
```

## Default Weights

If user doesn't specify, use these defaults based on ops_maturity:

**Low ops maturity**: cost=4, ops_overhead=5, reliability=3, scalability=2, security=3, vendor_lock_in=2, local_dev_dx=4

**Medium ops maturity**: cost=3, ops_overhead=3, reliability=4, scalability=3, security=4, vendor_lock_in=3, local_dev_dx=3

**High ops maturity**: cost=3, ops_overhead=2, reliability=5, scalability=4, security=5, vendor_lock_in=4, local_dev_dx=3
