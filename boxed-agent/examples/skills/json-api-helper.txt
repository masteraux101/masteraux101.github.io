---
name: JSON/API Helper
description: Parse, transform, validate JSON data and help design or debug REST APIs
---

# JSON/API Helper Skill

**When to use:** "parse JSON", "API design", "REST endpoint", "format JSON", "JSON转换", "接口设计"

## Capabilities

### JSON Operations
- **Format/Pretty-print:** Beautify minified JSON
- **Validate:** Check for syntax errors with clear error messages
- **Transform:** Reshape JSON structure (flatten, nest, rename keys, filter)
- **Compare:** Diff two JSON objects highlighting additions, deletions, changes
- **Generate:** Create sample JSON from a schema or description

### API Design
- **Endpoint Design:** RESTful URL structure, HTTP methods, status codes
- **Request/Response Examples:** Complete curl commands with headers and body
- **Error Handling:** Consistent error response format
- **Authentication:** JWT, API key, OAuth patterns

## Output Format

### For JSON operations:
```json
{
  "formatted": "result"
}
```

### For API design:
```
### GET /api/v1/resources

**Description:** List all resources with pagination

**Query Parameters:**
| Param  | Type   | Required | Description       |
|--------|--------|----------|-------------------|
| page   | int    | No       | Page number (default: 1) |
| limit  | int    | No       | Items per page (default: 20) |

**Response 200:**
```json
{
  "data": [...],
  "meta": { "page": 1, "limit": 20, "total": 100 }
}
```

**curl Example:**
```bash
curl -X GET "https://api.example.com/v1/resources?page=1&limit=20" \
  -H "Authorization: Bearer TOKEN"
```
```

## Rules
- Always validate JSON before transforming
- Use consistent naming conventions (camelCase or snake_case, not mixed)
- Include error cases in API examples
- Follow REST conventions (plural nouns for collections, proper HTTP verbs)
