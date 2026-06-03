# AI Roles

This project supports two distinct AI roles that handle different aspects of development.

## Role Types

### 1. Code Execution
- Handles all code generation, installation, scaffolding, and implementation
- Must follow `.ai/context/setup-flow.md` from the start
- Responsible for writing code, running commands, and building features
- Executes the strategy defined in `.ai/work/strategy.md`

### 2. Code Audit
- Reviews code quality, security, and consistency
- Runs checks from `.ai/work/audit.md`
- Provides findings that feed into the next strategy iteration
- Does NOT write or modify production code

## Separation of Concerns

Each role should be handled by a different AI instance/tool to maintain clear separation:
- **Code Execution AI** — focused on building and implementing
- **Code Audit AI** — focused on reviewing and validating

This ensures unbiased audits and focused execution.

## Role Clarification

AI roles are defined in `.ai/clarification/ai-roles.md`. This file specifies which AI/tool handles each role.

## Agent File Generation Rules

Agent files are **not** created during setup. They are generated **when the AI receives its role mandate**. The AI reads `.ai/clarification/ai-roles.md` to confirm its assigned role, then generates the appropriate agent file at root `/`.

### Code Execution Agent File

Generate at root `/` when assigned the Code Execution role. File name follows the tool name from `ai-roles.md`:
- Tool `opencode` → `/agents.md`
- Tool `gemini-cli` → `/gemini.md` (or `/gemini.md` if also used for audit, differentiate by purpose)
- Other tool → `/<tool-name>.md`

Must contain:
- **Role declaration** — confirms this agent handles Code Execution
- **Startup sequence** — mention `.ai/clarification/ai-roles.md`, `.ai/context/roles.md`, `.ai/context/setup-flow.md`
- **Context files reference** — list all `.ai/context/` files relevant to execution
- **Work files reference** — `.ai/work/strategy.md`, `.ai/work/workflow.md`, `.ai/work/api.docs.md`, `.ai/work/audit.md`
- **Rules** — strategy-first, user validation required, SRP, API consistency, type safety, post-execution audit

### Code Audit Agent File

Generate at root `/` when assigned the Code Audit role. File name follows the tool name from `ai-roles.md`:
- Tool `gemini-cli` → `/gemini.md`
- Tool `opencode` → `/agents.md`
- Other tool → `/tools-agents.md`

Must contain:
- **Role declaration** — confirms this agent handles Code Audit
- **Startup sequence** — mention `.ai/clarification/ai-roles.md`, `.ai/context/roles.md`, `.ai/work/strategy.md`, `.ai/work/audit.md`
- **Audit process** — type check, lint, review code quality, document findings in `.ai/work/audit.md`
- **Boundaries** — no source file modification, no dependency changes, no build/dev commands
- **Output** — summarize findings, suggest improvements for next strategy iteration
