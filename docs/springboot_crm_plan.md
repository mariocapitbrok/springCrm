# RESTful CRM API with Java + Spring Boot

## 🎯 Goal
Build a production-ready **CRM REST API** using Java + Spring Boot, focusing on speed, maintainability, and clean architecture.

---

## 📦 MVP Scope
- **Auth & Users**
  - Signup/login, JWT, refresh tokens, role-based access (ADMIN, USER).
- **Core Entities**
  - Account/Company
  - Contact
  - Deal
  - Activity/Task
  - Note
  - User
- **CRUD + Queries**
  - Pagination, sorting, filtering.
- **Relations**
  - Contacts ↔ Accounts
  - Deals ↔ (Account, Contacts)
  - Activities ↔ (Deal or Contact)
- **Business Flows**
  - Deal stages, activity reminders.
- **Observability**
  - Health, metrics, logs, tracing.
- **Docs**
  - OpenAPI (Swagger UI).
- **Security**
  - Endpoint authorization, input validation, rate limiting.
- **Data**
  - Migrations (Flyway), demo seed data.
- **Tests**
  - Unit + slice + integration (Testcontainers).

---

## ⚙️ Recommended Stack
- **Spring Boot**: Web, Validation
- **Spring Data JPA** (Hibernate) + PostgreSQL
- **Spring Security** + JWT
- **Flyway** (migrations)
- **MapStruct** (DTO mapping)
- **springdoc-openapi**
- **Testcontainers**, JUnit 5, Mockito
- **Gradle**, Java 17+

---

## 🗂 Suggested Milestones
1. **Project bootstrap**
   - Init project, Docker Compose (Postgres + pgAdmin), CI pipeline.
2. **Auth & users**
   - Password hashing, JWT/refresh tokens, roles, tests.
3. **Accounts & contacts**
   - CRUD, search/filter/pagination, DTOs + validation.
4. **Deals & pipeline**
   - Stages, filters, domain services for transitions.
5. **Activities & notes**
   - Tasks with due dates, notes linked to entities.
6. **Cross-cutting**
   - Exception handling, audit fields, logging, rate limiting.
7. **Docs & observability**
   - OpenAPI docs, actuator endpoints.
8. **Hardening**
   - Constraints, indexes, N+1 checks, transactions.
9. **Packaging & deploy**
   - Container image, config via env vars.
10. **QA & acceptance**
   - End-to-end happy/error paths, migration rehearsal.

---

## 📏 Estimation Framework
- Break into **user stories** with acceptance criteria.
- Assign **story points (SP)**.
- Track **velocity** (SP/iteration).
- **Duration = Total SP ÷ Velocity** (plug in your own team’s metrics).
- Include buffers for schema changes, auth, reporting, integrations.

---

## ✅ Definition of Done (per endpoint)
- DTO + validation, service, repository.
- Correct HTTP codes (200, 201, 204, 400, 401, 403, 404, 409).
- Unit + slice tests (≥1 happy, ≥2 error paths).
- OpenAPI documented.
- Pagination performance validated.

---

## 🚀 Accelerators
- Start with **ERD/data model** first.
- **OpenAPI-first** (generate controllers/clients).
- **MapStruct** for DTO ↔ entity.
- **Testcontainers** for DB realism.
- **Flyway** from day 1.

---

## 🔧 Quick Bootstrap
```bash
# Init via Spring Initializr
curl https://start.spring.io/starter.zip \
  -d dependencies=web,security,data-jpa,validation,postgresql,flyway,actuator \
  -d language=java -d type=gradle-project -d javaVersion=17 \
  -d name=crm-api -o crm-api.zip && unzip crm-api.zip -d crm-api

cd crm-api
```

