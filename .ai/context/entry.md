# Session Entry and Re-entry Protocol

This document defines how an AI agent should initialize its session and determine the project's current state.

## Context Loading Priority

When starting a session or resolving information conflicts, the following order of precedence applies (1 is highest priority):

1.  `.ai/clarification/ai-roles.md` (Who does what)
2.  `.ai/clarification/user-story.md` (What the user wants)
3.  `.ai/clarification/project-setup.md` (Technical setup answers)
4.  `.ai/clarification/project-learning.md` (Architecture & logic answers)
5.  `.ai/work/decision-logs.md` (History of technical choices)
6.  `.ai/work/api.docs.md` (API contracts)
7.  `.ai/work/workflow.md` (Execution sequence)
8.  `.ai/work/strategy.md` (Implementation plan)

## State Determination

Before taking any action, determine if the project is in **Setup Mode** or **Feature/Change Mode**.

### 1. Setup Mode (New Project)
If `.ai/clarification/project-setup.md` or `.ai/clarification/project-learning.md` are missing or incomplete:
- **Action**: Follow the protocol in `.ai/context/setup-flow.md`.

### 2. Feature/Change Mode (Existing Project)
If the project is already scaffolded and clarifications are complete:
- **Action**: Follow the **Re-entry Protocol** below.

## Re-entry Protocol

If you are tasked with adding a new feature, fixing a bug, or making changes to an existing project:

1.  **Skip Setup**: Do not repeat scaffolding, key generation, or basic installation steps.
2.  **Read History**: Read `decision-logs.md` to understand previous technical choices and avoid regressions.
3.  **Sync Context**: Read `workflow.md`, `api.docs.md`, and `user-story.md` to align with the overall project architecture and goals.
4.  **Create Strategy**: Generate a new entry in `strategy.md` specifically for the requested task.
5.  **Log Decision**: Write a "Pre-Execution Log" in `decision-logs.md` describing the final implementation plan.
6.  **Validate**: Wait for user approval of the strategy before writing any code.
7.  **Execute**: Implement the changes, following the standards in `.ai/context/execution.md`.
8.  **Verify & Log**: After implementation, verify the changes and write a "Post-Execution Log" in `decision-logs.md`.
