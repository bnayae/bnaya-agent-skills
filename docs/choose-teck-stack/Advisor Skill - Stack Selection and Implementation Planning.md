Advisor Skills – Stack Selection & Implementation Planning

Location: .claude/skills/advisor
Type: Advisor / Planner (Non-executing)

Goal

Architecture → evaluated stack options → user-selected final stack → implementation plan (executed later by other skills), with local dev support baked into both evaluation and planning.

⸻

General Principles
	•	Skills must be modular, composable, single-responsibility.
	•	Skills must be interactive and decision-driven; prefer closed questions with ranges/enums.
	•	Make trade-offs explicit (cost vs reliability, speed vs control, managed vs self-hosted).
	•	Prefer structured integrations over free-form reasoning:
	•	Prefer MCPs when available
	•	Else tools, APIs, or scripts
	•	External integrations must be safe/trustworthy, replaceable, and scoped to one responsibility.
	•	Avoid framework bias; stacks are derived from constraints.
	•	Local developer experience is a first-class requirement:
	•	Reproducibility, time-to-first-run, dependency orchestration, debuggability, and prod parity.
	•	Assume Docker Desktop exists and local Kubernetes exists.

⸻

Skill Topology

[Skill 1] Architecture Refinement
        ↓
[Skill 2] Stack Options & Evaluation (Orchestrator)
        ↓
[Skill 3] Final Stack Selection Confirmation
        ↓
[Skill 4] Implementation Plan Generator (includes Local Dev + CI/CD)
        ↓
[Skill 5] Skill Improvement PR Suggestor (Learning Loop)


⸻

Skill 1 – Architecture Refinement

Trigger

Architecture/system design/deployment strategy/choose stack/high-level design, etc.

Responsibility

Produce a decision-ready architecture brief (workload, constraints, quality attributes, assumptions). No forced tech choices.

Output Contract

architecture:
  domains: []
  data_flows: []
  workload_profile:
    throughput_range:
    latency_sensitivity:
    traffic_shape:
  constraints:
    budget_range:
    cloud_preference:
    compliance:
    team_size:
    ops_maturity:
  quality_attributes:
    availability_target:
    rto:
    rpo:
  local_dev_expectations:
    docker_desktop: true
    local_k8s: true
    dev_speed_priority: low|medium|high
  assumptions: []


⸻

Skill 2 – Stack Options & Evaluation (Orchestrator)

Trigger

User asks for stack selection/cost comparison, or automatically after Skill 1 completes.

Responsibility

Generate 2–5 coherent stack candidates, evaluate them, and present a comparable matrix. The user chooses the final stack.

Evaluation Factors (includes Local Dev)

Each candidate must be evaluated across:
	•	Monthly cost (baseline + scale curve)
	•	Maintainability / ops overhead
	•	Elasticity
	•	Reliability & DR posture
	•	Observability
	•	Security posture
	•	Vendor lock-in
	•	Local Dev DX (mandatory)
	•	time-to-first-run
	•	local dependency orchestration complexity
	•	prod parity (auth/data/async/config)
	•	local debugging + dev-time observability

⸻

Skill 2 – Sub-Skills

2.A Candidate Pattern Generator

Candidates may include (examples):
	•	React + AWS Amplify + Aurora Serverless
	•	React + AWS App Runner + RDS/Aurora
	•	React + Supabase
	•	React + Firebase
	•	React + Cloud Run + Cloud SQL
	•	Next.js + Vercel

Each candidate must include a local dev story (“how does a dev run this end-to-end on a laptop?”).

⸻

2.B Decision Matrix Sub-Skill

Add Local Dev DX as a scored and weightable criterion:

decision_matrix:
  criteria_weights:
    cost:
    ops:
    reliability:
    scalability:
    security:
    lock_in:
    local_dev_dx:
  options: []


⸻

2.C Cloud Provider Cost & Ops Sub-Skills

