# AI Context — Goravel v1.17

## Cloning

To clone this project template:

```bash
npx degit rachmanzz/goravel-ai-context project-name
```

## Before Starting (Recommended)

To ensure the AI understands your specific requirements, it is highly recommended to create a **User Story** file:

- **File Path**: `.ai/clarification/user-story.md`
- **Content**: Describe the features, workflows, and business logic you want to implement.

Providing this file "if possible" will significantly improve the accuracy of the generated strategy and implementation.

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

## Starting a Session

After the agent files are created, start the **Code Execution** agent with this prompt:

```
Read ./<your-agent-file>.md and follow the startup sequence.
Then read .ai/context/entry.md and follow it.
```

The agent will automatically determine if it needs to perform a new setup or handle a new feature request based on the project state.

## Working on Existing Projects (Re-entry)

If the project setup is already complete and you want to start a new task (add a feature, fix a bug):

1.  **Prompt**:
    ```
    Read ./<your-agent-file>.md
    Read .ai/context/entry.md
    
    Task: [Describe your new feature or change here]
    ```
2.  **AI Response**: The AI will skip the setup phase, read your project history (`decision-logs.md`), sync with the `user-story.md`, and generate a new implementation strategy in `strategy.md`.
3.  **Review**: Validate the strategy before the AI starts coding.

## Setup Flow (Initial Only)

If the project is new, the agent will follow `.ai/context/setup-flow.md` step by step:

1. **Ask questions** from `.ai/quest/setup.md` and `.ai/quest/learning.md` (or pre-fill from `user-story.md`)
2. **Setup AI roles** — generate `.ai/clarification/ai-roles.md`
3. **Scaffold & install** — Goravel v1.17 (Go 1.23) and dependencies
4. **Generate application key** using Artisan
5. **Check API docs** — optional step to add `.ai/work/api.docs.md`
6. **Workflow & Strategy** — planning the implementation
7. **Validation** — wait for your approval
8. **Execute & Audit** — implement and review
