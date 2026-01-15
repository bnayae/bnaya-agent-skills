Create a set of advisor agent-skills under .claude/skills/advisor.
All skills must be modular, composable, and professionally oriented toward software architecture.

⸻

General Principles
	•	Skills must be interactive and decision-driven.
	•	Prefer closed questions whenever possible to reduce ambiguity and speed up decision-making.
	•	Example: To understand expected load, ask “What is the expected throughput?” and provide predefined ranges (e.g., <100 msg/s, 100–1k msg/s, 1k–10k msg/s, >10k msg/s).
	•	Explicitly surface trade-offs (cost vs. reliability, simplicity vs. flexibility, etc.).
	•	Target senior / professional-level architecture discussions, not beginner guidance.
	•	Prefer structured integrations over free-form reasoning:
	•	Prefer MCPs when available.
	•	Otherwise prefer tools, APIs, or scripts.
	•	Free-text reasoning should be a fallback, not the default.
	•	Any external integration (MCP, tool, API, script) must be:
	•	Evaluated for safety and trust
	•	Scoped to a single responsibility
	•	Replaceable without breaking the skill contract

⸻

Skill 1 – Architecture Refinement

Trigger Conditions

This skill should be activated when the user:
	•	Mentions or requests:
	•	“refine architecture”
	•	“high-level design”
	•	“application architecture”
	•	“deployment strategy”
	•	“system design”
	•	Asks questions such as:
	•	“What architecture should I use?”
	•	“How should this system be structured?”
	•	“Help me design / restructure this service”
	•	Uses ambiguous or early-stage terms like:
	•	“choose stack”
	•	“what should I build this with”
	•	“is this architecture good”

⸻

Responsibility
	•	Accept a high-level or incomplete application architecture (verbal description, diagram, or assumptions).
	•	Drive clarification using structured, closed questions, focusing on:
	•	System boundaries
	•	Core components and responsibilities
	•	Data ownership and flow
	•	Expected load and growth
	•	Availability and DR expectations
	•	Operational and organizational constraints
	•	Avoid prescribing concrete technologies unless strictly required for clarification.

⸻

Output
	•	A refined, technology-agnostic high-level architecture.
	•	Clearly stated assumptions and constraints.
	•	This output acts as a formal input contract for downstream skills.

⸻

Skill 2 – Technology Stack & Cost-Aware Recommendations

Trigger Conditions

This skill should be activated when any of the following occur:
	•	The user explicitly asks to:
	•	“choose a tech stack”
	•	“recommend technologies”
	•	“compare AWS vs GCP vs Supabase”
	•	“optimize cost”
	•	“estimate monthly cost”
	•	The user mentions constraints such as:
	•	Budget limits
	•	SaaS vs self-hosted preference
	•	Cloud provider preference
	•	Immediately after Skill 1 completes successfully, using Skill 1’s output as its input contract.

⸻

Responsibility
	•	Consume the refined architecture produced by Skill 1.
	•	Propose multiple viable technology stack options, each evaluated across:
	•	Monthly cost (SaaS + cloud resources)
	•	Maintainability
	•	Elasticity and scalability
	•	Reliability and fault tolerance
	•	Observability
	•	Security posture
    •	Comparable via a decision matrix

⸻

Optimization Goals
	•	Optimize for:
	•	Cost efficiency
	•	Operational simplicity
	•	While balancing:
	•	Long-term maintainability
	•	Production readiness
	•	Non-functional requirements
	•	Explicitly explain why one option may be preferred over another under different priorities.

⸻

Sub-Skills (Separation of Concerns)

Skill 2 must be decomposed into specialized sub-skills, each with a distinct responsibility, for example:
	•	Next.js cost estimation
	•	AWS cost calculator
	•	GCP cost calculator
	•	Supabase cost evaluation

Sub-Skill Guidelines
	•	Each sub-skill:
	•	Focuses on one platform or concern only
	•	Has a clearly defined input and output
	•	Prefer MCP integrations when available.
	•	Otherwise use:
	•	Official calculators
	•	APIs
	•	Scripts
	•	Sub-skills must not reason about architecture decisions—only provide facts, estimates, or evaluations.

⸻

External Resources & Safety
	•	Evaluate provided links, tools, and scripts before use.
	•	Adopt only what fits the skill’s responsibility.
	•	Perform a basic safety, reliability, and maintenance assessment before integrating any external mechanism.