Fact providers:
	•	AWS Cost & Ops
	•	GCP Cost & Ops
	•	Supabase Cost
	•	Firebase Cost
	•	Vercel Cost

⸻

2.D Local Dev Experience Evaluator (New)

Responsibility: For each candidate, recommend the best local run approach given Docker Desktop + local K8s:
	•	docker-compose only
	•	local-k8s only
	•	hybrid (compose for deps + k8s for services)
	•	Aspire AppHost orchestration (when it fits)  ￼

⸻

2.E Aspire Fit & Enablement (New)

When it fits (signals):
	•	Multi-service/distributed architecture where local orchestration is painful
	•	Need consistent wiring of service dependencies + connections
	•	Desire a unified dev-time view of logs/traces/config

What it provides:
	•	AppHost: code-first declaration of services/relationships for local orchestration  ￼
	•	Dashboard: dev-time visibility into logs/traces/config, OTLP-based  ￼
	•	MCP enablement via aspire mcp init (recommended)  ￼

⸻

2.F Custom Combination Explorer

After the matrix, users can propose other combinations; return:
	•	fit assessment vs architecture
	•	pros/cons
	•	updated provider cost estimates
	•	local dev implications

⸻

Skill 3 – Final Stack Selection Confirmation

Trigger

After Skill 2 results (and optional exploration)

Output

selected_stack:
  stack_id:
  accepted_tradeoffs: []
  decision_notes: []


⸻

Skill 4 – Implementation Plan Generator

Trigger

Immediately after Skill 3 outputs selected_stack.

Responsibility

Produce a deterministic, execution-ready plan including local dev runbook + CI/CD. No execution.

Output Contract

implementation_plan:
  stack_id:
  assumptions: []
  phases:
    - name:
      goals: []
      steps: []

Required Plan Phases

1) Foundation

Accounts/projects, env model (dev/staging/prod), regions, DNS.

2) Networking & Security Baseline

IAM/access model, secrets management, network boundaries, secure-by-default policies.

3) Data Layer Setup

DB provisioning, backups/retention, migrations strategy, HA/replication.

4) Local Development Environment (Mandatory)

Must define how to run the solution locally assuming:
	•	Docker Desktop available
	•	Local Kubernetes available

Include:
	•	Local orchestration mode:
	•	docker-compose for dependencies (DB/cache/queue/object storage)
	•	k8s manifests/helm for services/deps
	•	hybrid approach
	•	Aspire AppHost (when chosen)  ￼
	•	Dependencies runbook:
	•	DB in local docker (ports/volumes/health checks)
	•	seeding/migrations workflow
	•	local secrets/config strategy
	•	Local observability wiring (logs/traces/metrics)
	•	Parity notes (what differs from prod and why)
	•	Aspire specifics (when selected):
	•	Define services + dependencies in AppHost  ￼
	•	Use Aspire dashboard for dev telemetry view (OTLP)  ￼
	•	Configure Aspire MCP via aspire mcp init  ￼

5) CI/CD (First-Class, Mandatory)

Repo strategy, branching, CI stages (build/test/lint/security scans), artifacts, CD per env, secrets injection, gates, rollout/rollback, preview envs.

6) Application Delivery

Scaffolding/templates, runtime config, env var contracts, deployment topology.

7) Observability

Dashboards, alerts, ownership.

8) Cost Controls

Budgets/alerts, guardrails, tagging/attribution.

9) Reliability Posture

DR strategy, failover, RTO/RPO alignment.

10) Hardening Checklist

Security hardening, operational readiness, runbooks.

⸻

Skill 5 – Skill Improvement PR Suggestor (Learning Loop)

Trigger

End of flow or repeated friction/corrections.

Responsibility

Propose PR-style improvements to the skill set based on interaction evidence.

pr_suggestion:
  title:
  motivation:
  evidence: []
  proposed_changes:
    - file:
      change:
  acceptance_criteria: []

