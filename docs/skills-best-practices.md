# Best Practices for Designing Agent Skills

Agent Skills are modular capabilities designed to extend Claude’s functionality for repetitive workflows, domain-specific tasks, and complex tool orchestrations.  

The architecture relies on progressive disclosure, meaning Claude loads information in stages (metadata first, instructions second, resources last) to save context tokens and reduce cost.  

## Optimization of Frontmatter (The "Trigger") 

The most critical component is the YAML frontmatter in SKILL.md, specifically the description field.

- **Be Specific**: The description must explain **what** the skill does and **when** to use it.
It should include specific keywords or phrases users might say (e.g., "Use when user mentions 'sprint' or 'Linear tasks'").
- **Avoid Generic Descriptions**: Descriptions like "Helps with projects" are insufficient and lead to under-triggering.
- **Negative Triggers**: If a skill triggers too often, explicitly state in the description when not to use it (e.g., "Do NOT use for simple data exploration").

## Structure for Progressive Disclosure

To keep the context window clean, leverage the three-level loading system:
- Level 1 (**Metadata**): Only the name and description load at startup (~100 tokens). 
This allows you to install many skills without context penalty.
- Level 2 (**Instructions**): The body of SKILL.md loads only when triggered. Keep this under 500 lines.
- Level 3 (**Resources**): Move **heavy documentation**, **API references**, or **templates**
 into subdirectories like references/ or assets/. 
 Claude reads these files via bash only when the instructions explicitly tell it to, keeping the main context free of unused data.

## Instruction Design (The "Recipe")

- **Deterministic Scripts**: For tasks requiring high reliability (like valid JSON output or specific file formatting), 
write a script (e.g., Python or Bash) in the scripts/ folder and instruct Claude to execute it. 
This is more reliable than asking the LLM to generate the code on the fly.
- **Degrees of Freedom**: Decide how much autonomy Claude needs. 
For strict tasks, use "low freedom" instructions (e.g., "Run this exact script"). 
For open-ended tasks (e.g., "Check for bugs"), allow higher freedom.
- **Error Handling**: Explicitly list common errors and their solutions within the skill instructions
 so Claude can self-correct without user intervention.

--------------------------------------------------------------------------------
## Guardrailing Skills in Claude Code

In Claude Code, you can enforce security and behavioral guardrails using configuration files and specific architectural patterns. 
This turns the agent from a "vibe-based" coder into a reliable engineering tool.

### CLAUDE.md: The Project "Constitution"

This file acts as the root context loaded at the start of every session. 
It is the primary place to set behavioral boundaries.

- High-Level **Guardrails**: Use CLAUDE.md to define "always-on" rules (e.g., "Always typecheck after changes," "Never commit without running tests").
- **Avoid "Never" without Alternatives**: Do not write negative-only constraints like "Never use flag X." Instead, provide the approved alternative: "Never use X, prefer Y." 
This prevents the agent from getting stuck.
- Pointer, Not Manual: Do not paste massive documentation here. Instead, point to the documentation: "For complex errors, see docs/troubleshooting.md." This keeps the context window efficient.
- **Memory Management**: Keep this file small (under ~500 lines) to prevent context bloat. 
If it grows too large, move specific workflows into on-demand Skills instead.

### Hooks: Deterministic Process Control

Hooks allow you to enforce rules programmatically 
by running scripts at specific lifecycle events (e.g., before a tool runs or before a prompt is sent).

- **Block-at-Submit** (Recommended): Use a PreToolUse hook to wrap commands like git commit. 
For example, configure a hook that checks for a "tests-passed" token or file. 
If the tests haven't passed, the hook rejects the commit command, forcing Claude to fix the code and retry.
- **Avoid Block-at-Write**: Avoid blocking the agent while it is writing or editing code ("Block-at-write"). 
Interrupting the agent mid-plan often confuses it. 
Let it finish the bad implementation, then block the commit/submission until it is fixed.

### settings.json: Environmental Sandboxing

This file controls the operational environment of the agent.

- **Network Guardrails**: Use HTTPS_PROXY or HTTP_PROXY to inspect traffic or sandbox the agent's network access, 
ensuring it isn't sending data to unauthorized endpoints.
- **Permission Auditing**: You can self-audit the permissions list in settings to verify which commands Claude is allowed to auto-run. 
While you can use --dangerously-skip-permissions for "vibe coding," 
enterprise or secure workflows should rely on strict permission sets.
- **Timeout Limits**: Adjust MCP_TOOL_TIMEOUT or BASH_MAX_TIMEOUT_MS to prevent the agent from hanging on long-running processes or to force it to fail fast if a script stalls.

### Skill-Level Restrictions

- **allowed-tools** Field: In the SKILL.md frontmatter, 
you can use the experimental allowed-tools field to specify a space-delimited list of pre-approved tools (e.g., Bash(git:*)). This restricts the skill to using only safe, necessary tools.
- **Script Isolation**: By encapsulating complex logic into scripts (e.g., scripts/validate.py) rather than asking Claude to perform the logic in chat, you ensure the operation is deterministic and follows exact code-defined constraints rather than probabilistic LLM behavior
