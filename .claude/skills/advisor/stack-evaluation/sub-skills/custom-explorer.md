---
name: custom-explorer
description: Evaluate custom stack combinations proposed by the user. Use when user wants to explore alternatives not in generated candidates.
---

# Custom Combination Explorer

Evaluate user-proposed stack combinations that weren't in the generated candidates.

## When to Use

After presenting the decision matrix, if the user proposes a different combination.

## Evaluation Process

1. **Capture Components** - Document proposed stack
2. **Fit Assessment** - Evaluate against architecture brief
3. **Pros/Cons Analysis** - Compare to recommended candidates
4. **Cost Estimate** - Use relevant cost evaluators
5. **Local Dev Implications** - Assess local development story

## Output Contract

```yaml
custom_evaluation:
  proposed_components:
    frontend: "<tech>"
    backend: "<tech>"
    database: "<tech>"
    auth: "<tech>"
    hosting: "<platform>"
  fit_assessment:
    overall_score: "<strong|moderate|weak>"
    fits_budget: true|false
    fits_ops_maturity: true|false
    fits_compliance: true|false
    notes: "<explanation>"
  pros:
    - "<advantage>"
  cons:
    - "<disadvantage>"
  cost_estimate:
    baseline_monthly: "<$X>"
    at_10x_scale: "<$X>"
  local_dev:
    recommended_approach: "<approach>"
    complexity: "<low|medium|high>"
  comparison_to_recommended:
    better_at: []
    worse_at: []
    verdict: "<recommendation>"
```
