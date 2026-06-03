# Transaction Skill

This skill defines database transaction and locking patterns in Goravel. Transactions ensure atomicity, consistency, and isolation for multi-step database operations.

## Ownership

Transactions belong exclusively in the **Service** layer:

```
Controller → Service (Transaction) → Model / Repository
```

Controllers must **never** open or manage transactions.

## Transaction Patterns

### Closure-Based (Recommended)

Auto-commits on success, auto-rollbacks on error:

```go
func (s *OrderServiceImpl) Create(items []CreateItemInput) (*models.Order, error) {
    var order models.Order

    err := facades.Orm().Transaction(func(tx orm.Query) error {
        if err := tx.Create(&order); err != nil {
            return err
        }
        for _, item := range items {
            if err := tx.Create(&models.OrderItem{
                OrderID: order.ID,
                ProductID: item.ProductID,
                Qty: item.Qty,
            }); err != nil {
                return err
            }
        }
        return nil
    })

    return &order, err
}
```

If the closure returns an error, the transaction is rolled back. If nil, it is committed.

### Manual Control

Use when you need conditional rollback decisions outside the closure scope:

```go
func (s *OrderServiceImpl) Create(input CreateOrderInput) (*models.Order, error) {
    tx, err := facades.Orm().Query().Begin()
    if err != nil {
        return nil, err
    }
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
            panic(r)
        }
    }()

    var order models.Order
    if err := tx.Create(&order); err != nil {
        tx.Rollback()
        return nil, err
    }

    if err := tx.Commit(); err != nil {
        return nil, err
    }
    return &order, nil
}
```

Always pair `Begin()` with `Commit()` or `Rollback()`. Use `defer` with `Rollback` for panic safety.

