# Routing Standards Skill

This skill documents the mandatory standards for defining routes in Goravel to ensure maintainability and prevent deep nesting.

## Variable-Based Prefixing

Always capture top-level prefixes into variables. This avoids deep nesting of sub-groups and makes the routing structure flatter and more readable.

```go
// Good: Capturing the version prefix
v1Route := facades.Route().Prefix("/api/v1")
```

## Scoped Grouping

Use the prefix variables to create logical groups for specific features or modules. This keeps routes within a clear scope.

```go
// Defining a group with middleware
v1Route.Prefix("/auth").Middleware(baseMidd.Throttle("apis")).Group(func(router route.Router) {
    router.Post("/login", authController.Login)
    router.Post("/register", authController.Register)
})
```

## Mandatory Standards

1.  **Flattened Structure**: Avoid deep sub-grouping (group inside group inside group). Use the variable-based prefixing approach to keep the hierarchy shallow.
2.  **Single Scope**: Maintain routing within the same scope where possible to avoid confusion and redundant middleware declarations.
3.  **Variable Naming**: Use clear, descriptive names for prefix variables (e.g., `v1Route`, `adminRoute`, `publicRoute`).

## Benefits

-   **Reduces Complexity**: Prevents "callback hell" in routing files.
-   **Easier Refactoring**: Changing a version or base prefix only requires updating one variable.
-   **Better Organization**: Clearly separates different API versions or logical domains.
