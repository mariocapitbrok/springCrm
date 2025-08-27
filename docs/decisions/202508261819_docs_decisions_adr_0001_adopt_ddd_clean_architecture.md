# ADR-0001: Adopt DDD + Clean Architecture (Modular Monolith)

**Status:** Accepted  
**Date:** 2025-08-26  
**Decision Owners:** Architecture & Docs Thread  
**Visibility:** Public (`/docs/decisions/ADR-0001-adopt-ddd-clean-architecture.md`)

---

## Context
We need a clear, durable architecture for a portfolio‑grade CRM API that:
- Keeps the **core domain** independent from frameworks.
- Is **fast to develop** as a single deployable unit today.
- Allows **straightforward extraction** of services later if scale/teams require it.
- Aligns with our chosen stack: **Java 17**, **Spring Boot**, **Gradle**, **PostgreSQL**, **Spring Data JPA (Hibernate)**, **MapStruct**, **Flyway**, **Spring Security + JWT**, **springdoc‑openapi**, **Testcontainers**, **Micrometer/Actuator**, **Resilience4j** (Lombok optional; prefer Java records when appropriate).

A conventional layered monolith risks framework coupling and unclear boundaries; microservices from day one adds premature complexity. We need strong internal boundaries without distributed systems overhead.

---

## Decision
Adopt **DDD + Clean Architecture** within a **modular monolith**.

### 1) Architectural Style
- **DDD** to define ubiquitous language and bounded contexts.
- **Clean Architecture (Ports & Adapters)** to isolate the core from frameworks.
- Single deployable artifact; **strong internal modularity**.

```mermaid
flowchart LR
  subgraph Core[Domain / Application]
    D[Domain] --> A[Application]
  end
  subgraph Adapters[Adapters]
    W[Web (HTTP/REST)]
    P[Persistence (JPA/Hibernate)]
  end
  W --> A --> P
```

### 2) Dependency Rule
- `domain` → depends on nothing (pure Java).  
- `application` → depends only on `domain` and **ports** (interfaces).  
- `adapters` → depend on `application`; hold all framework code.  
- Spring/JPA/web annotations/classes **do not** leak into domain.

### 3) Packaging by Bounded Context
Top‑level package: `com.example.crm`

```
com.example.crm
├─ sales/
│  ├─ domain/            # Aggregates, Value Objects, Domain Events
│  ├─ application/       # Use cases, ports (interfaces)
│  ├─ adapters/
│  │  ├─ web/            # Controllers, DTOs, mappers (DTO↔Domain)
│  │  └─ persistence/    # JPA entities, repos (Domain↔JPA)
│  └─ config/            # Spring wiring for this context
├─ activities/           # same pattern
└─ shared/               # Shared Kernel (common VOs, error types, etc.)
```

**Shared Kernel rules**: only cross‑context VOs/utilities; no business rules. Changes require multi‑context review.

### 4) Cross‑Cutting Policies
- **Errors**: RFC‑7807 Problem+JSON (web adapter).  
- **Validation**: Bean Validation at DTOs; invariants inside domain.  
- **Transactions**: demarcated at **application services** (`@Transactional`).  
- **Logging/Tracing**: structured logs + correlation IDs; domain stays silent.  
- **Observability**: Actuator + Micrometer.  
- **Resilience**: Resilience4j for retries/timeouts/circuit breakers.

### 5) Persistence & Mapping
- **PostgreSQL** via **Spring Data JPA**.  
- **Flyway** for schema migrations (startup + tests).  
- **MapStruct** for DTO↔Domain and Domain↔JPA mappings; manual mapping when complex.

### 6) Security
- **Spring Security + JWT** (stateless).  
- Token/refresh specifics are deferred to **ADR‑0002 (JWT strategy)**.

### 7) API & Docs
- RESTful controllers; auto‑generate OpenAPI via **springdoc‑openapi**.  
- CI exports spec to `docs/openapi/openapi.json` and links from `docs/index.md`.

### 8) Testing
- **Slice tests** (`@WebMvcTest`, `@DataJpaTest`, `@JsonTest`) for speed.  
- **E2E** via **Testcontainers (Postgres)** with Flyway applied.  
- Target ≈ **70%** coverage focusing on domain rules and core flows.

### 9) Build & Delivery
- **Gradle** build, multi‑stage **Dockerfile**; `docker-compose` for local dev.  
- CI: build, slice + E2E tests, OWASP dep check, export OpenAPI, push image.

---

## Alternatives Considered
1. **Big monolith (framework‑centric)** — Faster to start but couples domain to Spring/JPA; extraction becomes risky.
2. **Classic layered architecture** — Better than #1 but layers tend to erode; domain purity not enforced.
3. **Microservices from day one** — Operational complexity, premature distribution, slower learning loop.
4. **Hexagonal architecture** — Essentially equivalent to Ports & Adapters; chosen terminology is Clean Architecture.

---

## Consequences
**Positive**
- High **testability** and **maintainability**; domain logic remains portable.  
- Clear seams for future **service extraction**.  
- Tech changes (e.g., swap JPA/provider) isolated to adapters.

**Negative / Trade‑offs**
- Extra **boilerplate** (ports, mappers, DTOs).  
- Requires **discipline** to preserve boundaries.  
- Slightly slower initial feature velocity vs. a framework‑centric monolith.

Mitigations: MapStruct for mapping; templates for ports/DTOs; periodic architecture reviews.

---

## Implementation Notes
- Start with bounded contexts: **sales**, **activities**, **shared**.  
- MVP focus: `sales` with `/api/deals` list/create (OpenAPI stub first).  
- Keep a **single Gradle module** initially; enforce boundaries by package and reviews. Optionally add **ArchUnit** tests later.
- Use **records** where suitable (DTOs/VOs). Lombok optional.
- Apply **Index Policy**: public docs in `/docs`, sensitive notes in `/docs/private`.

---

## Acceptance Criteria
- `docs/architecture.md` present and aligned with this ADR.  
- Package blueprint exists in codebase skeleton.  
- First OpenAPI stub under `docs/openapi/` linked from `docs/index.md`.  
- RFC‑7807 error contract defined in API guidelines.

---

## References
- `docs/architecture.md`  
- `docs/api-guidelines.md` (to add Problem+JSON details)  
- `docs/ops.md` (observability, deployment)  
- `docs/openapi/openapi.json` (exported via CI)

