# Member 2 — Agile Documentation
## Procurement Category Management & Department Management

---

## 1. Epics Owned

- **Epic: Procurement Category Management**
- **Epic: Department Management**

---

## 2. User Stories

### Category Management

**US-2.1 — Create Category**
> As an Admin, I want to create a procurement category with a unique name and code, so
> that suppliers can be classified consistently.
- **Acceptance Criteria:**
  - `categoryName` and `categoryCode` are mandatory.
  - Duplicate name or code (case-insensitive) is rejected with a clear `409` error.
  - New categories default to `active = true` unless specified otherwise.

**US-2.2 — Update Category**
> As an Admin, I want to update a category's details, so information stays accurate.
- **Acceptance Criteria:**
  - Uniqueness is re-validated excluding the record being updated.
  - Non-existent category id returns `404`.

**US-2.3 — Prevent Deletion of Linked Categories**
> As an Admin, I should not be able to delete a category that suppliers are still
> assigned to, so that supplier records never point to a missing category.
- **Acceptance Criteria:**
  - Delete is blocked with `422` and a descriptive message if `Supplier.category_id`
    references the category.
  - Delete succeeds (`204`) once no suppliers reference it.

**US-2.4 — Search Categories**
> As an Admin, I want to search categories by name or code, and optionally filter by
> active status, so I can quickly locate the right classification.
- **Acceptance Criteria:**
  - Keyword search matches partial, case-insensitive text in name **or** code.
  - `active` filter can be combined with keyword search.

### Department Management

**US-3.1 — Create Department**
> As an Admin, I want to register a department with a unique name and code, so
> procurement requests can be attributed to the correct team.
- **Acceptance Criteria:**
  - `departmentName` and `departmentCode` are mandatory and unique.
  - Optional `departmentHead` and `description` fields.

**US-3.2 — Update Department**
> As an Admin, I want to update department details (e.g. change of department head),
> so records stay current.
- **Acceptance Criteria:** same uniqueness re-validation pattern as categories.

**US-3.3 — Prevent Deletion of Departments with Active Workflows**
> As an Admin, I should not be able to delete a department that already has an
> approval hierarchy configured, so I don't silently break an active approval workflow.
- **Acceptance Criteria:**
  - Delete is blocked with `422` if `ApprovalHierarchy.department_id` references it.
  - Delete succeeds once all approval levels for that department are removed.

**US-3.4 — Search Departments**
> As an Admin, I want to search departments by name or code, and filter by active
> status, so I can find the right department quickly in a large organization.
- **Acceptance Criteria:** same behavior as US-2.4, scoped to departments.

---

## 3. Task Breakdown (Sprint Board View)

| Story | Tasks | Status |
|---|---|---|
| US-2.1 | Entity + Repository + DTO + Bean Validation + Service + Controller + duplicate-check logic | Done |
| US-2.2 | Update endpoint + uniqueness-excluding-self check | Done |
| US-2.3 | `existsByCategoryId` check in `SupplierRepository` + deletion guard in `CategoryServiceImpl` | Done |
| US-2.4 | Derived query methods + `/categories/search` endpoint | Done |
| US-3.1 | Entity + Repository + DTO + Bean Validation + Service + Controller + duplicate-check logic | Done |
| US-3.2 | Update endpoint + uniqueness-excluding-self check | Done |
| US-3.3 | `existsByDepartmentId` check in `ApprovalHierarchyRepository` + deletion guard in `DepartmentServiceImpl` | Done |
| US-3.4 | Derived query methods + `/departments/search` endpoint | Done |

---

## 4. Definition of Done

- [x] Layered implementation (Entity → Repository → Service → Controller)
- [x] Bean Validation applied on all request DTOs
- [x] Duplicate-prevention business rule enforced at service layer
- [x] Deletion-guard business rule enforced at service layer
- [x] Search API implemented and combinable with filters
- [x] All errors return structured JSON via shared `GlobalExceptionHandler`
- [x] Manually verified via Postman (see testing checklist in `MODULE_DOCUMENTATION.md`)
- [x] Documentation delivered (this file + `MODULE_DOCUMENTATION.md` + `API_DOCUMENTATION.md`)

---

## 5. Dependencies on Other Members

| Dependency | Owner | Why |
|---|---|---|
| `Supplier.category_id` foreign key | Member 1 | Needed to check "is this category linked to a supplier" before allowing delete |
| `ApprovalHierarchy.department_id` foreign key | Member 3 | Needed to check "does this department have approval levels" before allowing delete |
| Shared `GlobalExceptionHandler`, `ApiResponse<T>` | Shared/common | Both modules reuse these rather than defining their own |

---

## 6. Retrospective Notes (template to fill in after demo)

- **What went well:**
- **What could improve:**
- **Action items for next sprint:**
