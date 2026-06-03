# AI Roles

This document defines the roles and responsibilities for AI agents participating in the project.

## Role Types

### 1. Code Execution
- **Who**: The primary builder and implementer.
- **Responsibility**: Scaffolding, installation, code generation, and feature implementation.
- **Primary Guide**: Must follow the protocols defined in `.ai/context/entry.md`, `.ai/context/execution.md`, and `.ai/context/setup-flow.md`.
- **Primary Files**: Writes to `.ai/work/strategy.md`, `.ai/work/workflow.md`, and `.ai/work/decision-logs.md`.

### 2. Code Audit
- **Who**: The reviewer and quality gatekeeper.
- **Responsibility**: Reviewing code quality, security, and consistency.
- **Primary Guide**: Must follow the protocols defined in `.ai/work/audit.md`.
- **Constraint**: Does **NOT** write or modify production code.

## Separation of Concerns

To maintain an unbiased development process, each role should be handled by a different AI instance or tool:
- **Code Execution AI**: Dedicated to implementation and solving technical requirements.
- **Code Audit AI**: Dedicated to validation and identifying potential issues.

## Role Assignment

Final role assignments are documented in `.ai/clarification/ai-roles.md`. This file acts as the source of truth for which tool/model is assigned to which role.

## Agent File Generation

When an AI receives its role mandate, it must generate its own agent file at the project root (`/`) based on its assigned role in `ai-roles.md`.

### Code Execution Agent File
- **File Name**: Based on the tool (e.g., `/gemini.md`, `/agents.md`).
- **Content Requirements**:
    - **Role Declaration**: Explicitly state "This agent handles Code Execution".
    - **Reference**: Must point to `.ai/context/execution.md` as its primary execution protocol.
    - **Startup Sequence**: Include instructions to read `ai-roles.md`, `roles.md`, and `entry.md`.

### Code Audit Agent File
- **File Name**: Based on the tool (e.g., `/gemini.md`, `/agents.md`).
- **Content Requirements**:
    - **Role Declaration**: Explicitly state "This agent handles Code Audit".
    - **Reference**: Must point to `.ai/work/audit.md` as its primary review protocol.
    - **Startup Sequence**: Include instructions to read `ai-roles.md`, `roles.md`, and the target `strategy.md`.
    - **Boundaries**: Strictly prohibit any code modification commands.
