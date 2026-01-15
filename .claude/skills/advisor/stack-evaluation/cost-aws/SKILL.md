---
name: cost-aws
description: Estimate AWS infrastructure costs for any stack. Use when user asks about AWS pricing, cost estimation for EC2, Lambda, ECS, RDS, DynamoDB, CloudFront, Aurora, S3, or any AWS service combination.
---

# AWS Cost & Ops Evaluator

Estimate costs and operational overhead for AWS-based infrastructure.

## Standalone Usage

Can be invoked directly for cost estimation:
- "How much would ECS Fargate + Aurora cost?"
- "Estimate AWS costs for a Node.js API with PostgreSQL"
- "Compare Lambda vs App Runner pricing"

## Common AWS Services Pricing

### Compute

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Lambda | Free: 1M req/mo | $0.20/1M + duration | Low |
| App Runner | ~$25/mo (1 vCPU) | $50-200/mo | Low |
| ECS Fargate | ~$30/mo (0.5 vCPU) | $100-500/mo | Medium |
| EKS + Fargate | ~$70/mo + pods | $200-1000/mo | Medium |
| EKS + EC2 | ~$70/mo + EC2 | $200-2000/mo | High |
| EC2 (t3.micro) | ~$8/mo | $50-500/mo | High |
| EC2 (t3.small) | ~$15/mo | $100-1000/mo | High |

### Database

| Service | Baseline | At Scale | Ops Level |
|---------|----------|----------|-----------|
| Aurora Serverless v2 | ~$50/mo (0.5 ACU) | $200-2000/mo | Low |
| Aurora Provisioned | ~$30/mo (db.t3.small) | $150-1500/mo | Medium |
| RDS PostgreSQL | ~$15/mo (db.t3.micro) | $100-500/mo | Medium |
| RDS MySQL | ~$15/mo (db.t3.micro) | $100-500/mo | Medium |
| DynamoDB | Pay per use (~$1-10/mo) | Variable | Low |
| ElastiCache Redis | ~$15/mo (t3.micro) | $50-300/mo | Medium |

### Storage & CDN

| Service | Baseline | At Scale |
|---------|----------|----------|
| S3 Standard | ~$0.023/GB + requests | Variable |
| S3 (typical app) | ~$1-5/mo | $10-100/mo |
| CloudFront | ~$5-20/mo | $50-500/mo |
| EBS gp3 | ~$0.08/GB/mo | Variable |

### Other Services

| Service | Cost | Notes |
|---------|------|-------|
| Cognito | Free < 50k MAU | $0.0055/MAU after |
| SQS | ~$0.40/1M requests | Usually < $5/mo |
| SNS | ~$0.50/1M requests | Usually < $5/mo |
| Secrets Manager | $0.40/secret/mo | Per secret |
| Parameter Store | Free (standard) | Advanced: $0.05/param |
| ALB | ~$20/mo + LCU | Load balancer |
| NAT Gateway | ~$32/mo + data | Per AZ |
| Route 53 | $0.50/zone/mo | + queries |

## Example Calculations

### Basic API + Database
```
ECS Fargate (0.5 vCPU, 1GB): $15/mo
RDS PostgreSQL (db.t3.micro): $15/mo
ALB: $20/mo
S3 (10GB): $1/mo
CloudWatch: $5/mo
─────────────────────────────
Total: ~$56/mo baseline
```

### Serverless Stack
```
Lambda (1M requests): Free tier
API Gateway: ~$3.50/1M requests
Aurora Serverless (0.5 ACU min): $50/mo
S3 + CloudFront: $10/mo
Cognito (10k MAU): Free
─────────────────────────────
Total: ~$65/mo baseline
```

### K8s + Aurora + CloudFront
```
EKS cluster: $72/mo
Fargate pods (2x 0.5vCPU): $30/mo
Aurora Serverless v2: $50/mo
CloudFront: $15/mo
ALB: $20/mo
S3: $5/mo
Secrets Manager (5 secrets): $2/mo
─────────────────────────────
Total: ~$194/mo baseline
```

## Output Contract

```yaml
aws_cost_estimate:
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
    - "<secondary driver>"

  ops_overhead: "<low|medium|high>"
  required_aws_knowledge: "<basic|intermediate|advanced>"

  optimization_tips:
    - "<tip 1>"
    - "<tip 2>"
```

## Cost Optimization Tips

1. **Reserved Instances**: Save 30-60% on EC2/RDS with 1-3 year commitments
2. **Savings Plans**: Flexible compute savings (EC2, Fargate, Lambda)
3. **Spot Instances**: Up to 90% off for fault-tolerant workloads
4. **Aurora Serverless**: Scale to zero for dev environments
5. **S3 Lifecycle**: Move old data to cheaper tiers
6. **NAT Gateway**: Consider NAT instances or VPC endpoints
7. **Right-sizing**: Use Compute Optimizer recommendations
