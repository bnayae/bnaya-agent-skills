---
name: aspire-evaluator
description: Evaluate whether .NET Aspire is a good fit for the stack candidate. Use when .NET is involved in stack-evaluation.
---

# Aspire Fit & Enablement Evaluator

Assess whether .NET Aspire is appropriate for the stack candidate.

## When Aspire Fits (Signals)

Strong fit indicators:
- Multi-service/distributed architecture where local orchestration is painful
- Need consistent wiring of service dependencies + connections
- Desire unified dev-time view of logs/traces/config
- Using .NET for at least some services
- Team wants code-first infrastructure definition

Weak fit indicators:
- Single service/monolith
- Non-.NET stack
- Simple dependencies (just a database)
- Team unfamiliar with .NET ecosystem

## What Aspire Provides

### AppHost
Code-first declaration of services and relationships for local orchestration.

### Dashboard
Dev-time visibility (OTLP-based): logs, traces, config view, resource health.

### MCP Enablement
Configure via `aspire mcp init` for AI-assisted development.

## Output Contract

```yaml
aspire_assessment:
  candidate_id: "<id>"
  fit_score: "<strong|moderate|weak|not_applicable>"
  signals_present:
    - signal: "<signal name>"
      present: true|false
  benefits:
    - "<benefit>"
  considerations:
    - "<consideration>"
  recommendation: "<use_aspire|consider_aspire|skip_aspire>"
```

## Fit Score Criteria

| Score | Criteria |
|-------|----------|
| **strong** | 3+ services, .NET backend, complex local orchestration needed |
| **moderate** | 2 services, .NET backend, would benefit from dashboard |
| **weak** | Single service or non-.NET, limited benefit |
| **not_applicable** | No .NET components |
