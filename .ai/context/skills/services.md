# Services Skill

This skill defines the Service layer pattern in Goravel. Services encapsulate business logic and orchestration, keeping controllers thin and focused on HTTP concerns.

## Architecture

```
Controller → Service → Model / Repository
```

- **Controller**: Validates input, calls service, returns response
- **Service**: Business logic, facade calls, database operations, orchestration
- **Model**: Data structure, query scopes, relationships

## Repository Usage

Optional. Use Repository when:
- Query complexity is high
- Multiple services share the same queries
- Database access needs abstraction

Otherwise Service may access ORM directly via `facades.Orm()`.

## Mandatory Standards

### Controller Restrictions

Controllers must **NOT** contain:
- Business logic or domain rules
- Direct facade calls (`facades.Orm()`, `facades.Route()`, etc.)
- Database queries or raw SQL
- Orchestration of multiple models or services

**STRICT RULE**: If a controller method needs to call `facades.Orm()`, `facades.Cache()`, or any other facade, that logic must be inside a Service.

### Service Responsibilities

Services handle:
- Business logic and domain rules
- Database operations via facades or repositories
- Caching, queuing, and external API calls
- Transaction management (`facades.Orm().Transaction()`)
- Cross-model orchestration

## Service Location

Services live in `app/services/`.

### File Naming
- File: `app/services/user_service.go`
- Type: `UserServiceImpl`

```go
package services

type UserServiceImpl struct {
    // no state, stateless service
}

func (s *UserServiceImpl) GetByID(id string) (*models.User, error) {
    var user models.User
    err := facades.Orm().Query().Find(&user, id)
    if err != nil {
        return nil, err
    }
    return &user, nil
}
```

## Service Interface

Define interfaces for testability and loose coupling.

```go
// app/services/user_service.go
type UserService interface {
    GetByID(id string) (*models.User, error)
    Create(name, email string) (*models.User, error)
    Update(id string, data map[string]any) (*models.User, error)
    Delete(id string) error
}

type UserServiceImpl struct{}

func (s *UserServiceImpl) GetByID(id string) (*models.User, error) {
    // implementation
}
```

Controllers depend on the interface, not the concrete type:

```go
type UserController struct {
    userService services.UserService
}

func NewUserController(userService services.UserService) *UserController {
    return &UserController{userService: userService}
}

## Injection Strategy

**Default — Constructor Injection:**

```go
type UserController struct {
    userService services.UserService
}

func NewUserController(userService services.UserService) *UserController {
    return &UserController{userService: userService}
}
```

**Exception — Service Container Resolution:**

Use `facades.App().Make()` only for services requiring runtime initialization or external SDK bootstrapping (e.g., Firebase, AWS, Redis cluster).

```go
func (r *OrderController) SendNotification(ctx http.Context) http.Response {
    fb, err := services.FirebaseApp()
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Firebase unavailable",
        })
    }
    fb.SendPushNotification(...)
}
```

Rule: If the service can be constructed with static arguments, use constructor injection. If the service needs runtime configuration or SDK bootstrapping, use the resolver pattern.

## Provider Binding (Complex Initialization)

Services that require complex initialization (config, external SDK, connection pooling) must be bound to the service container and resolved via `facades.App().Make()`.

**When to bind**: The service needs setup beyond simple constructor params — e.g., Firebase SDK, AWS clients, Redis cluster.

### Provider Registration

```go
// app/providers/firebase_service_provider.go
package providers

import (
    "projectname/app/services"
    "github.com/goravel/framework/contracts/binding"
    "github.com/goravel/framework/contracts/foundation"
)

type FirebaseServiceProvider struct{}

func (r *FirebaseServiceProvider) Relationship() binding.Relationship {
    return binding.Relationship{
        Bindings: []string{
            services.FirebaseBinding,
        },
        Dependencies: []string{
            binding.Config,
        },
    }
}

func (r *FirebaseServiceProvider) Register(app foundation.Application) {
    app.Singleton(services.FirebaseBinding, func(app foundation.Application) (any, error) {
        return services.NewFirebase()
    })
}

func (r *FirebaseServiceProvider) Boot(app foundation.Application) {
    // Only implement Boot if the service needs to start during boot
    // e.g., health check, warm-up connections
}
```

### Resolver Helper

Provide a resolver function in the service package for clean access:

```go
// app/services/firebase.go
func FirebaseApp() (Firebase, error) {
    resolve, err := facades.App().Make(FirebaseBinding)
    if err != nil {
        return nil, err
    }
    app, ok := resolve.(Firebase)
    if !ok {
        return nil, fmt.Errorf("instance is not services.Firebase")
    }
    return app, nil
}
```

### Usage in Controller

```go
func (r *OrderController) SendNotification(ctx http.Context) http.Response {
    fb, err := services.FirebaseApp()
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Firebase unavailable",
        })
    }
    fb.SendPushNotification(...)
}
```

## Best Practices

1. **Stateless**: Services must not hold state. Forbidden patterns:

```go
// FORBIDDEN — holds state
type UserService struct {
    CurrentUser models.User
}

// FORBIDDEN — internal cache
type UserService struct {
    cache map[string]any
}
```

All dependencies are injected via constructor or resolved from the container at call time.
2. **Single Responsibility**: One service per domain aggregate (e.g., `UserService`, `OrderService`).
3. **Interface-First**: Define service interfaces in the same file as the implementation.
4. **Thin Controllers**: Controllers should remain thin. Large business logic belongs in Services — a 25-line controller can be fine; an 8-line controller full of business logic is not.
5. **Testability**: Services are testable without HTTP context — pure Go unit tests.

## Forbidden Patterns

The following patterns must never appear in the codebase:

- Controllers calling `facades.Orm()`
- Controllers calling `facades.Cache()`
- Controllers opening or managing transactions
- Services returning HTTP responses (e.g., `ctx.Response().Json(...)`)
- Services accessing `http.Context`
- Models calling external APIs or facades
