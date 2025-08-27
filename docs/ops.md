# Operations Guide (Ops)

> Status: **Draft**  
> Visibility: **Public** (`/docs/ops.md`)

This document defines runtime operations, observability, deployment, and CI/CD policies for the CRM modular monolith.

---

## 1. Runtime Environments
- **Local dev**: Docker Compose (`app`, `postgres`, optional `pgadmin`).
- **CI**: GitHub Actions with Testcontainers.
- **Staging/Prod**: Docker image deployed via container orchestrator (future: Kubernetes).

### Profiles
- `dev` → local with hot reload, seeded data.
- `test` → used by Testcontainers, Flyway migrations auto-run.
- `prod` → hardened config, migrations run once at startup, no seeded data.

---

## 2. Configuration
- **Source**: environment variables; no secrets in repo.
- **Spring profiles** set via `SPRING_PROFILES_ACTIVE`.
- **DB**: `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`.
- **JWT**: `JWT_SECRET`, `JWT_EXPIRATION_MINUTES`.
- **Other**: log level, observability endpoints.

---

## 3. CI/CD (GitHub Actions)
### Pipeline Stages
1. **Build**: Gradle wrapper build.
2. **Slice tests**: `@WebMvcTest`, `@DataJpaTest`, `@JsonTest`.
3. **E2E tests**: `@SpringBootTest` with Testcontainers (Postgres).
4. **Static analysis**: Spotless, Checkstyle, OWASP Dependency-Check.
5. **Export OpenAPI**: call `/v3/api-docs`, save to `docs/openapi/openapi.json`.
6. **Build Docker image**: multi-stage Dockerfile.
7. **Publish image**: GHCR or Docker Hub; tags = `vX.Y.Z` + Git SHA.

---

## 4. Deployment
- **Container image**: built via multi-stage Dockerfile.
- **Compose**: `docker-compose.yml` for local dev.
- **Prod**: run container with env vars mounted; attach to managed Postgres.
- **Migrations**: Flyway auto-runs at app startup; fail fast on schema drift.

---

## 5. Observability
### Health Checks
- `/actuator/health` (liveness, readiness).

### Metrics
- `/actuator/metrics` via Micrometer.
- Default: JVM, HTTP, DB pool; extend with custom counters/timers.

### Tracing
- Expose trace IDs; correlation via `X-Request-Id`.
- Integrate with external tracing backend (future: OpenTelemetry).

### Logging
- JSON structured logs.
- Fields: timestamp, level, logger, message, requestId, userId (if available).
- No sensitive data (JWTs, passwords).

### Dashboards
- Export Prometheus metrics.
- Visualize with Grafana dashboards (staging/prod).

---

## 6. Security Ops
- JWT secrets via env vars or secret manager.
- HTTPS enforced in non-local environments.
- Rotate secrets regularly.
- Limit DB role to schema used by app.
- Enable Dependabot + vulnerability scans.

---

## 7. Runbook (Dev UX)
- `make build` → Gradle build.
- `make test` → unit + slice + E2E.
- `make run` → start app locally.
- `make compose:up` → launch Postgres + app in Docker Compose.
- Troubleshooting documented in `getting-started.md`.

---

## 8. Backups & Data
- Prod DB (Postgres) configured with automated daily backups.
- Local dev: ephemeral volumes via Docker Compose.

---

## 9. Incident Response
- Monitor health endpoints.
- On failure: check logs + metrics; roll back with previous image tag.
- Postmortems required for prod incidents.

---

## 10. Future Enhancements
- Deploy to Kubernetes (Helm charts).
- Add centralized logging (ELK or OpenSearch).
- Add distributed tracing (OpenTelemetry + Jaeger/Zipkin).
- Set up SLO/SLI metrics for uptime, latency.

