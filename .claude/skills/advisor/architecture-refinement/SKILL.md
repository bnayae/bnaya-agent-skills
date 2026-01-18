---
description: Gather requirements and produce a decision-ready architecture brief for stack selection. Use when user mentions architecture, system design, deployment strategy, choose stack, tech stack selection, cost comparison, technology options, or high-level design.
name: architecture-refinement
---

# Architecture Refinement

Produce a decision-ready architecture brief covering workload, constraints, quality attributes, and assumptions. **Do not make technology choices** – this skill only gathers requirements.

## Process

### Step 1: Project Context

Ask the user:

1. **Project Overview**: What is being built? (e.g., SaaS platform, mobile backend, data pipeline).
2. **Domain Boundaries**: What are the main functional domains/capabilities?
3. **Data Flows**: How does data move through the system?

Start from the user prompt and the attached files or links.
When Figma is available use Figma MCP. if the MCP is not available instruct the user how to activate it.
If the prompt or the app spec (might be an attached md file doesn't answer all the questions, communicate with the user to get the missing answers).

### Step 2: Workload Profile

Use closed questions with ranges:

**Throughput Range**:

- Low: < 100 req/sec
- Medium: 100-1,000 req/sec
- High: 1,000-10,000 req/sec
- Very High: > 10,000 req/sec

**Latency Sensitivity**:

- Relaxed: > 1 second acceptable
- Standard: 100ms-1s acceptable
- Sensitive: 10ms-100ms required
- Critical: < 10ms required

**Traffic Shape**:

- Steady: predictable, consistent load
- Bursty: occasional spikes (2-5x baseline)
- Highly Variable: extreme spikes (10x+ baseline)
- Event-Driven: load tied to specific events

### Step 3: Constraints

**Budget Range** (monthly infrastructure):

- Minimal: < $100/month
- Seed: $100-$200/month
- Startup: $200-$1,000/month
- Growth: $1,000-$10,000/month
- Scale: $10,000-$100,000/month
- Enterprise: > $100,000/month

**Cloud Preference**

- AWS
- GCP
- Azure
- Vercel
- Supabase
- Mongo Atlas
- Multi-cloud
- No preference
- On-premises.

Allow multi choise like AWS or GCP.

**Compliance** (multi-select): None | SOC 2 | HIPAA | GDPR | PCI-DSS | FedRAMP | Other

**Team Size**: Solo (1) | Small (2-5) | Medium (6-15) | Large (16-50) | Enterprise (50+)

**Ops Maturity**:

- Low: Limited DevOps experience, prefer managed services
- Medium: Some DevOps capability, can manage some infrastructure
- High: Strong DevOps team, comfortable with self-managed solutions

### Step 4: Quality Attributes

**Availability Target**: 99% | 99.9% | 99.99% | 99.999%

**RTO** (Recovery Time Objective): Relaxed (24+ hours) | Standard (4-24h) | Fast (1-4h) | Critical (< 1h)

**RPO**: Relaxed (24+ hours) | Standard (1-24h) | Tight (1min-1h) | Zero

> maximum amount of data (measured in time) that an organization can afford to lose

### Step 5: Local Development

Confirm assumptions:

- Docker Desktop available: Yes/No
- Local Kubernetes available: Yes/No

**Dev Speed Priority**: Low | Medium | High

### Step 5.5: Offline Requirement

Use closed question with explicit semantics:

**Which level of offline capability is required?**

| Level | Description | Session Close Behavior | Sync Strategy |
|-------|-------------|------------------------|---------------|
| **None** | Continuous connectivity assumed | N/A | N/A |
| **Transient** | Short disconnections only, in-memory buffering | Data may be lost | None required |
| **Session-Durable** | Works across browser/app restarts | Persists locally | Background sync on reconnect |
| **Strong Offline-First** | Full functionality offline | Durable local storage | Guaranteed eventual sync (CRDT/merge) |

When clarifying this requirement, be explicit about:

- What happens if the session or app is closed during offline
- Whether offline support is only to smooth network instability
- Whether the client must reconcile and sync data later

### Step 5.6: Cost vs. Risk Comparison Preference

**Would you like multiple stack options reflecting different offline guarantees?**

| Option | Description |
|--------|-------------|
| **Single Best** | Recommend one best-fit stack matching the offline requirement |
| **Limited Comparison** | Compare minimal offline vs. durable offline (2 options) |
| **Full Spectrum** | Compare all levels: none → transient → session-durable → strong offline-first, with data-loss guarantees, sync complexity, and operational cost |

### Step 6: Document Assumptions

List assumptions driven from the givven promt/spec plus requirements made during the conversation.

## Output Contract

```yaml
architecture:
  project_name: "<name>"
  project_description: "<description>"
  domains:
    - name: "<domain>"
      description: "<description>"
      capabilities: []
  data_flows:
    - source: "<source>"
      destination: "<destination>"
      description: "<what flows>"
  workload_profile:
    throughput_range: "<low|medium|high|very_high>"
    latency_sensitivity: "<relaxed|standard|sensitive|critical>"
    traffic_shape: "<steady|bursty|highly_variable|event_driven>"
  constraints:
    budget_range: "<minimal|startup|growth|scale|enterprise>"
    cloud_preference: "<aws|gcp|azure|multi_cloud|no_preference|on_premises>"
    compliance: []
    team_size: "<solo|small|medium|large|enterprise>"
    ops_maturity: "<low|medium|high>"
  quality_attributes:
    availability_target: "<99|99.9|99.99|99.999>"
    rto: "<relaxed|standard|fast|critical>"
    rpo: "<relaxed|standard|tight|zero>"
  local_dev_expectations:
    docker_desktop: true
    local_k8s: true
    dev_speed_priority: "<low|medium|high>"
  offline_requirement:
    level: "<none|transient|session_durable|strong_offline_first>"
    session_close_behavior: "<loses_data|persists_locally|na>"
    sync_strategy: "<none|background_sync|manual_sync|crdt>"
  comparison_preference: "<single_best|limited_comparison|full_spectrum>"
  assumptions: []
```

## Next Step

After completing, proceed to **stack-evaluation** skill to generate and evaluate stack candidates.
