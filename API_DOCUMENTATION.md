# Member 2 — API Documentation
## Procurement Category Management & Department Management

Base URL: `http://localhost:8080/api/v1`

All successful responses are wrapped as:
```json
{ "success": true, "message": "...", "data": { ... }, "timestamp": "2026-07-21T10:15:30" }
```

---
## Category APIs — `/categories`

### Create — `POST /categories`
Request:
```json
{
  "categoryName": "IT Hardware",
  "categoryCode": "IT-HW",
  "description": "Laptops, servers, networking equipment",
  "active": true
}
```
Response `200`:
```json
{
  "success": true,
  "message": "Procurement category created successfully",
  "data": {
    "id": 1,
    "categoryName": "IT Hardware",
    "categoryCode": "IT-HW",
    "description": "Laptops, servers, networking equipment",
    "active": true,
    "supplierCount": 0,
    "createdAt": "2026-07-21T10:15:30",
    "updatedAt": "2026-07-21T10:15:30"
  }
}
```
Error `409` (duplicate name/code):
```json
{ "status": 409, "error": "Conflict", "message": "Procurement Category already exists with name: 'IT Hardware'" }
```

### Update — `PUT /categories/{id}`
Same body shape as create.

### Get one — `GET /categories/{id}`
### Get all — `GET /categories`

### Search — `GET /categories/search?keyword={text}&active={true|false}`
- `keyword` matches category name **or** code (partial, case-insensitive).
- `active` optionally filters to active/inactive only.
- Both params optional; omit both to behave like "get all".

Example: `GET /categories/search?keyword=hardware&active=true`

### Delete — `DELETE /categories/{id}`
- Returns `204 No Content` on success.
- Returns `422` if suppliers are still linked to this category:
```json
{ "status": 422, "error": "Unprocessable Entity",
  "message": "Cannot delete category 'IT Hardware' because it is linked to one or more suppliers. Reassign or remove those suppliers first." }
```

---
## Department APIs — `/departments`

### Create — `POST /departments`
Request:
```json
{
  "departmentName": "Information Technology",
  "departmentCode": "IT",
  "departmentHead": "Asha Rao",
  "description": "Handles all IT procurement",
  "active": true
}
```
Response `200`:
```json
{
  "success": true,
  "message": "Department created successfully",
  "data": {
    "id": 1,
    "departmentName": "Information Technology",
    "departmentCode": "IT",
    "departmentHead": "Asha Rao",
    "description": "Handles all IT procurement",
    "active": true,
    "approvalLevelCount": 0,
    "createdAt": "2026-07-21T10:15:30",
    "updatedAt": "2026-07-21T10:15:30"
  }
}
```

### Update — `PUT /departments/{id}`
### Get one — `GET /departments/{id}`
### Get all — `GET /departments`

### Search — `GET /departments/search?keyword={text}&active={true|false}`
Example: `GET /departments/search?keyword=IT`

### Delete — `DELETE /departments/{id}`
- Returns `204 No Content` on success.
- Returns `422` if the department still has approval-hierarchy levels defined:
```json
{ "status": 422, "error": "Unprocessable Entity",
  "message": "Cannot delete department 'Information Technology' because it has an active approval hierarchy defined. Remove approval levels first." }
```

---
## Validation Error Example (both modules)

`POST /categories` with a blank name:
```json
{
  "timestamp": "2026-07-21T10:15:30",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed for one or more fields",
  "path": "/api/v1/categories",
  "validationErrors": { "categoryName": "Category name is required" }
}
```

---
## Quick Reference Table

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/categories` | Create category |
| PUT | `/categories/{id}` | Update category |
| GET | `/categories/{id}` | Get category by id |
| GET | `/categories` | List all categories |
| GET | `/categories/search` | Search by keyword / active flag |
| DELETE | `/categories/{id}` | Delete category (blocked if suppliers linked) |
| POST | `/departments` | Create department |
| PUT | `/departments/{id}` | Update department |
| GET | `/departments/{id}` | Get department by id |
| GET | `/departments` | List all departments |
| GET | `/departments/search` | Search by keyword / active flag |
| DELETE | `/departments/{id}` | Delete department (blocked if approval levels linked) |
