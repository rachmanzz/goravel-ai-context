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

### 4. Providers (`app/providers/`)
- The central place to configure the application.
- Used for binding services into the Service Container.
- Registering event listeners, middleware, and routes.
- **Rule**: Use providers for dependency injection and framework extension.

### 5. Config (`config/`)
- Contains all application configuration.
- Values are typically pulled from `.env`.
- **Rule**: Never hardcode configuration values; use the `config` package.

## Implementation Pattern (Example)

### Route & Controller

```go
// routes/api.go
facades.Route().Get("/users/{id}", userController.Show)

// app/http/controllers/user_controller.go
func (r *UserController) Show(ctx http.Context) http.Response {
    id := ctx.Request().Input("id")
    var user models.User
    if err := facades.Orm().Query().Find(&user, id); err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": err.Error(),
        })
    }
    return ctx.Response().Success().Json(http.Json{
        "data": user,
    })
}
```

### Dependency Injection (Service Provider)

```go
// app/providers/app_service_provider.go
func (receiver *AppServiceProvider) Register(app foundation.Application) {
    app.Bind("userService", func(app foundation.Application) (any, error) {
        return &services.UserServiceImpl{}, nil
    })
}
```

By adhering to this structure, we ensure that the project remains scalable and easy to audit according to Goravel standards.
