# API Guidelines

> Status: **Draft**  
> Visibility: **Public** (`/docs/api-guidelines.md`)

These guidelines define the REST conventions for the CRM backend. They apply to all bounded contexts and adapters.

---

## 1. Fundamentals
- **Style:** Resource‑oriented REST over HTTP/1.1+ with JSON bodies.
- **MIME:** `application/json; charset=utf-8` for success; `application/problem+json` for errors.
- **Charset:** Always UTF‑8.
- **Time:** All timestamps are ISO‑8601 in UTC (e.g., `2025-08-26T18:00:00Z`).
- **IDs:** Prefer UUIDv4 for public identifiers.
- **Case:** JSON field names are `camelCase`.

---

## 2. Versioning
- Base path versioning: `/api/v1/...`.
- Backward‑compatible changes (adding fields, new optional params) do not require a new major.
- Breaking changes require bump to `/api/v2` and deprecation notice period.

---

## 3. Resource Naming & URLs
- Plural nouns: `/api/v1/deals`, `/api/v1/contacts`.
- Use nested scopes sparingly: `/api/v1/companies/{companyId}/deals`.
- Avoid verbs; use sub‑resources or actions when necessary: `/api/v1/deals/{id}:close`.

---

## 4. HTTP Methods & Semantics
- `GET` – retrieve (safe, idempotent)
- `POST` – create / non‑idempotent actions
- `PUT` – full update (idempotent)
- `PATCH` – partial update (idempotent at semantics level)
- `DELETE` – delete (idempotent)

### Status Codes (canonical)
- `200 OK` – success with body
- `201 Created` + `Location` header – on creation
- `202 Accepted` – async processing started
- `204 No Content` – success without body
- `400 Bad Request` – malformed syntax
- `401 Unauthorized` – no/invalid credentials
- `403 Forbidden` – authenticated but not allowed
- `404 Not Found` – resource missing
- `409 Conflict` – version/uniqueness conflict
- `410 Gone` – permanently removed
- `412 Precondition Failed` – ETag precondition failed
- `422 Unprocessable Entity` – validation error (DTO level)
- `429 Too Many Requests` – rate limit
- `5xx` – server errors

---

## 5. Errors — RFC‑7807 Problem+JSON
All errors use `application/problem+json` with the following schema:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "detail": "One or more fields are invalid.",
  "instance": "/api/v1/deals",
  "errors": [
    { "field": "title", "message": "must not be blank" },
    { "field": "amount.currency", "message": "must be 3-letter ISO code" }
  ]
}
```

- `type` SHOULD be a resolvable URL documenting the error category.
- `errors[]` is recommended for field‑level issues.
- Do not leak stack traces or internal IDs.

---

## 6. Pagination, Sorting, Filtering
### 6.1 Offset Pagination (default)
Query params: `page` (1‑based), `size`, `sort` (e.g., `createdAt,desc`).

Response envelope:
```json
{
  "content": [ { /* item */ } ],
  "page": { "page": 1, "size": 20, "totalElements": 57, "totalPages": 3 }
}
```

### 6.2 Cursor Pagination (optional for large datasets)
- Request: `cursor=<opaque>&size=20`
- Response headers: `Next-Cursor: <opaque>`

### 6.3 Filtering
- Use simple query params (`q`, `stage=PROPOSAL`), or explicit filters (`?ownerId=...&minAmount=...`).
- Avoid complex RSQL; prefer multiple explicit params.

---

## 7. Concurrency & Caching
- **ETags** on GET for single resources.
- Updates require `If-Match` with ETag; else return `412 Precondition Failed`.
- Support `If-None-Match` for cache validation on GET.
- Strong ETags preferred; weak only for large, computed representations.

---

## 8. Idempotency
- For idempotent create/execute semantics (e.g., external callbacks), accept header `Idempotency-Key` and store keys for a safe window.

---

## 9. Security
- **Auth**: `Authorization: Bearer <JWT>`.
- **Transport**: HTTPS only in non‑local environments.
- **Scopes/Roles**: enforce at controller or method level (Spring Security @PreAuthorize).
- **Input Hardening**: validate DTOs with Bean Validation; reject unknown enum values; set size limits.

---

## 10. Observability
- Correlate requests with `X-Request-Id` (generate if missing; echo back).
- Log structured JSON; never log secrets or JWTs.
- Expose health/metrics via Actuator; secure non‑public endpoints.

---

## 11. Problem Types (registry)
- `https://api.example.com/problems/validation-error` → 422
- `https://api.example.com/problems/conflict` → 409
- `https://api.example.com/problems/not-found` → 404
- `https://api.example.com/problems/unauthorized` → 401

(Backed by a static page in public docs.)

---

## 12. Examples
### 12.1 Create Deal — 201
**Request** `POST /api/v1/deals`
```http
Content-Type: application/json
Authorization: Bearer <JWT>
```
```json
{ "title": "ACME Renewal Q4", "amount": { "amount": 25000, "currency": "USD" }, "ownerId": "u-123" }
```
**Response** `201 Created`
```http
Location: /api/v1/deals/d4a1b2c3-4e5f-6789-8abc-1234567890de
```
```json
{ "id": "d4a1b2c3-4e5f-6789-8abc-1234567890de", "title": "ACME Renewal Q4", "amount": { "amount": 25000, "currency": "USD" }, "stage": "NEW", "ownerId": "u-123", "createdAt": "2025-08-26T18:00:00Z", "updatedAt": "2025-08-26T18:00:00Z" }
```

### 12.2 Validation Error — 422
```json
{ "type": "https://api.example.com/problems/validation-error", "title": "Validation failed", "status": 422, "errors": [ { "field": "title", "message": "must not be blank" } ] }
```

---

## 13. Compatibility Contract
- Unknown JSON properties from clients are **ignored** by default (non‑breaking).  
- Do not remove or repurpose fields without a major version change.  
- Use enums conservatively; document allowed values and add new ones as backward‑compatible extension.

---

## 14. Checklist (per endpoint)
- [ ] Uses `/api/v{n}` prefix
- [ ] Validates input with Bean Validation
- [ ] Returns Problem+JSON on errors
- [ ] Supports pagination/sort where applicable
- [ ] Emits/validates ETags for single resources
- [ ] Secured with JWT
- [ ] Documented in OpenAPI with examples

