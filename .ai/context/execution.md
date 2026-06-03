# Execution and Implementation Workflow

This document defines the mandatory implementation protocol for the **Code Execution** role. It ensures a controlled, transparent, and high-quality development process.

## Strategy-First Requirement

Implementation must always be preceded by a documented strategy.

1.  **Mandatory Documentation**: All proposed changes must be written to `.ai/work/strategy.md`.
2.  **No Immediate Execution**: After documenting the strategy, the agent must stop and wait for explicit user validation to proceed.
3.  **Core Principles Check**: Every strategy must explicitly state how it maintains technical excellence (e.g., type safety, performance, security).

## Decision Logging

To ensure traceability, all significant technical decisions must be logged in `.ai/work/decision-logs.md`.

1.  **Pre-Execution Log**: After strategy approval but before coding, document the final plan and any last-minute technical choices.
2.  **Post-Execution Log**: After implementation and verification, record the outcome, any deviations from the strategy, and critical technical insights.

## Implementation Boundaries and Standards

### API Excellence & Reliability
- **STRICT RULE**: Prioritize high-quality, modern, and reliable API design.
- **API Documentation**: For every new route or modification, `.ai/work/api.docs.md` MUST be updated immediately (request body, response codes, error formats).

### Technical Standards
- **Stack Consistency**: Adhere to established technologies (Goravel, Facades, Service Providers).
- **Validation**: Ensure all inputs are validated via Goravel's validation system.
- **Security**: Sensitive data must never be logged or returned in API responses.
- **Integrity**: Confirm the application follows best practices for its target runtime (Go binary, Docker).

## Build and Distribution

- **Tooling**: Use `go build` for compilation and Artisan for management tasks.
- **Dependency Management**: **STRICT RULE**: Always use `go get` and `go mod tidy`.
- **Deployment**: Align changes with the established deployment pipeline.

## Task Completion

- **Cleanup**: Delete the strategy entry from `.ai/work/strategy.md` only after the task is successfully implemented, verified, and audited.
- **Verification**: Run all relevant tests and linters before marking a task as complete.
