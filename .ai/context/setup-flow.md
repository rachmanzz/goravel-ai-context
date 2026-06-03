# Setup Flow

This document defines the step-by-step flow for setting up a new project. Follow these instructions in order.

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

## 1. Check Existing Clarification

Look inside `.ai/clarification/` for these files:
- `user-story.md` — optional but highly recommended user requirements
- `project-setup.md` — contains answers from `@.ai/quest/setup.md`
- `project-learning.md` — contains answers from `@.ai/quest/learning.md`

### If project-setup.md and project-learning.md exist AND contain complete answers:
- Skip all questions
- Read their contents and use them directly
- Proceed to step 3

### If either project-setup.md or project-learning.md is missing or incomplete:
- **First Step**: Check if `.ai/clarification/user-story.md` exists. If it does, read it carefully to understand the context and requirements. Use this information to pre-fill or answer as many questions as possible from the quests.
- Ask the user the remaining questions from `@.ai/quest/setup.md`.
- Ask the user the remaining questions from `@.ai/quest/learning.md`.
- Generate `./.ai/clarification/project-setup.md` with the final answers.
- Generate `./.ai/clarification/project-learning.md` with the final answers.
- Proceed to step 2

## 2. Validate Clarification

Confirm the generated files are saved correctly in `.ai/clarification/`.

## 3. Check Goravel Installation

Check if Goravel is already installed in the project:
- Look for `go.mod` and check for `github.com/goravel/framework` in `require`.
- Check for common Goravel files (e.g., `main.go`, `artisan`, `bootstrap/app.go`).

### If Goravel is NOT installed:
- Read `@.ai/context/skills/goravel-setup.md` for scaffolding instructions.
- Follow the guide to create a new Goravel project.

### If Goravel IS already installed:
- Skip scaffolding.
- Proceed to step 4.


## 4. Read Clarification Context

Read the following clarification files:
- `.ai/clarification/project-setup.md`
- `.ai/clarification/project-learning.md`

Extract:
- **Go Version** (e.g., 1.23)
- **Additional Libraries** list

## 5. Standardize Dependency Management

Ensure `go.mod` is initialized and tidy.

**Rules:**
- Use `go mod tidy` to manage dependencies.
- Ensure the Go version in `go.mod` matches the project requirement.

## 6. Install Additional Libraries

Read the `Additional Libraries` field from `.ai/clarification/project-setup.md`.

For each library listed:
- Install using `go get <library-path>`
- Example: `go get github.com/goravel/postgres`

## 7. Install Required Stack Libraries

Based on `project-learning.md`, install the agreed tech stack if not already present:

Common examples:
- Database drivers (e.g., `github.com/goravel/postgres`, `github.com/goravel/mysql`)
- Authentication (built-in Goravel Auth)
- Logging (built-in Goravel Log)

## 8. Verify Setup

- Confirm `go.mod` contains all expected dependencies.
- Confirm `.env` file exists and is configured (copy from `.env.example` if missing).
- Generate application key: `go run . artisan key:generate`.
- Run a dry build or dev server check (`go run .`) to ensure no errors.

## 9. Setup AI Roles

Check `.ai/clarification/ai-roles.md`:
- If it exists with complete role definitions, read and use it
- For each role listed, ensure the corresponding agent file exists

### If ai-roles.md is missing or incomplete:

Ask the user:
- "Which AI/tool handles **Code Execution**? (e.g., `opencode` with OpenAI, Claude CLI, etc.)"
- "Which AI/tool handles **Code Audit**? (e.g., `gemini-cli`, `opencode` with different model, etc.)"

Generate `.ai/clarification/ai-roles.md`:

```markdown
# AI Roles Clarification

## Code Execution
- **Tool:** {{code-execution-tool}}
- **Model/Provider:** {{code-execution-model}}

## Code Audit
- **Tool:** {{code-audit-tool}}
- **Model/Provider:** {{code-audit-model}}
```

**Note:** Agent files (`agents.md`, `gemini.md`, etc.) are NOT generated here. They are generated later when the AI receives its role mandate, following the templates in `.ai/context/roles.md`.

## 10. Check API Documentation

Check `.ai/work/api.docs.md`:
- If it contains real API documentation (content beyond the placeholder `[ do not delete this line... ]`), proceed to step 11
- If it is empty or only has the placeholder line:
  - Ask the user: **"Do you want to add API documentation first? You'll need to manually edit `.ai/work/api.docs.md`."**
  - **If yes**: Say "I will stop execution. After you fill in the API docs, type `continue execution` to continue." Then STOP — do not proceed further. Wait for the user to type `continue execution` before continuing.
  - **If no**: Proceed to step 11 without API docs. During implementation, create mock data/responses as needed.

## 11. Create Workflow & Strategy

Create or update these files:

### `.ai/work/workflow.md`
- Write a step-by-step execution plan based on the project learning context
- Each step must be clearly defined and validated before moving to the next

### `.ai/work/strategy.md`
- Document the implementation strategy based on facts (not assumptions)
- Reference `.ai/work/api.docs.md` if it exists with real content
- If no API docs exist, note that backend mocking will be used
- Reference `.ai/clarification/ai-roles.md` to determine which AI handles execution
- Wait for user validation of the strategy before executing

## 12. Execute Strategy

- After the user validates the strategy, implement the backend features and API endpoints
- If external services are not yet available, create mock data/responses using Goravel middleware or local state to demonstrate functionality
- Ensure all controllers, middleware, and data layers are properly connected
- Run final checks to confirm no errors were introduced

## Existing Project Changes

If the initial setup has already been completed and you are tasked with adding a new feature or making changes:

1.  **Skip Setup Phases**: Do not repeat scaffolding or basic installation steps.
2.  **Read History**: Read `decision-logs.md` to understand previous technical choices.
3.  **Sync Context**: Read `workflow.md` and `user-story.md` to align with the overall project goals.
4.  **Plan**: Generate a new entry in `strategy.md` specifically for the requested change.
5.  **Validation**: Wait for user approval of the new strategy before implementing.
6.  **Execute**: Follow the "Execute Strategy" protocol (Step 12) for the new task.
