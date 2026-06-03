# Project Structure & Architecture

This document defines the architectural patterns and directory structure for the Goravel project. We follow the standard Goravel (Laravel-inspired) architecture to ensure separation of concerns, testability, and maintainability.

## Directory Overview

```text
app/
├── console/        # Artisan console commands
├── grpc/           # gRPC controllers and middleware
├── http/           # HTTP controllers and middleware
│   ├── controllers/# HTTP request handlers
│   ├── middleware/ # HTTP middleware
│   └── kernel.go   # HTTP kernel registration
├── models/         # ORM models (Gorm based)
├── services/       # Business logic and orchestration
└── providers/      # Service providers (Dependency Injection)
bootstrap/          # Application bootstrapping (app.go)
config/             # Configuration files
database/           # Migrations and seeders
routes/             # Route definitions (web.go, api.go)
tests/              # Automated tests
main.go             # Application entry point
```

## Layer Responsibilities

### 1. Routes (`routes/`)
- Define HTTP routes and map them to Controllers.
- Responsible for path definitions and assigning middleware.
- **Rule**: Do not put business logic here. Only route definitions.

### 2. Controllers (`app/http/controllers/`)
- Responsible for HTTP request/response handling.
- Validates input using Goravel's validation logic.
- Calls **Services** or **Models** to perform operations.
- Returns responses (JSON, View, etc.).
- **Rule**: Controllers should focus on request handling and response formatting.

### 3. Models (`app/models/`)
- Represents the data structure and database schema.
- Uses Goravel ORM (powered by Gorm) for data interactions.
- Can contain model-specific logic or hooks.
- **Rule**: Encapsulate database interactions within models or repositories.

### 4. Services (`app/services/`)
- Encapsulates business logic and domain rules.
- Handles database operations, caching, and external API calls.
- Orchestrates multiple models and manages transactions.
- **Rule**: Services must be stateless and depend on interfaces, not concrete types.
- **Rule**: Never access HTTP context (`http.Context`) inside services — keep them transport-agnostic.

### 5. Providers (`app/providers/`)
- The central place to configure the application.
- Used for binding services into the Service Container.
- Registering event listeners, middleware, and routes.
- **Rule**: Use providers for dependency injection and framework extension.

### 6. Config (`config/`)
- Contains all application configuration.
- Values are typically pulled from `.env`.
- **Rule**: Never hardcode configuration values; use the `config` package.

## Implementation Pattern (Example)

### Route & Controller (With Service Layer)

Controllers must delegate business logic and database operations to Services.

```go
// routes/api.go
facades.Route().Get("/users/{id}", userController.Show)

// app/http/controllers/user_controller.go
func (r *UserController) Show(ctx http.Context) http.Response {
    id := ctx.Request().Input("id")

    user, err := r.userService.GetByID(id)
    if err != nil {
        return ctx.Response().Json(http.StatusNotFound, http.Json{
            "error": "User not found",
        })
    }

    return ctx.Response().Success().Json(http.Json{
        "data": user,
    })
}

// app/services/user_service.go
func (s *UserServiceImpl) GetByID(id string) (*models.User, error) {
    var user models.User
    err := facades.Orm().Query().Find(&user, id)
    return &user, err
}
```

By adhering to this structure, we ensure that the project remains scalable and easy to audit according to Goravel standards.
