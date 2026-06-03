# Project Setup Quest

AI **must** ask the user these questions before scaffolding:

## Questions

### 1. Project Name
- Default: folder name or `module` from `go.mod` if available
- Ask: "What is the project name?"

### 2. Project Description
- Ask: "What is the project description?"

### 3. Go Version
- Default: `1.23` (Required for Goravel v1.17)
- Ask: "Which Go version to use? (Default 1.23)"

### 4. Database Driver
- Options: `postgres`, `mysql`, `sqlite`, `sqlserver`
- Ask: "Which database driver do you need?"

### 5. Additional Libraries
- Ask: "Any additional libraries needed? (e.g., grpc, redis, etc.)"
- Free text input

## Example User Clarification

```
Project Name: my-awesome-api
Project Description: A high-performance REST API using Goravel
Go Version: 1.23
Database Driver: postgres
Additional Libraries: grpc, redis
```

## Output

After user answers, AI creates `./.ai/clarification/project-setup.md` with the filled template:

```markdown
# Project Setup Clarification

- **Project Name:** {{project-name}}
- **Project Description:** {{project-description}}
- **Go Version:** {{go-version}}
- **Database Driver:** {{database-driver}}
- **Additional Libraries:** {{additional-libraries}}
```
