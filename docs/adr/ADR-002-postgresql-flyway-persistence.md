# ADR-002: PostgreSQL And Flyway Persistence Foundation

Date: 2026-08-21

Status: Accepted

## Context

The Identity service needs a production-ready database foundation before feature branches add durable user, role, and roster registration data.

The service is independently deployable and owns its module data. Secrets must not be hardcoded or committed.

## Decision

Identity will use PostgreSQL as its main database.

Supabase may be used as the managed PostgreSQL provider, but only for database hosting. Supabase Auth is not part of this decision.

The service will be configured with environment variables:

- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`

Local development may use a local `.env` file loaded by Spring Boot config import. The real `.env` file is ignored by Git. `.env.example` is committed as the template.

Flyway will own schema migrations under:

```text
src/main/resources/db/migration
```

Identity will use the shared/default PostgreSQL schema. Flyway will use `identity_flyway_schema_history` so Identity migration versions do not collide with another service's Flyway history table in the same database.

Because the shared/default schema may already contain objects before Identity starts tracking migrations, Flyway will use `baseline-on-migrate=true` with baseline version `1`.

Hibernate will validate schema with:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

Open Session In View is disabled.

For IPv4-only deployments on Supabase, runtime traffic should use a Supabase pooler connection string. Persistent Spring Boot services should prefer Supavisor session mode. Transaction mode is reserved for serverless or short-lived workloads and requires prepared statements to be disabled in the JDBC URL.

## Consequences

Positive:

- Production configuration can be supplied by deployment environment or secret manager.
- Local development can use `.env` without committing credentials.
- Schema changes are explicit and reviewable through Flyway migrations.
- Hibernate cannot silently create or mutate production schema.
- Supabase can be used without replacing the Identity service's authentication and access-rule ownership.

Tradeoffs:

- The service will fail startup when required database environment variables are missing.
- Local development requires PostgreSQL and a populated `.env` before running the full service.
- Tests that do not need a database must explicitly avoid database auto-configuration until persistence integration tests are added.
- Supabase connection limits must be managed across AIASPES services with conservative Hikari pool settings.

## Follow-Up

- Add persistence integration tests after Docker/local PostgreSQL validation is available.
- Replace any temporary in-memory repositories with PostgreSQL-backed repositories after their domain models are finalized.
- Add immutable migrations for the first persisted Identity tables.
- Add JPA auditing when the first persisted entity exists.
