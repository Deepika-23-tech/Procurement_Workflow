# Member 2 — Module Documentation
## Procurement Category Management & Department Management

**Owner:** Member 2
**Modules:** Procurement Category Management, Department Management
**Parent Project:** Procurement Master Data Management System — Module 1

---

## 1. Scope of Work

| Area | Delivered |
|---|---|
| Category CRUD | Create, Read (single/all), Update, Delete |
| Department CRUD | Create, Read (single/all), Update, Delete |
| Uniqueness validation | Category name & code unique; Department name & code unique |
| Deletion guards | Category cannot be deleted if linked to suppliers; Department cannot be deleted if it has an approval hierarchy defined |
| Search APIs | Keyword search (name/code) and active-status filter, for both modules |
| DTOs | Request/Response DTOs, decoupled from entities |
| Validations | Bean Validation annotations on all request DTOs |
| Exception handling | Delegates to the shared `GlobalExceptionHandler` (409 for duplicates, 422 for business-rule violations, 404 for not found, 400 for validation failures) |

---

## 2. Architecture (Layered)

```
CategoryController / DepartmentController   → REST endpoints, request/response only
        │
CategoryService / DepartmentService         → interfaces (contracts)
        │
CategoryServiceImpl / DepartmentServiceImpl → business rules & validation logic
        │
ProcurementCategoryRepository / DepartmentRepository → Spring Data JPA
        │
ProcurementCategory / Department (Entities) → MySQL tables
```

Both modules follow the exact same pattern used across the rest of the system, so they
plug into the shared `GlobalExceptionHandler` and `ApiResponse<T>` wrapper without any
module-specific exception code.

---

## 3. Entity Design

### ProcurementCategory (`procurement_categories`)

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT | PK, auto-increment |
| category_name | VARCHAR(100) | NOT NULL, UNIQUE |
| category_code | VARCHAR(20) | NOT NULL, UNIQUE |
| description | VARCHAR(500) | nullable |
| is_active | BOOLEAN | NOT NULL, default true |
| created_at | DATETIME | auto-set on insert |
| updated_at | DATETIME | auto-set on update |

Relationship: `ProcurementCategory (1) ──< (M) Supplier` (via `Supplier.category_id`)

### Department (`departments`)

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT | PK, auto-increment |
| department_name | VARCHAR(100) | NOT NULL, UNIQUE |
| department_code | VARCHAR(20) | NOT NULL, UNIQUE |
| department_head | VARCHAR(100) | nullable |
| description | VARCHAR(500) | nullable |
| is_active | BOOLEAN | NOT NULL, default true |
| created_at | DATETIME | auto-set on insert |
| updated_at | DATETIME | auto-set on update |

Relationship: `Department (1) ──< (M) ApprovalHierarchy` (via `ApprovalHierarchy.department_id`)

---

## 4. DTOs

**CategoryRequest** — `categoryName*, categoryCode*, description, active`
**CategoryResponse** — `id, categoryName, categoryCode, description, active, supplierCount, createdAt, updatedAt`

**DepartmentRequest** — `departmentName*, departmentCode*, departmentHead, description, active`
**DepartmentResponse** — `id, departmentName, departmentCode, departmentHead, description, active, approvalLevelCount, createdAt, updatedAt`

`*` = mandatory. Response DTOs additionally surface derived counts
(`supplierCount`, `approvalLevelCount`) so the client can show at a glance whether a
record is safe to delete.

---

## 5. Bean Validation Rules

| DTO | Field | Rule |
|---|---|---|
| CategoryRequest | categoryName | `@NotBlank`, 2–100 chars |
| CategoryRequest | categoryCode | `@NotBlank`, 2–20 chars |
| CategoryRequest | description | max 500 chars |
| DepartmentRequest | departmentName | `@NotBlank`, 2–100 chars |
| DepartmentRequest | departmentCode | `@NotBlank`, 2–20 chars |
| DepartmentRequest | departmentHead | max 100 chars |
| DepartmentRequest | description | max 500 chars |

All violations return HTTP `400` with a field → message map via `GlobalExceptionHandler`.

---

## 6. Business Rules (Service Layer)

1. **Uniqueness (case-insensitive):**
   - Category name and code must each be unique across all categories.
   - Department name and code must each be unique across all departments.
   - On create: checked against the whole table. On update: checked against all
     *other* records (`...AndIdNot` repository queries) so a record can be saved
     without tripping over its own existing values.
   - Violation → `DuplicateResourceException` → HTTP `409 Conflict`.

2. **Deletion guards:**
   - A category **cannot** be deleted while any supplier is still linked to it
     (`SupplierRepository.existsByCategoryId`). → `BusinessValidationException` → `422`.
   - A department **cannot** be deleted while it still has approval-hierarchy levels
     defined (`ApprovalHierarchyRepository.existsByDepartmentId`). → `422`.
   - Rationale: prevents orphaned foreign keys and silently breaking a live
     supplier's classification or a department's approval workflow.

3. **Code normalization:** codes are trimmed and upper-cased on save
   (`IT-hw` → `IT-HW`) so lookups and comparisons are consistent.

4. **Soft "active" flag:** both entities carry `active` (default `true`) instead of a
   hard delete, so records can be retired from use without breaking historical
   references — hard delete is still available but gated by rule #2.

---

## 7. Search API Design

Both modules expose a `GET /search` endpoint supporting:

- `keyword` — matches against **name OR code**, case-insensitive, partial match
  (`LIKE %keyword%`).
- `active` — filters to only active (`true`) or inactive (`false`) records.
- Both parameters are optional and combinable; with neither supplied, it behaves like
  "get all".

This was implemented via derived Spring Data query methods
(`findByCategoryNameContainingIgnoreCaseOrCategoryCodeContainingIgnoreCase`, and the
department equivalent), with the `active` filter applied in the service layer when both
parameters are present together.

---

## 8. Error Handling Reference

| Scenario | Exception Thrown | HTTP Status |
|---|---|---|
| Category/Department not found by id | `ResourceNotFoundException` | 404 |
| Duplicate name or code | `DuplicateResourceException` | 409 |
| Delete category still linked to suppliers | `BusinessValidationException` | 422 |
| Delete department with approval levels | `BusinessValidationException` | 422 |
| Missing/invalid request fields | `MethodArgumentNotValidException` | 400 |

All handled centrally by `GlobalExceptionHandler` — no try/catch blocks needed in the
controller or service code for these cases.

---

## 9. Testing Checklist (manual / Postman)

- [ ] Create category with valid data → 200, record returned with `active: true`
- [ ] Create category with duplicate name → 409
- [ ] Create category with duplicate code (different case) → 409
- [ ] Update category to a name already used by another category → 409
- [ ] Delete category with no suppliers → 204
- [ ] Delete category with suppliers attached → 422
- [ ] Search categories by partial keyword → matches on name and code
- [ ] Search categories with `active=false` → only inactive records returned
- [ ] Repeat all of the above for Department (substitute "approval hierarchy" for "suppliers" in the deletion-guard test)
- [ ] Submit request with blank `categoryName` / `departmentName` → 400 with field-level message
