# Audit Checklist

Run this audit after each execution phase to verify quality and consistency.

## Code Quality
- [ ] No Go compilation errors (`go build ./...`)
- [ ] Code follows project formatting standards (`go fmt ./...`)
- [ ] All imports are correct and used
- [ ] Functions follow Single Responsibility Principle

## Dependencies
- [ ] No missing dependencies in `go.mod`
- [ ] No unused imports or variables
- [ ] All installed libraries are actually used

## API & Controllers
- [ ] Request validation using Goravel's validator is present
- [ ] Controllers return appropriate HTTP status codes (200, 201, 400, 404, 500, etc.)
- [ ] Response body follows established JSON schema
- [ ] Error handling is implemented (global or per-controller)
- [ ] Middleware (auth, logger, etc.) is correctly applied

## Data & Database
- [ ] Database logic is encapsulated in Models or Repositories
- [ ] Business logic is encapsulated in Services/Logic layer
- [ ] Database migrations are created/applied if schema changed
- [ ] Database results are correctly mapped to structs

## Build & Deploy
- [ ] Build passes without errors (`go build .`)
- [ ] Environment variables are properly documented in `.env.example`
- [ ] `config/` files are up to date and correctly configured

## File Structure
- [ ] Files follow `routes/`, `app/http/controllers/`, `app/models/` structure
- [ ] No dead code or commented-out blocks
- [ ] Related files are grouped in the same directory

AI may also add custom audit findings beyond this checklist. These notes can capture issues, improvements, or observations that become input for the next strategy iteration.

[ do not delete this line, mark items as done with `[x]` during audit]

## Custom Audit Notes

Use this section to document additional findings discovered during audit. These findings can inform the next strategy and workflow.

### Findings
- (Add findings here as needed)
