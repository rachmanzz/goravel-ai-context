# Database Management Standards

This document defines the standards for database integration, schema management, and migrations within the Goravel Project.

## Library & Tooling Selection

Goravel uses a built-in ORM powered by **GORM**, providing a powerful and expressive interface for database interactions.

### 1. Goravel ORM (GORM Based)
- **Model Location**: `app/models/`.
- **Naming**: Models should use **PascalCase** (e.g., `User`, `OrderHistory`).
- **Encapsulation**: All database logic should be encapsulated within the models or a dedicated repository layer.
- **Modular Drivers**: Starting from v1.16, database drivers are separate packages. Ensure the correct driver is installed (e.g., `github.com/goravel/postgres`, `github.com/goravel/mysql`).

---

## Migration Management

Goravel uses a built-in migration system managed via **Artisan**.

### 1. Migration Location
- **STRICT RULE**: All migration files must be placed in `database/migrations/`.
- **Naming**: Artisan handles naming automatically with a timestamp (e.g., `20240602120000_create_users_table.go`).

### 2. Migration Commands
Use Artisan for migration management:
- **Create**: `go run . artisan make:migration <name>`
- **Run**: `go run . artisan migrate`
- **Rollback**: `go run . artisan migrate:rollback`
- **Status**: `go run . artisan migrate:status`

### 3. Schema Definition
Migrations are written in Go using the `facades.Schema()` builder.

```go
func (r *CreateUsersTable) Up() {
    facades.Schema().Create("users", func(table schema.Blueprint) {
        table.BigIncrements("id")
        table.String("name")
        table.String("email").Unique()
        table.Timestamp("email_verified_at").Nullable()
        table.String("password")
        table.RememberToken()
        table.Timestamps()
    })
}
```

---

## Seeding

Maintain seeders in `database/seeders/` for initial data or development environments.
- **Create**: `go run . artisan make:seeder <name>`
- **Run**: `go run . artisan db:seed`

## General Principles

- **Model Integrity**: Define proper tags (e.g., `gorm:"primaryKey"`) to ensure the ORM maps correctly to the database.
- **Transaction Safety**: Use `facades.Orm().Transaction()` for multi-step database operations to ensure atomicity.
- **Global Scopes**: Utilize Goravel's GlobalScope feature (v1.17+) for cross-cutting query concerns (e.g., Soft Deletes, Multi-tenancy).
