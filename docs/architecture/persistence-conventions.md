# Identity Persistence Conventions

Last updated: 2026-08-21

## Purpose

This document records the PostgreSQL persistence foundation for the AIASPES Identity service.

## Confirmed Decisions

- PostgreSQL is the main database for Identity.
- Runtime database configuration must come from environment variables.
- Local development may load those variables from a local `.env` file.
- Real `.env` files must not be committed.
- `.env.example` is committed as the local setup template.
- Flyway owns schema migrations.
- Hibernate must validate schema; it must not create or update production schema.
- Open Session In View is disabled.

## Environment Variables

Required:

- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`

Optional:

- `DB_POOL_MAX_SIZE`
- `DB_POOL_MIN_IDLE`
- `DB_CONNECTION_TIMEOUT_MS`
- `DB_VALIDATION_TIMEOUT_MS`
- `DB_IDLE_TIMEOUT_MS`
- `DB_MAX_LIFETIME_MS`

Local development can copy `.env.example` to `.env` and fill in local values.

## Spring Configuration

The service imports local environment files with:

```properties
spring.config.import=optional:file:.env[.properties]
```

Production deployments should provide the same variables through the hosting platform or secret manager, not through a committed file.

## Supabase Deployment

Supabase is used only as a managed PostgreSQL database for Identity.

Confirmed:

- Identity still owns authentication and access rules unless a future ADR says otherwise.
- Supabase Auth is not part of the current Identity foundation.
- Local credentials stay in `.env`.
- Production credentials must be supplied through deployment secrets.

For IPv4-only deployment environments:

- Use a Supabase pooler connection string in `DB_URL`.
- Prefer Supavisor session mode for persistent Spring Boot services.
- Use transaction mode only for serverless or short-lived execution models.
- Transaction mode does not support prepared statements; disable prepared statements in the JDBC URL if transaction mode is used.
- Keep Hikari pool settings conservative so multiple AIASPES services do not exhaust Supabase connection limits.

Recommended runtime defaults:

- `DB_POOL_MAX_SIZE=5`
- `DB_POOL_MIN_IDLE=1`

## Migration Rules

- Migrations live under `src/main/resources/db/migration`.
- Identity uses the shared/default PostgreSQL schema.
- Flyway stores Identity migration history in `identity_flyway_schema_history`.
- Flyway uses `baseline-on-migrate=true` because the shared/default schema may already be non-empty before Identity starts tracking migrations.
- Migrations are immutable after they are shared.
- Use explicit snake_case names for tables, columns, indexes, and constraints.
- Do not use `ddl-auto=create`, `ddl-auto=create-drop`, or `ddl-auto=update`.
- Use `spring.jpa.hibernate.ddl-auto=validate`.

## Entity Conventions

For future persisted models:

- Use UUID identifiers.
- Use `Instant` for audit timestamps.
- Use `@Version` for mutable records.
- Prefer explicit table, column, index, and constraint names.
- Keep persistence code inside the service repository.
- Do not import persistence contracts from another backend service.
- Add JPA auditing configuration when the first persisted entity is introduced.

## Current Schema

`V1__initialize_identity_schema.sql` intentionally creates no application tables. It initializes Flyway history for the service while the first persisted Identity model is still being finalized.

Local validation confirmed that Spring Boot loads `.env`, connects to PostgreSQL, creates `public.identity_flyway_schema_history`, baselines Identity at version 1 in the shared/default schema, and initializes Hibernate.
