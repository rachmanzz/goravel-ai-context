# Goravel Setup Skill

This skill provides instructions for scaffolding and configuring a Goravel project.

## Scaffolding

Use the Goravel installer to create a new project:

```bash
# Install the installer if not already present
go install github.com/goravel/installer/goravel@latest

# Create a new project
goravel new .
```

Alternatively, clone the skeleton repository:

```bash
git clone https://github.com/goravel/goravel.git .
go mod tidy
cp .env.example .env
go run . artisan key:generate
```

## Architectural Pattern

Goravel follows a **Service Provider** and **Facade** pattern, providing a clean and expressive API.

### Core Concept: Facades

Facades provide a static interface to classes that are available in the application's service container.

```go
import "github.com/goravel/framework/facades"

// Using the Route facade
facades.Route().Get("/", func(ctx http.Context) http.Response {
    return ctx.Response().Json(200, http.Json{
        "Hello": "Goravel",
    })
})
```

### Request Validation

Use Goravel's built-in validation in your controllers.

```go
func (r *UserController) Store(ctx http.Context) http.Response {
    validator, err := ctx.Request().Validate(map[string]string{
        "name": "required",
        "email": "required|email",
    })

    if err != nil {
        return ctx.Response().Json(http.StatusBadRequest, http.Json{
            "error": err.Error(),
        })
    }

    if validator.Fails() {
        return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
            "errors": validator.Errors().All(),
        })
    }

    // Process valid data...
}
```

## Core Dependencies

- `github.com/goravel/framework`
- Database drivers (e.g., `github.com/goravel/postgres`, `github.com/goravel/mysql`)
- `github.com/gorm-cpp/gorm` (underlying ORM)
