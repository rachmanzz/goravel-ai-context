# Execution and Implementation Workflow

This document establishes the mandatory protocol for developing and maintaining the Project.

## Strategy-First Requirement

A comprehensive strategy must be documented before any implementation.

1.  **Mandatory Documentation**: All proposed changes must be written to `.ai/work/strategy.md`.
2.  **No Immediate Execution**: After documenting the strategy, stop and wait for a specific command to proceed.
3.  **Security & Technical Excellence Review**: Every strategy must explicitly state how it maintains core project principles and technical excellence (e.g., type safety, performance, security).

## Decision Logging

To ensure traceability and accountability, all significant technical decisions must be logged.

1.  **Pre-Execution Log**: Before starting implementation (after strategy approval), document the final plan and any last-minute technical decisions in `.ai/work/decision-logs.md`.
2.  **Post-Execution Log**: After implementation and verification, record the outcome, any deviations from the strategy, and critical technical insights in `.ai/work/decision-logs.md`.

## Implementation Boundaries and Standards

- **API Excellence & Reliability**: **STRICT RULE**: Prioritize high-quality, modern, and reliable API design. Every change should contribute to a performant, well-documented, and consistent backend experience.
- **API Documentation**: **STRICT RULE**: For every new route or modification to existing routes, the corresponding documentation in `.ai/work/api.docs.md` MUST be updated immediately. The documentation must include request body structure, all possible response codes, error responses, and detailed explanations for body keys that require special handling.
- **Technology Stack Consistency**: Adhere to the established technologies (Goravel, Facades, Service Providers) and APIs used in the project.
- **Integrity**: Ensure the app passes relevant audits and follows best practices for its target runtime (Go binary, Docker).
- **Validation**: 
    - Verify that sensitive data is never logged to the console or returned in API responses.
    - Confirm that every endpoint returns appropriate HTTP status codes and follows the established JSON response schema.
    - Ensure all inputs are validated via Goravel's validation system.

## Build and Distribution

- **Build Tooling**: Use `go build` for compilation and Artisan for management tasks.
- **Dependency Management**: **STRICT RULE**: Always use `go get` and `go mod tidy` for managing dependencies.
- **Deployment**: Follow the established deployment pipeline (e.g., Docker, Binary deployment).
- **Cleanup**: Delete the strategy entry from `.ai/work/strategy.md` once the task is successfully implemented and verified.

By following this workflow, we maintain a controlled, transparent, and high-quality development process.
