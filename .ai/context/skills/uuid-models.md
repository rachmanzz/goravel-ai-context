# UUID Models Skill

This skill documents the standard implementation for models using UUIDs as primary keys, replacing the default auto-incrementing integer IDs.

## Base Implementation

The base models are located in `app/models/model.go`. These structs should be embedded in your actual models instead of `orm.Model`.

```go
package models

import (
	"github.com/google/uuid"
	"github.com/goravel/framework/database/orm"
	"gorm.io/gorm"
)

// Model base for UUID primary keys
type Model struct {
	orm.Timestamps
	ID uuid.UUID `gorm:"primaryKey;autoIncrement:false" json:"id"`
}

func (m *Model) BeforeCreate(tx *gorm.DB) (err error) {
	if m.ID == uuid.Nil {
		m.ID = uuid.New()
	}
	return
}

// SoftModel base for UUID primary keys with Soft Deletes
type SoftModel struct {
	orm.SoftDeletes
	orm.Timestamps
	ID uuid.UUID `gorm:"primaryKey;autoIncrement:false" json:"id"`
}

func (m *SoftModel) BeforeCreate(tx *gorm.DB) (err error) {
	if m.ID == uuid.Nil {
		m.ID = uuid.New()
	}
	return
}
```

## Mandatory Standards

- **Prohibited Usage**: **NEVER** use `orm.Model` from `github.com/goravel/framework/database/orm` when working with UUID models.
- **Embedding**: Always embed `models.Model` or `models.SoftModel` from the local `models` package.
- **ID Generation**: The `BeforeCreate` hook ensures a new UUID is generated automatically if one isn't provided.

## Usage Example

```go
package models

type Product struct {
	Model // Embedding the local UUID base model
	Name  string
	Price float64
}

func (p *Product) TableName() string {
	return "products"
}
```
