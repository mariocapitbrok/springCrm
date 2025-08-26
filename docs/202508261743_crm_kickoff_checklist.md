# CRM Project Kickoff Checklist

This is the actionable checklist to bootstrap the CRM REST API following our **DDD + Clean Architecture** modular monolith.

---

## 0) Prereqs
- 🔲 Install **Java 17**, **Gradle**, **Docker** & **Docker Compose**
- 🔲 Ensure **PostgreSQL** images pull correctly (`docker pull postgres:16`)
- 🔲 VS Code extensions (Java, Lombok if used), Git configured

---

## 1) Repo & Project Init
- 🔲 Create repo: `crm-api`
- 🔲 Generate project via Spring Initializr (Web, Security, Data JPA, Validation, PostgreSQL, Flyway, Actuator)
- 🔲 Add base packages per **bounded contexts** (`sales`, `activities`, `shared`)
- 🔲 Commit: `chore: init spring boot project`

---

## 2) Docs Scaffolding
- 🔲 Create `/docs/index.md` (public portal)
- 🔲 Create `/docs/private/index.md`
- 🔲 Add `getting-started.md`, `architecture.md`, `ops.md`, `api-guidelines.md`
- 🔲 Add `/docs/decisions/README.md` and start ADR-0001 (architecture choice)

---

## 3) Dockerization
- 🔲 **Multi-stage Dockerfile** (build JAR → slim runtime)
- 🔲 `docker-compose.yml` with services: `app`, `postgres`, optional `pgadmin`
- 🔲 Healthchecks + volumes for Postgres data

---

## 4) Configuration & Profiles
- 🔲 `application.yml`: `dev`, `test`, `prod` profiles
- 🔲 Env vars for DB + JWT secrets; disable `ddl-auto` (Flyway-only)
- 🔲 Logging pattern with correlation/request IDs

---

## 5) Flyway & Database
- 🔲 Baseline migration: `V1__init.sql` (users, accounts, contacts, deals, activities, notes)
- 🔲 Seed/dev data script `R__seed_dev.sql` (optional)
- 🔲 Verify migrations run locally with Compose

---

## 6) Domain & Application Skeleton (sales context)
- 🔲 Domain: `Deal`, `DealId`, `Money` (VO), `Stage` (enum), domain events
- 🔲 Application ports: `DealRepository`, `CreateDealUseCase`, `AdvanceDealUseCase`
- 🔲 Application services: `CreateDealService`, `AdvanceDealService` (transactional boundaries)

---

## 7) Adapters (Web + Persistence)
- 🔲 Persistence: JPA entities + Spring Data repo (`JpaDealRepository` implements port)
- 🔲 MapStruct mappers (DTO↔domain, JPA↔domain)
- 🔲 Web: `DealController` with DTOs + validation + Problem+JSON error handling

---

## 8) Security (JWT)
- 🔲 Add JWT config (resource server or custom filter)
- 🔲 Public endpoints (health, docs), protected `/api/**`
- 🔲 Role model (ADMIN/USER); sample user bootstrap for dev

---

## 9) OpenAPI / springdoc
- 🔲 Add springdoc; Swagger UI available at `/swagger-ui.html`
- 🔲 Annotate controller methods with examples
- 🔲 CI step to export `/v3/api-docs` → `docs/openapi/openapi.json`

---

## 10) Testing Setup
- 🔲 **Unit**: domain rules & application services (pure Java)
- 🔲 **Slice**: `@WebMvcTest` for controller; `@DataJpaTest` for repository
- 🔲 **E2E**: `@SpringBootTest` + **Postgres Testcontainer**; Flyway enabled
- 🔲 Coverage ~70%; add Jacoco if desired

---

## 11) CI/CD (GitHub Actions)
- 🔲 Workflow: build → unit/slice tests → E2E (Testcontainers) → export OpenAPI → build & push image
- 🔲 Cache Gradle; set up GHCR or Docker Hub login
- 🔲 Tag images with SHA + `vX.Y.Z`

---

## 12) Quality Gates
- 🔲 Add **Spotless** (format) + **Checkstyle** (rules)
- 🔲 Enable Dependabot alerts & updates
- 🔲 Optional: OWASP Dependency-Check, secret scanning

---

## 13) First Feature End-to-End
- 🔲 POST `/api/deals` → 201 Created + `Location`
- 🔲 GET `/api/deals` (pagination, filtering by stage/title)
- 🔲 Advance deal stage use case with invariant checks
- 🔲 E2E test proves create → query → advance works on Postgres

---

## 14) Runbook & Dev UX
- 🔲 `make` or `npm-scripts`-style aliases for `build`, `test`, `run`, `compose:up`
- 🔲 Troubleshooting section in `getting-started.md`
- 🔲 Document local auth flow (how to obtain a JWT in dev)

---

## 15) Next Steps (Backlog Seeds)
- 🔲 Contacts ↔ Accounts relations
- 🔲 Activities/Tasks with due dates & reminders
- 🔲 Notes & attachments (future: S3)
- 🔲 Search & filtering spec
- 🔲 Auditing (created/updated by)
- 🔲 Observability dashboards (Prometheus/Grafana)