**WARNING**: Do not make external API calls (HTTP, payment gateway, email, SMS) between `Begin()` and `Commit()`. The transaction holds database connections and locks. If you need external calls based on the transaction result, commit first, then handle the external call — or use the [Transactional Outbox](#transactional-outbox) pattern for reliability.

## Locking Strategies

Use database locks to prevent race conditions when concurrent requests access the same rows.

### Pessimistic Locking

**FOR UPDATE** — Exclusive row lock. Other transactions cannot read (with FOR UPDATE) or modify locked rows until the transaction ends.

```go
func (s *InventoryServiceImpl) DeductStock(productID string, qty int) error {
    return facades.Orm().Transaction(func(tx orm.Query) error {
        var product models.Product
        if err := tx.Clause(clause.Locking{Strength: "UPDATE"}).
            Where("id", productID).
            First(&product); err != nil {
            return err
        }

        if product.Stock < qty {
            return fmt.Errorf("insufficient stock: have %d, need %d", product.Stock, qty)
        }

        product.Stock -= qty
        return tx.Save(&product)
    })
}
```

**FOR SHARE** — Shared lock. Other transactions can read the locked rows but cannot modify them.

```go
tx.Clause(clause.Locking{Strength: "SHARE"}).
    Where("id", productID).
    First(&product)
```

**SKIP LOCKED** — Skip rows locked by other transactions instead of waiting.

```go
func (s *JobServiceImpl) PickNext() (*models.Job, error) {
    var job models.Job
    err := facades.Orm().Transaction(func(tx orm.Query) error {
        return tx.Clause(clause.Locking{
            Strength: "UPDATE",
            Options:  "SKIP LOCKED",
        }).Where("status", "pending").
            Order("created_at ASC").
            Limit(1).
            First(&job)
    })
    if err != nil {
        return nil, err
    }
    return &job, nil
}
```

**NOWAIT** — Return error immediately if the row is already locked, instead of waiting.

```go
tx.Clause(clause.Locking{
    Strength: "UPDATE",
    Options:  "NOWAIT",
}).First(&product)
```

### Optimistic Locking

Use a version column. No lock is held; conflicts are detected at write time.

```go
type Product struct {
    orm.Model
    Name    string
    Stock   int
    Version int `gorm:"default:0"`
}

func (s *InventoryServiceImpl) DeductStock(productID string, qty int) error {
    return facades.Orm().Transaction(func(tx orm.Query) error {
        var product models.Product
        if err := tx.Where("id", productID).First(&product); err != nil {
            return err
        }

        if product.Stock < qty {
            return fmt.Errorf("insufficient stock")
        }

        result := tx.Where("id", productID).
            Where("version", product.Version).
            Updates(map[string]any{
                "stock":   product.Stock - qty,
                "version": product.Version + 1,
            })

        if result.RowsAffected == 0 {
            return fmt.Errorf("conflict: record was modified by another request")
        }
        return result.Error
    })
}
```

Use optimistic locking when contention is low and you prefer throughput over immediate conflict detection.

### Advisory Lock (PostgreSQL)

Application-level mutex scoped by key. Not tied to a specific row.

```go
func (s *PayoutServiceImpl) ProcessPayout(batchID string) error {
    lockKey := "payout:" + batchID
    return facades.Orm().Transaction(func(tx orm.Query) error {
        var acquired bool
        tx.Raw("SELECT pg_try_advisory_xact_lock(hashtext(?))", lockKey).Scan(&acquired)
        if !acquired {
            return fmt.Errorf("payout batch %s is already being processed", batchID)
        }
        // process payout
        return nil
    })
}
```

Use advisory locks for cross-row or cross-table coordination where row-level locking is insufficient.

## Selecting a Locking Strategy

| Scenario | Strategy | Reason |
|----------|----------|--------|
| Deduct inventory, balance update | FOR UPDATE | Prevents double-spend, concurrent writes |
| Job queue (worker picks next) | SKIP LOCKED | Workers don't fight over the same row |
| Low-contention counter | Optimistic (version) | No lock overhead, retry on conflict |
| Immediate user feedback | NOWAIT | Fail fast instead of blocking |
| Cross-row coordination | Advisory lock | Not tied to a single table row |

## Deadlock Handling

Deadlocks are possible when transactions lock rows in different orders.

### Prevention

Acquire locks in a consistent order across all code paths:

```go
// ALWAYS lock AccountA then AccountB
func (s *TransferServiceImpl) Transfer(fromID, toID string, amount float64) error {
    return facades.Orm().Transaction(func(tx orm.Query) error {
        // lock in ID order to prevent deadlock
        ids := []string{fromID, toID}
        sort.Strings(ids)

        var accounts []models.Account
        tx.Clause(clause.Locking{Strength: "UPDATE"}).
            Where("id IN ?", ids).
            Order("id ASC").
            Find(&accounts)
        // ...
    })
}
```

### Retry

Retry on deadlock error (PostgreSQL error code `40P01`):

```go
func (s *TransferServiceImpl) TransferWithRetry(fromID, toID string, amount float64) error {
    maxRetries := 3
    for i := 0; i < maxRetries; i++ {
        err := s.transfer(fromID, toID, amount)
        if err == nil {
            return nil
        }
        if !isDeadlock(err) {
            return err
        }
        time.Sleep(time.Duration(math.Pow(2, float64(i))) * 50 * time.Millisecond)
    }
    return fmt.Errorf("transfer failed after %d retries", maxRetries)
}

func isDeadlock(err error) bool {
    var pgErr *pgconn.PgError
    return errors.As(err, &pgErr) && pgErr.Code == "40P01"
}
```

Use exponential backoff between retries to reduce pressure on the database.

## Transaction Timeout

Set a timeout to prevent long-running transactions from holding locks:

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

err := facades.Orm().WithContext(ctx).Transaction(func(tx orm.Query) error {
    // slow operation will be cancelled after 5s
})
```

## Isolation Levels

PostgreSQL default is READ COMMITTED. Change only when the default does not guarantee correctness.

| Level | Behavior | When to Use |
|-------|----------|-------------|
| **READ COMMITTED** | Each statement sees only committed data before it executed. Default. | General-purpose API requests |
| **REPEATABLE READ** | Transaction sees a snapshot of data as of the first query. Prevents non-repeatable reads and phantom reads. | Reporting, read-only snapshots |
| **SERIALIZABLE** | Transactions execute as if serialized. Highest isolation, highest cost. | Financial transfers, critical balance operations |

```go
import "github.com/goravel/framework/contracts/database/orm"

func (s *TransferServiceImpl) Transfer(fromID, toID string, amount float64) error {
    return facades.Orm().Transaction(func(tx orm.Query) error {
        tx.Set("isolation_level", "serializable")
        // ...
    })
}
```

**Rule**: Start with READ COMMITTED. Only escalate when you have a proven race condition — higher isolation levels reduce throughput and increase deadlock probability.

## Transactional Outbox

When a transaction must be followed by side effects (email, push, webhook, Kafka), the outbox pattern prevents partial failures where the DB write succeeds but the side effect fails.

### Pattern

```text
1. Begin transaction
2. Write business data + Insert outbox event
3. Commit
4. Worker reads outbox events → publishes side effect → marks as sent
```

### Implementation

```go
// app/services/order_service.go
func (s *OrderServiceImpl) Create(input CreateOrderInput) (*models.Order, error) {
    var order models.Order

    err := facades.Orm().Transaction(func(tx orm.Query) error {
        if err := tx.Create(&order); err != nil {
            return err
        }
        // write side effect as data, not as actual I/O
        return tx.Create(&models.OutboxEvent{
            AggregateType: "order",
            AggregateID:   order.ID.String(),
            EventType:     "order.created",
            Payload:       input,
        })
    })
    if err != nil {
        return nil, err
    }
    return &order, nil
}
```

```go
// app/services/outbox_worker.go
func (s *OutboxWorkerImpl) Process() error {
    var event models.OutboxEvent
    err := facades.Orm().Transaction(func(tx orm.Query) error {
        // pick next unprocessed event
        if err := tx.Clause(clause.Locking{Strength: "UPDATE", Options: "SKIP LOCKED"}).
            Where("status", "pending").
            Limit(1).
            First(&event); err != nil {
            return err
        }
        return tx.Model(&event).Update("status", "processing")
    })
    if err != nil {
        return err
    }

    // outside transaction — actual I/O
    switch event.EventType {
    case "order.created":
        s.email.SendOrderConfirmation(event.AggregateID)
    }

    facades.Orm().Query().Model(&event).Update("status", "sent")
    return nil
}
```

This guarantees that side effects are at-least-once delivered. The worker retries on failure, and idempotency on the receiver handles duplicates.

## Transaction Decision Tree

```
Need atomic multi-step write?
├── No → No transaction
└── Yes
    ├── High contention / concurrent writes?
    │   ├── Yes → Pessimistic Lock (FOR UPDATE)
    │   └── No → Optimistic Lock (version column)
    │
    ├── Worker / job queue?
    │   └── SKIP LOCKED
    │
    ├── Immediate failure preferred over wait?
    │   └── NOWAIT
    │
    ├── Cross-table or cross-row coordination?
    │   └── Advisory Lock
    │
    └── Side effects after commit (email, webhook)?
        └── Transactional Outbox
```

## Best Practices

1. **Short-lived**: Keep transactions as short as possible. Do not make external API calls inside a transaction unless absolutely necessary.
2. **No HTTP in transactions**: Never access `http.Context`, call external HTTP endpoints, or perform I/O inside a transaction. The transaction holds DB connections and locks.
3. **Consistent lock order**: Always acquire locks in the same order (e.g., sorted IDs) to prevent deadlocks.
4. **Prefer closure-based**: Use `facades.Orm().Transaction(func(tx orm.Query) error { ... })` over manual `Begin/Commit/Rollback`. It handles rollback and panic recovery automatically.
5. **Error wrapping**: Return wrapped errors from transaction closures so the caller can distinguish business errors from DB errors.
6. **Read consistency**: Use FOR UPDATE only when the write depends on the current value read from the database. Common cases: stock deduction, balance transfer, quota consumption. Not all read-then-write scenarios need locking — e.g., reading a profile then updating a bio does not require FOR UPDATE because the bio update does not depend on the current value.

## Forbidden Patterns

- Controllers calling `facades.Orm().Transaction()`, `Begin()`, `Commit()`, or `Rollback()`
- External HTTP calls inside a transaction closure
- Sending emails, push notifications, or SMS inside a transaction
- Holding a transaction open while waiting for user input
- Using `SELECT ... FOR UPDATE` without a transaction
- Nested transactions without savepoint management (GORM does not support true nested transactions)
