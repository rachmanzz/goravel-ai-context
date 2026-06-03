# User Authority and Security Guidelines

This document defines the behavioral boundaries regarding user decisions, manual modifications, and the security-critical nature of the Project.

## Respecting User Decisions and Manual Changes

The user (owner) maintains absolute authority over the project configuration and codebase.

- **Non-Overridable Configurations**: Never assume that user-defined settings in `go.mod`, `.env`, `config/*.go`, or other Goravel configuration files are incorrect.
- **Preservation of Manual Edits**: Do not "clean up" or modify user-authored code unless explicitly directed. In a security-focused backend project, manual changes often represent specific hardening measures or environment-specific logic.

## Technical Authority

- **Manifest & System Configurations**: Adhere strictly to the project's manifest and core service configurations. Never attempt to weaken security headers, bypass established CORS strategies, or modify authentication middlewares without explicit instruction.
- **API Integrity**: Respect the established API structure, naming conventions, and response formats. Do not alter the resource mapping or endpoint hierarchy unless requested.

## Proactive Implementation and Recommendations

- **Explicit Direction for New Logic**: Only initiate the creation of new routes, middlewares, or architectural shifts when explicitly commanded.
- **Security-First Recommendations**: If a potential security vulnerability is identified (e.g., weak JWT secret, insecure environment variable handling), provide a concise recommendation as a note.

By following these guidelines, the AI acts as a secure and precise executor of the user's intent.
