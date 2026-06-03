# Providers Skill

This skill defines how to create, structure, and register Service Providers in Goravel v1.17.

## Overview

Service Providers are the central place to configure the application. They bind services into the container, register event listeners, middleware, and routes. All providers are configured in `bootstrap/providers.go`.

The kernel calls `Register` on all providers first. After all providers are registered, it calls `Boot` on all providers.

## Create a Provider

Use Artisan to generate a provider:

```bash
go run . artisan make:provider FirebaseServiceProvider
```

Generated at `app/providers/firebase_service_provider.go`. The provider is automatically registered in `bootstrap/providers.go::Providers()`.

## Register a Provider

```go
// bootstrap/providers.go
package bootstrap

import (
    "github.com/goravel/framework/contracts/foundation"
    "projectname/app/providers"
)

func Providers() []foundation.ServiceProvider {
    return []foundation.ServiceProvider{
        ...
        &providers.FirebaseServiceProvider{},
        ...
    }
}

func Boot() contractsfoundation.Application {
    return foundation.Setup().
        WithProviders(Providers).
        WithConfig(config.Boot).
        Create()
}
```

Providers with `Relationship()` declared will be ordered by their dependencies. Providers without `Relationship()` are registered last.

## Provider Structure

Every provider has a `Register` method (mandatory) and `Boot` method (optional).

```go
package providers

import (
    "github.com/goravel/framework/contracts/binding"
    "github.com/goravel/framework/contracts/foundation"
)

type FirebaseServiceProvider struct{}

// Optional: declares binding keys and dependencies for registration ordering
func (r *FirebaseServiceProvider) Relationship() binding.Relationship {
    return binding.Relationship{
        Bindings: []string{
            services.FirebaseBinding,
        },
        Dependencies: []string{
            binding.Config,
        },
        ProvideFor: []string{
            // binding.Cache,  // if this provider is needed by Cache
        },
    }
}

// Mandatory: bind services into the container
// Should NOT register event listeners, routes, or other functionality here
func (r *FirebaseServiceProvider) Register(app foundation.Application) {
    app.Singleton(services.FirebaseBinding, func(app foundation.Application) (any, error) {
        return services.NewFirebase()
    })
}

// Optional: runs after ALL providers are registered
func (r *FirebaseServiceProvider) Boot(app foundation.Application) {
    // Register event listeners, routes, middleware here
}
```

### Relationship Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `Bindings` | Yes | List of binding keys this provider registers |
| `Dependencies` | No | List of services this provider depends on (e.g., `binding.Config`) |
| `ProvideFor` | No | List of services that depend on this provider's bindings |

Available dependency constants: `binding.Config`, `binding.Cache`, `binding.Route`, `binding.Queue`, `binding.Log`, `binding.Orm`, etc.

## Bind vs Singleton

| Method | Behavior |
|--------|----------|
| `app.Bind(key, fn)` | New instance every time `app.Make(key)` is called |
| `app.Singleton(key, fn)` | Same instance on subsequent calls (recommended for external SDKs) |

Use `Singleton` for services that manage connections, config, or shared state (Firebase, Redis, DB clients).

## Runners (v1.17)

Service providers can implement the `Runners` interface to start/shutdown services defined in the provider. Used for built-in services like Route, Schedule, Queue.

Each runner implements `ShouldRun()`, `Run()`, and `Shutdown()`:

```go
type Runner interface {
    ShouldRun() bool
    Run() error
    Shutdown() error
}
```

### Via ServiceProvider

Return runners from the provider:

```go
type ServiceProvider struct{}

func (r *ServiceProvider) Register(app foundation.Application) {}
func (r *ServiceProvider) Boot(app foundation.Application)    {}

func (r *ServiceProvider) Runners(app foundation.Application) []foundation.Runner {
    return []foundation.Runner{NewRouteRunner(app.MakeConfig(), app.MakeRoute())}
}
```

```go
// route/runner.go
package route

type RouteRunner struct {
    config config.Config
    route  route.Route
}

func NewRouteRunner(config config.Config, route route.Route) *RouteRunner {
    return &RouteRunner{config: config, route: route}
}

func (r *RouteRunner) ShouldRun() bool {
    return r.route != nil && r.config.GetString("http.default") != ""
}

func (r *RouteRunner) Run() error {
    return r.route.Run()
}

func (r *RouteRunner) Shutdown() error {
    return r.route.Shutdown()
}
```

### Via bootstrap/app.go

Register global runners that don't belong to a specific provider:

```go
// bootstrap/app.go
func Boot() contractsfoundation.Application {
    return foundation.Setup().
        WithProviders(Providers).
        WithConfig(config.Boot).
        WithRunners(func() []foundation.Runner{
            return []foundation.Runner{
                NewYourCustomRunner(),
            }
        }).
        Create()
}
```

**Note**: `Runners` are for framework-level services (Route, Schedule, Queue). For domain services, use `Register` to bind and `Boot` for warm-up if needed.

## Resolver Helper Pattern

Service providers go hand-in-hand with resolver helpers for clean access:

```go
// app/services/firebase.go
const FirebaseBinding = "google.firebase"

type Firebase interface {
    SendPushNotification(ctx context.Context, token string, title string, body string) error
    NewToken(ctx context.Context, uid string) (string, error)
}

// Resolver — hides container logic from callers
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

## Best Practices

1. **One file per provider** — name matches the concern (e.g., `firebase_service_provider.go`)
2. **Keep Register lean** — only bind services, no routes, events, or other functionality
3. **Use Boot for features** — register event listeners, routes, middleware in `Boot`, not `Register`
4. **Declare Relationships** — prevents registration order bugs
5. **Single responsibility** — one provider per domain service, not a monolithic provider
