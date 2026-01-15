---
name: aws-cost-evaluator
description: Estimate AWS infrastructure costs for stack candidates. Use as part of stack-evaluation for AWS-based stacks.
---

# AWS Cost & Ops Evaluator

Estimate costs and ops overhead for AWS-based stack candidates.

## Common AWS Services Pricing

### Compute

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Lambda | Free tier: 1M req/mo | $0.20/1M req + duration | Low |
| App Runner | ~$25/mo (1 vCPU) | $50-200/mo | Low |
| ECS Fargate | ~$30/mo (0.5 vCPU) | $100-500/mo | Medium |
| EC2 | ~$15/mo (t3.micro) | $100-1000/mo | High |

### Database

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Aurora Serverless v2 | ~$50/mo (0.5 ACU) | $200-2000/mo | Low |
| RDS (db.t3.micro) | ~$15/mo | $100-500/mo | Medium |
| DynamoDB | Pay per use | Variable | Low |

### Other Services

| Service | Baseline | Notes |
|---------|----------|-------|
| S3 | ~$1-5/mo | Storage + requests |
| CloudFront | ~$5-20/mo | CDN |
| Cognito | Free < 50k MAU | Auth |
| SQS | ~$1-5/mo | Queuing |
| Secrets Manager | $0.40/secret/mo | Secrets |

## Cost Estimation Template

```yaml
aws_cost_estimate:
  candidate_id: "<id>"
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
    total: "<$X>"
  cost_drivers:
    - "<primary cost driver>"
  ops_overhead: "<low|medium|high>"
  required_aws_knowledge: "<basic|intermediate|advanced>"
```

## Ops Overhead Factors

- **Low**: Fully managed (Lambda, Aurora Serverless, App Runner)
- **Medium**: Some config needed (ECS, RDS)
- **High**: Full management (EC2, self-managed DBs)
