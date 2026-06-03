# AI Context — Goravel v1.17

## Cloning

To clone this project template:

```bash
npx degit rachmanzz/ai-context-goravel your-project
```

This project uses two AI roles: **Code Execution** (build/implement) and **Code Audit** (review/check).

Only **one** initial prompt is needed to set up both roles.

## Initial Prompt

```
Read .ai/context/roles.md

You are the role orchestrator. Based on roles.md, set up the following:

Code Execution → gemini-cli
Code Audit → opencode with OpenAI

Generate the required agent files at root for each role per the rules in roles.md.
Also generate .ai/clarification/ai-roles.md to document this assignment.

After all files are created, print instructions for each agent on how to proceed.
```

Replace `gemini-cli` and `opencode` with the actual tools being used.

## What Happens

| Step | Result |
|------|--------|
| 1 | `.ai/clarification/ai-roles.md` — role assignment documentation |
| 2 | Root agent file for Code Execution (e.g., `/gemini.md`) |
| 3 | Root agent file for Code Audit (e.g., `/agents.md`) |
| 4 | Instructions for each agent on how to proceed |

## Generated Files

| File | Content |
|------|---------|
| `/gemini.md` | Agent file for Code Execution — startup sequence, context files, rules |
| `/agents.md` | Agent file for Code Audit — audit process, boundaries |
| `.ai/clarification/ai-roles.md` | Tool → Role mapping |

Each agent file follows the structure defined in `.ai/context/roles.md`.

## After Agent Files Are Created

Run the **Code Execution** agent with this prompt:

```
Read ./<your-agent-file>.md and follow the startup sequence.
Then read .ai/context/setup-flow.md and execute it step by step.
```

Example for gemini-cli as Code Execution:

```
Read ./gemini.md and follow the startup sequence.
Then read .ai/context/setup-flow.md and execute it step by step.
```

The agent will follow `.ai/context/setup-flow.md` step by step:

1. **Ask questions** from `.ai/quest/setup.md` and `.ai/quest/learning.md` if clarification files don't exist yet
2. **Scaffold & install** — Goravel v1.17 (Go 1.23), then all dependencies
3. **Generate application key** using Artisan
4. **Setup AI roles** — generate `.ai/clarification/ai-roles.md` if missing
5. **Check API docs** — ask if you want to add `.ai/work/api.docs.md` first (optional)
6. **Generate workflow** — writes `.ai/work/workflow.md` with step-by-step execution plan
7. **Generate strategy** — writes `.ai/work/strategy.md` with implementation plan
8. **Wait for validation** — pauses and waits for your approval before writing any code
9. **Execute** — after validation, implements features (with mock data if API docs are not available)
10. **Audit** — Code Audit agent reviews the result using `.ai/work/audit.md`
