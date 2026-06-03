# Controller Skill

This skill defines the Controller layer pattern in Goravel. Controllers handle HTTP concerns only — validation, calling services, and returning responses.

## Architecture

```
Route → Middleware → Controller → Service → Model / Repository
```

- **Controller**: Validates input, calls service, returns response
- **Service**: Business logic, facade calls, database operations
- **Model**: Data structure, query scopes, relationships

## Location

Controllers live in `app/http/controllers/`.

### File Naming

- File: `app/http/controllers/user_controller.go`
- Type: `UserController`

```go
package controllers

import (
    "projectname/app/services"
    "github.com/goravel/framework/contracts/http"
)

type UserController struct {
    userService services.UserService
}

func NewUserController(userService services.UserService) *UserController {
    return &UserController{userService: userService}
}
```

## Controller Responsibilities

Controllers handle:

1. Extract request data (body, query, params)
2. Validate input
3. Call service methods
4. Map service results to HTTP responses
5. Return `http.Response`

Controllers must **NOT** contain:

- Business logic or domain rules
- Direct facade calls (`facades.Orm()`, `facades.Cache()`, etc.)
- Database queries or raw SQL
- Transaction management (`Begin`, `Commit`, `Rollback`)
- Orchestration of multiple models or services

## Request Validation

### Inline Validation

```go
func (r *UserController) Create(ctx http.Context) http.Response {
    var input CreateUserRequest
    if err := ctx.Request().Validate(&input); err != nil {
        return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
            "error": err.Error(),
        })
    }

    user, err := r.userService.Create(input.Name, input.Email)
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to create user",
        })
    }

    return ctx.Response().Success().Json(http.Json{
        "data": user,
    })
}
```

### Validation Rules

```go
type CreateUserRequest struct {
    Name  string `form:"name" json:"name" validate:"required|min_len:3|max_len:255"`
    Email string `form:"email" json:"email" validate:"required|email"`
}
```

Rules use Goravel's validation syntax: `required`, `email`, `min_len:N`, `max_len:N`, `numeric`, `in:A,B,C`, etc.

## Response Patterns

### Success

```go
return ctx.Response().Success().Json(http.Json{
    "data": user,
})
```

### Created (201)

```go
return ctx.Response().Json(http.StatusCreated, http.Json{
    "data": user,
})
```

### No Content (204)

```go
return ctx.Response().NoContent()
```

### Validation Error (422)

```go
return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
    "error": "Validation failed",
    "details": err.Error(),
})
```

### Not Found (404)

```go
return ctx.Response().Json(http.StatusNotFound, http.Json{
    "error": "User not found",
})
```

### Server Error (500)

```go
return ctx.Response().Json(http.StatusInternalServerError, http.Json{
    "error": "Internal server error",
})
```

Do not expose internal error details in 500 responses. Log them server-side.

### Paginated Response

```go
func (r *UserController) Index(ctx http.Context) http.Response {
    page := ctx.Request().QueryInt("page", 1)
    limit := ctx.Request().QueryInt("limit", 15)

    users, total, err := r.userService.Paginate(page, limit)
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to fetch users",
        })
    }

    return ctx.Response().Success().Json(http.Json{
        "data": users,
        "meta": http.Json{
            "page":       page,
            "limit":      limit,
            "total":      total,
            "total_page": (total + limit - 1) / limit,
        },
    })
}
```

## Dependency Injection

### Constructor Injection (Default)

```go
type UserController struct {
    userService services.UserService
}

func NewUserController(userService services.UserService) *UserController {
    return &UserController{userService: userService}
}
```

Wire in route registration:

```go
userController := controllers.NewUserController(services.NewUserServiceImpl())
v1Route.Prefix("/users").Group(func(router route.Router) {
    router.Get("/", userController.Index)
    router.Post("/", userController.Create)
    router.Get("/{id}", userController.Show)
    router.Put("/{id}", userController.Update)
    router.Delete("/{id}", userController.Delete)
})
```

### Container Resolution (Exception)

For services that need runtime initialization, use the resolver pattern from the service package:

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

Only use this for external SDKs (Firebase, AWS, Redis cluster). Default to constructor injection.

