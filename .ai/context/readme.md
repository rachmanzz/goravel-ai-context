# Project Context Map

This directory contains the foundational documents that define the identity, rules, and engineering standards for the Project.

## Context Files

### 1. [Authority & Boundaries](./authority.md)
- **Purpose**: Establishes the relationship between the AI and the User/Owner.
- **Key Concepts**: Respecting manual edits, API structure integrity, and proactive recommendations.

### 2. [Execution Workflow](./execution.md)
- **Purpose**: Mandates the protocols for implementation and development.
- **Key Concepts**: Strategy-first requirement, API Excellence priority, and validation protocols.

### 3. [Engineering Standards](./responsibility.md)
- **Purpose**: Defines high-level engineering principles and technical responsibilities.
- **Key Concepts**: Security Integrity, Single Responsibility Principle (SRP), and Type Safety.

### 4. [Project Structure & Architecture](./project-structures.md)
- **Purpose**: Defines the layered architecture (Services, Repositories, Routes, Handlers).
- **Key Concepts**: Separation of concerns and implementation patterns.

### 5. [Database Management](./database.md)
- **Purpose**: Standards for database integration, migrations, and Goravel ORM rules.
- **Key Concepts**: Migration locations, GORM-based conventions, and model encapsulation.

### 6. [Utility Library Standards (Validation)](./libraries/utilities.md)
- **Purpose**: Standards for data validation and schema definitions using Goravel's built-in validation.

### 7. [Roles & Responsibilities](./roles.md)
- **Purpose**: Defines the two AI roles (Code Execution and Code Audit) and their separation of concerns.
- **Key Concepts**: Role assignment, agent file generation, Code Audit constraints.

### 8. [Session Entry Protocol](./entry.md)
- **Purpose**: Defines how AI agents initialize sessions and determine project state.
- **Key Concepts**: Context loading priority, setup mode vs feature mode, re-entry protocol.

### 9. [Goravel Setup Skill](./skills/goravel-setup.md)
- **Purpose**: Procedure for scaffolding and configuring a Goravel project.

### 10. [Model Creation Skill](./skills/models.md)
- **Purpose**: Instructions for creating models using Artisan CLI.
- **Key Concepts**: Mandatory `--table` flag, GORM tags, JSONB serialization, model relations.

### 11. [Routing Standards Skill](./skills/route.md)
- **Purpose**: Standards for defining routes with variable-based prefixing.
- **Key Concepts**: Flattened structure, scoped grouping, descriptive variable naming.

### 12. [UUID Models Skill](./skills/uuid-models.md)
- **Purpose**: Standard implementation for models using UUID primary keys.
- **Key Concepts**: Local `Model`/`SoftModel` base structs, `BeforeCreate` hook, prohibition of `orm.Model`.

### 13. [Services Skill](./skills/services.md)
- **Purpose**: Defines the Service layer pattern for business logic and orchestration.
- **Key Concepts**: Thin controllers, service interfaces, dependency injection, transaction management.

### 14. [Providers Skill](./skills/providers.md)
- **Purpose**: Defines how to create, structure, and register Service Providers.
- **Key Concepts**: Relationship declaration, `Register`/`Boot` lifecycle, `Runners` interface, Singleton vs Bind.

### 15. [Service Container Skill](./skills/container.md)
- **Purpose**: Defines how to bind and resolve services in the container.
- **Key Concepts**: `Bind`/`Singleton`/`Instance`/`BindWith`, `Make`/`MakeWith`, access via `app` vs `facades.App()`.


---
## Related Work Files
- **[Strategy Buffer](../work/strategy.md)**: The required staging area for all implementation plans.
- **[Execution Workflow](../work/workflow.md)**: Step-by-step execution plan for complex tasks.
- **[API Documentation](../work/api.docs.md)**: Documentation for the API endpoints.
- **[Audit Checklist](../work/audit.md)**: Findings and reviews from the Code Audit agent.
