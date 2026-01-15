---
name: stack-evaluation
description: Generate and evaluate stack candidates with a decision matrix. Use when user asks for stack selection, cost comparison, technology options, or after architecture-refinement completes.
---

# Stack Options & Evaluation (Orchestrator)

Generate 2-5 coherent stack candidates, evaluate them across multiple dimensions (including local dev experience), and present a comparable decision matrix.

## Prerequisites

Requires architecture brief from **architecture-refinement** skill. If not available, invoke that skill first.

## Process Overview

1. Generate stack candidates using [candidate-generator](sub-skills/candidate-generator.md)
2. Build decision matrix using [decision-matrix](sub-skills/decision-matrix.md)
3. Evaluate costs using provider-specific evaluators in [cost-evaluators/](cost-evaluators/)
4. Assess local dev experience using [local-dev-evaluator](sub-skills/local-dev-evaluator.md)
5. Check Aspire fit using [aspire-evaluator](sub-skills/aspire-evaluator.md) (when applicable)
6. Allow custom combinations via [custom-explorer](sub-skills/custom-explorer.md)

## Evaluation Factors

Each candidate must be evaluated across:
- Monthly cost (baseline + scale curve)
- Maintainability / ops overhead
- Elasticity
- Reliability & DR posture
- Observability
- Security posture
- Vendor lock-in
- **Local Dev DX** (mandatory):
  - Time-to-first-run
  - Local dependency orchestration complexity
  - Prod parity (auth/data/async/config)
  - Local debugging + dev-time observability

## Output Contract

```yaml
stack_evaluation:
  architecture_brief_id: "<reference>"
  candidates:
    - id: "<unique_id>"
      name: "<name>"
      summary: "<one-line>"
      components:
        frontend: { technology: "", hosting: "" }
        backend: { technology: "", hosting: "" }
        database: { technology: "", hosting: "" }
        auth: { provider: "" }
        async: { technology: "" }
        storage: { technology: "" }
      local_dev_story: "<how to run locally>"
      cost_estimate:
        baseline: "<$/month>"
        at_scale: "<$/month>"
      scores:
        cost: <1-5>
        ops_overhead: <1-5>
        reliability: <1-5>
        scalability: <1-5>
        security: <1-5>
        vendor_lock_in: <1-5>
        local_dev_dx: <1-5>
  decision_matrix:
    criteria_weights:
      cost: <1-5>
      ops_overhead: <1-5>
      reliability: <1-5>
      scalability: <1-5>
      security: <1-5>
      vendor_lock_in: <1-5>
      local_dev_dx: <1-5>
    ranked_options:
      - candidate_id: "<id>"
        weighted_score: <score>
        rank: 1
  recommendation:
    top_choice: "<candidate_id>"
    rationale: "<why>"
    tradeoffs: "<key tradeoffs>"
```

## Presentation

Present as comparison table:

| Criterion | Weight | Candidate A | Candidate B | Candidate C |
|-----------|--------|-------------|-------------|-------------|
| Cost | 4 | 4 ($$) | 3 ($$$) | 5 ($) |
| Ops Overhead | 3 | 5 (minimal) | 3 (moderate) | 4 (low) |
| Local Dev DX | 4 | 4 (good) | 3 (fair) | 5 (excellent) |
| **Weighted Total** | - | **X.X** | **X.X** | **X.X** |

## Next Step

After user reviews candidates, proceed to **stack-selection** skill to confirm the chosen stack.
