# Utility Library Standards: Validation

This document defines the standards for data validation and utility library usage within the Goravel Project.

## Core Stack
- **Validation**: Goravel Built-in Validation
- **Integration**: `ctx.Request().Validate()` or Custom Form Requests

## Implementation Rules

### 1. Request Validation
- **Context-based**: Use `ctx.Request().Validate(rules)` for simple, inline validation within controllers.
- **Rule Mapping**: Define rules using Goravel's standard string-based format (e.g., `"required|email|max:255"`).

### 2. Form Requests (Recommended)
- **Separation of Concerns**: For complex validation logic, create dedicated Form Request classes.
- **Create**: `go run . artisan make:request <name>`
- **Usage**: Inject the Form Request into the controller method.

### 3. Validation Rules
- **Built-in Rules**: Leverage standard rules: `required`, `email`, `unique`, `min`, `max`, `confirmed`, etc.
- **Custom Rules**: Implement the `contractsvalidation.Rule` interface for domain-specific validation logic.
- **Rule Composition**: Rules can be combined using the pipe `|` character.

### 4. Error Handling
- **Fails Check**: Always check `validator.Fails()` before proceeding with business logic.
- **Response Format**: Return a consistent JSON response with validation errors:
  ```go
  if validator.Fails() {
      return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
          "errors": validator.Errors().All(),
      })
  }
  ```

### 5. Type Safety & Casting
- **Input Extraction**: Use `ctx.Request().Input("key")` or `ctx.Request().Bind(&struct)` to safely extract and cast validated data.
- **Struct Mapping**: Prefer binding to a struct for complex payloads to ensure type safety.

By adhering to these standards, we ensure that our data validation logic remains robust and consistent across the Goravel Project.
