# API Documentation

This document serves as the single source of truth for all API endpoints. Every route created or modified must be documented here following the template below.

[ do not delete this line, please put your api documentation down below]

---

## Template (Copy for new routes)

### `METHOD /path/to/endpoint`
**Description**: Brief explanation of what the endpoint does.

#### Request
- **Headers**: (e.g., Authorization: Bearer <token>)
- **Query Params**: (if any)
- **Body**: (JSON structure)
  - `key_name`: (Type) - Explanation (only if special treatment/validation logic is required).

#### Response
- **Success (200/201)**:
  ```json
  {
    "success": true,
    "data": { ... }
  }
  ```
- **Error (400/401/404/500)**:
  ```json
  {
    "success": false,
    "error": "Error message"
  }
  ```

---
