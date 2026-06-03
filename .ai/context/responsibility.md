# Responsibility and Engineering Standards

To ensure the performance, security, and reliability of the Project, all code must adhere to these core principles.

## Security and Integrity (Highest Priority)

Technical decisions must prioritize data protection and system integrity.

### Guidelines for Security:
- **Server-Side Security**: All sensitive operations must occur within the backend context. Never expose raw internal error stacks to the client.
- **Data Protection**: Cryptographic operations should use standard Go standard libraries (e.g., `crypto/` packages). Sensitive secrets must be managed via environment variables or secret vaults.
- **Input Validation**: **STRICT RULE**: Every request MUST be validated using Goravel's built-in validation before reaching the controller logic.

## Single Responsibility Principle (SRP)

Each module must have a single, well-defined purpose.

### Guidelines for SRP:
- **Core Logic Modules**: Dedicated exclusively to their specific domain logic (e.g., Services, Models). They should not be tightly coupled with the HTTP transport layer.
- **Middleware**: Focused on cross-cutting concerns (e.g., Auth, Logging, CORS). Use Goravel middleware and register them in the HTTP kernel.
- **Controllers**: Focused on processing requests and returning responses. Delegate complex logic to specialized service modules.

## Type Safety and Integrity

### Guidelines for Type Safety:
- **Strong Typing**: Leverage Go's strong type system for all data structures, request payloads, and API responses.
- **Context Handling**: Use Goravel's `http.Context` to safely access request data and manage response flow.

## Performance and Quality Standards

### Guidelines for Efficiency:
- **Binary Performance**: Write efficient Go code, leveraging goroutines for concurrent tasks when appropriate.
- **Efficient Caching**: Use Goravel's Cache facade for optimized data retrieval.
- **Database Optimization**: Utilize ORM features like Eager Loading to prevent N+1 query problems.

## Reusability and Portability

- **Decoupled Logic**: Keep core logic pure and decoupled from framework-specific APIs where possible (e.g., business logic should be framework-agnostic).
- **Composition over Inheritance**: Use functional composition and middleware chains to build flexible API pipelines.

By adhering to these standards, we ensure the Project is a trustworthy and efficient tool.