## CRUD Template

```go
package controllers

import (
    "projectname/app/services"
    "github.com/goravel/framework/contracts/http"
)

type UserController struct {
    userService services.UserService
}

func NewUserController(userService services.UserService) *UserController {
    return &UserController{userService: userService}
}

func (r *UserController) Index(ctx http.Context) http.Response {
    users, err := r.userService.All()
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to fetch users",
        })
    }
    return ctx.Response().Success().Json(http.Json{
        "data": users,
    })
}

func (r *UserController) Show(ctx http.Context) http.Response {
    user, err := r.userService.GetByID(ctx.Request().Input("id"))
    if err != nil {
        return ctx.Response().Json(http.StatusNotFound, http.Json{
            "error": "User not found",
        })
    }
    return ctx.Response().Success().Json(http.Json{
        "data": user,
    })
}

func (r *UserController) Create(ctx http.Context) http.Response {
    var input struct {
        Name  string `form:"name" json:"name" validate:"required|min_len:3"`
        Email string `form:"email" json:"email" validate:"required|email"`
    }
    if err := ctx.Request().Validate(&input); err != nil {
        return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
            "error": err.Error(),
        })
    }

    user, err := r.userService.Create(input.Name, input.Email)
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to create user",
        })
    }
    return ctx.Response().Json(http.StatusCreated, http.Json{
        "data": user,
    })
}

func (r *UserController) Update(ctx http.Context) http.Response {
    var input struct {
        Name  string `form:"name" json:"name" validate:"required|min_len:3"`
        Email string `form:"email" json:"email" validate:"required|email"`
    }
    if err := ctx.Request().Validate(&input); err != nil {
        return ctx.Response().Json(http.StatusUnprocessableEntity, http.Json{
            "error": err.Error(),
        })
    }

    user, err := r.userService.Update(ctx.Request().Input("id"), input.Name, input.Email)
    if err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to update user",
        })
    }
    return ctx.Response().Success().Json(http.Json{
        "data": user,
    })
}

func (r *UserController) Delete(ctx http.Context) http.Response {
    if err := r.userService.Delete(ctx.Request().Input("id")); err != nil {
        return ctx.Response().Json(http.StatusInternalServerError, http.Json{
            "error": "Failed to delete user",
        })
    }
    return ctx.Response().NoContent()
}
```

## Middleware Integration

Apply middleware at the route level, not inside the controller:

```go
v1Route.Prefix("/admin").Middleware(midd.Auth()).Group(func(router route.Router) {
    router.Get("/users", userController.Index)
})
```

Controllers should not check auth, check roles, or apply rate limiting — that is middleware's responsibility.

## Request Data Access

### Route Parameters

```go
ctx.Request().Input("id")
```

### Query Parameters

```go
ctx.Request().Query("search", "")
ctx.Request().QueryInt("page", 1)
ctx.Request().QueryArray("tags")
```

### Body (JSON/Form)

```go
var input CreateUserRequest
ctx.Request().Validate(&input)
```

### Headers

```go
ctx.Request().Header("Authorization")
```

## Best Practices

1. **Thin**: Each method should be 10-25 lines. If it grows, extract the logic into the Service layer.
2. **One method per action**: `Index`, `Show`, `Create`, `Update`, `Delete` — follow REST conventions.
3. **Consistent responses**: Use the same JSON envelope structure across all endpoints.
4. **No raw facade calls**: Every database, cache, or queue operation goes through a Service.
5. **No business logic**: Controllers orchestrate HTTP, not domain rules.
6. **Validate early**: Return 422 immediately if input is invalid — do not call the service.

## Forbidden Patterns

- Controllers calling `facades.Orm()`, `facades.Cache()`, `facades.Queue()`
- Controllers opening or managing transactions
- Controllers containing raw SQL or query builders
- Controllers with business logic (calculations, state machines, domain rules)
- Controllers accessing service internals or concrete types instead of interfaces
- Controllers performing external I/O (HTTP calls, file uploads directly)
- Controllers checking auth or roles (use middleware)
- Controllers holding state or caching data in struct fields
