# Project Learning Quest

AI **must** ask these questions to deeply understand the project before generating a learning document.

## Questions

### 1. Project Overview
- What is the core purpose of this project?
- Who are the target users?
- What problem does it solve?

### 2. Tech Stack
- Framework: Goravel v1.17
- Language: Go 1.23+
- Database: PostgreSQL, MySQL, SQLite, or SQL Server?
- Cache: Redis, Memcached, or Local?
- ORM: Goravel ORM (GORM)

### 3. Architecture & Data Flow
- How does data flow within the app? (e.g., Route → Middleware → Controller → Service → Model)
- Authentication flow: JWT, Session, or Custom?
- Key architectural decisions: Service Providers, Facades, Dependency Injection.

### 4. Security & Privacy
- Secret management (Environment variables via `.env`).

### 5. API & Backend
- REST API or gRPC?
- Main endpoints and purposes.

### 6. Key Business Logic
- Core modules (e.g., Auth, Task Management, etc.).
- Complex validations or business rules.

### 7. Deployment & Infrastructure
- Platform: Docker, VPS, Fly.io, etc.?
- CI/CD setup?

## Example User Clarification

```
Project Overview: Backend API for a task management system
Tech Stack: Goravel v1.17, Go 1.23, PostgreSQL, Redis
Architecture: Route -> Auth Middleware -> Controller -> Service -> Model
Security: JWT tokens, secrets in .env
API: REST endpoints for task CRUD
Deployment: Docker on VPS
```

## Output

After user answers, AI creates `./.ai/clarification/project-learning.md` with the filled template:

```markdown
# Project Learning: [Project Name]

## Main Principles

### 1. ...
### 2. ...
### 3. ...

---

## Data Flow

### Endpoint A
```
flow diagram (Route -> Middleware -> Controller -> Service)
```

### Endpoint B
```
flow diagram
```

---

## Data Architecture

### Model Schema
```go
type User struct {
    gorm.Model
    Name  string
    Email string `gorm:"unique"`
}
```

---

## Key Business Logic

```
