# Service Container Skill

This skill defines how to use the Goravel Service Container for binding and resolving services.

## Overview

The service container manages class dependencies and dependency injection. It contains all Goravel modules and lets you bind/resolve your own services.

Bindings are registered inside [Service Providers](./providers.md), but the container can also be accessed directly via `facades.App()`.

## Binding Methods

| Method | Behavior | Use Case |
|--------|----------|----------|
| `app.Bind(key, fn)` | New instance every resolve | Stateless services, helpers |
| `app.Singleton(key, fn)` | Same instance on subsequent calls | SDK clients, connections, config |
| `app.Instance(key, instance)` | Bind existing object directly | Pre-built instances, mocks |
| `app.BindWith(key, fn)` | Pass extra parameters to closure | Services needing runtime params |

### Bind

```go
app.Bind("goravel.route", func(app foundation.Application) (any, error) {
    return NewRoute(app.MakeConfig()), nil
})
```

New instance every time `app.Make("goravel.route")` is called.

### Singleton

```go
app.Singleton("google.firebase", func(app foundation.Application) (any, error) {
    return services.NewFirebase()
})
```

Same instance returned on subsequent calls. Recommended for external SDKs (Firebase, AWS, Redis).

### Instance

```go
app.Instance("myService", &MyService{Name: "cached"})
```

Binds an already-constructed object directly. Useful for tests or pre-built singletons.

### BindWith

```go
app.BindWith(Binding, func(app foundation.Application, params map[string]any) (any, error) {
    return NewRoute(app.MakeConfig(), params["id"]), nil
})
```

Extra parameters are passed as `map[string]any` when resolved via `MakeWith`.

## Resolving Methods

| Method | Description |
|--------|-------------|
| `app.Make(key)` | Resolve by binding key |
| `app.MakeWith(key, params)` | Resolve with extra parameters (for `BindWith`) |

### Make

```go
instance, err := app.Make("google.firebase")
if err != nil {
    // handle not found
}
firebase, ok := instance.(services.Firebase)
```

Always type-assert after resolving.

### MakeWith

```go
instance, err := app.MakeWith("goravel.route", map[string]any{"id": 1})
```

Used with bindings registered via `BindWith`.

## Accessing the Container

### Inside Service Providers

The container is available as the `app` parameter in `Register` and `Boot`:

```go
func (r *ServiceProvider) Register(app foundation.Application) {
    app.Singleton("key", func(app foundation.Application) (any, error) {
        return NewService(app.MakeConfig())
    })
}
```

### Outside Service Providers

Use the `App` facade:

```go
import "github.com/goravel/framework/facades"

instance, err := facades.App().Make("key")
```

This works anywhere — controllers, services, commands, etc.

## Convenience Methods

The framework provides shortcuts for resolving common facades:

- `app.MakeArtisan()`
- `app.MakeAuth()`
- `app.MakeCache()`
- `app.MakeConfig()`
- `app.MakeOrm()`
- `app.MakeQueue()`
- `app.MakeRoute()`
- etc.

These are equivalent to `app.Make("goravel.artisan")`, `app.Make("goravel.auth")`, etc.

## Best Practices

1. **Bind in providers, resolve in services** — keep binding logic inside Service Providers
2. **Always check error** — `Make` returns an error if the key is not bound
3. **Always type-assert** — `Make` returns `any`, cast to the expected interface
4. **Prefer Singleton for SDKs** — Firebase, Redis, DB clients should not be re-created
5. **Use Instance for tests** — swap real implementations with mocks via `app.Instance()`
6. **Bind keys as constants** — avoid string literals scattered across the codebase
