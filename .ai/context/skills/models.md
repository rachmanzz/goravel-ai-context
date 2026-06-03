# Model Creation Skill

This skill provides instructions for creating and configuring models in a Goravel project using the Artisan CLI.

## Artisan Command

To create a new model, **always** include the `--table` flag to explicitly link it to its database table:

```bash
./artisan make:model --table=<table_name> <ModelName>
```

### Options

- `--table=<table>`: **MANDATORY**: Specify the exact table name for the model.
- `-m, --migration`: Create a new migration file for the model.
- `-c, --controller`: Create a new controller for the model.
- `-f, --force`: Force overwrite if the model already exists.
- `--factory`: Create a new factory for the model.
- `--seed`: Create a new seeder for the model.

**Example: Create model with specific table and force overwrite**
```bash
./artisan make:model --table=users -f User
```

**Example: Create model with migration, controller, and table reference**
```bash
./artisan make:model --table=products Product -mc
```

## Mandatory Standards

- **Table Reference**: **NEVER** use `make:model` without the `--table` flag. Explicitly defining the table ensures clarity and prevents issues with Goravel's default pluralization logic.

## Model Location

Models are typically generated in the `app/models` directory.

## Model Structure

A basic Goravel model uses GORM tags for database mapping:

```go
package models

import (
	"github.com/goravel/framework/database/orm"
)

type User struct {
	orm.Model
	Name  string
	Email string `gorm:"unique"`
}
```

## JSONB & Serialization

For storing dynamic JSON data in PostgreSQL using the `jsonb` type, use the GORM serializer:

```go
type Product struct {
	Model
	Name string
	Meta map[string]any `json:"meta" db:"meta" gorm:"serializer:json;type:jsonb"`
}
```

- `serializer:json`: Tells GORM to encode/decode the map to/from JSON when saving/reading.
- `type:jsonb`: Explicitly sets the database column type to `jsonb`.

## Model Relations

Relations are defined using struct fields and GORM tags to specify foreign keys.

### Has Many (Array)
Used when a model has multiple related records.

```go
type User struct {
	Model
	Name  string
	Posts []Post `json:"posts" gorm:"foreignKey:UserId"`
}
```

### Belongs To / Has One (Direct)
Used for 1:1 or the "child" side of a relationship.

```go
type Post struct {
	Model
	UserId uuid.UUID
	User   User `json:"user" gorm:"foreignKey:UserId"`
}
```

- `foreignKey`: In a **Has Many** relationship, it specifies the field name on the *related* model. In a **Belongs To** relationship, it specifies the field name on the *current* model.

## Best Practices

1. **Naming**: Use PascalCase for model names (e.g., `User`, `ProductCategory`).
2. **Embedding**: Always embed `orm.Model` (or `orm.SoftDeletes` if needed) to include standard fields like `ID`, `CreatedAt`, `UpdatedAt`, and `DeletedAt`.
3. **Table Names**: Even though GORM pluralizes by default, always define the `TableName` method for absolute clarity.

```go
func (u *User) TableName() string {
	return "users"
}
```
