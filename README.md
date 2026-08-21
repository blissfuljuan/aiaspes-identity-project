# AIASPES Identity

AIASPES Identity is the identity, access, and access-role service for the AI-Assisted Software Project Evaluation System.

This repository is a separately deployable Spring Boot service. System-level architecture and module coordination live in the main AIASPES documentation repository:

```text
C:\Projects\AIASPES\ai-assisted-software-project-eval-system
```

## Current Status

Status: foundation documentation in progress.

Confirmed:

- This service owns Identity module business logic and module-owned data.
- External/client access should be routed through the AIASPES API Gateway.
- Internal service-to-service workflows should use direct service calls or event/message communication as appropriate.
- Business rules stay inside services, not in the gateway, broker, or event bus.

Not yet implemented:

- User account model
- Authentication/login
- Role assignment
- Access checks
- Application persistence schema
- Cross-module authenticated user context

## Service Scope

Confirmed responsibilities:

- Identity management
- Access management
- Access-role management

Confirmed core roles:

- Student
- Instructor
- Admin

Optional roles, deferred until explicitly needed:

- Evaluator
- Adviser
- Panelist

## Technical Baseline

- Spring Boot parent: `4.1.1`
- Java release target: `17`
- Build tool: Maven
- Group ID: `com.blissfuljuan.aiaspes`
- Artifact ID: `aiaspes-identity`
- Base package: `com.blissfuljuan.aiaspes.identity`

## Local Validation

On this Windows workspace, the Maven wrapper may fail before Maven starts. If `mvnw.cmd` reports `Cannot start maven from wrapper`, use the cached Maven distribution with the local JDK:

```powershell
$env:JAVA_HOME='C:\Users\eric\.jdks\corretto-21.0.11'
& 'C:\Users\eric\.m2\wrapper\dists\apache-maven-3.9.16\0daed3be3ebd1c706f0e69e8b07c6b73f5cc4ea3dfce72a8d0ec2e849ca2ddb0\bin\mvn.cmd' test
```

Current baseline result:

```text
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

## Documentation

- `PROJECT_STATUS.md` records the latest continuation checkpoint.
- `docs/architecture/module-boundaries.md` records service boundaries and package direction.
- `docs/architecture/identity-foundation.md` records planned foundation decisions before feature implementation.
- `docs/architecture/persistence-conventions.md` records PostgreSQL and Flyway conventions.
- `docs/adr/ADR-001-identity-service-boundary-and-foundation.md` records the first architecture decision.
- `docs/adr/ADR-002-postgresql-flyway-persistence.md` records the database foundation decision.

## Database Configuration

PostgreSQL is the main Identity database. Runtime configuration is read from environment variables:

```text
DB_URL
DB_USERNAME
DB_PASSWORD
```

Optional connection-pool settings:

```text
DB_POOL_MAX_SIZE
DB_POOL_MIN_IDLE
DB_CONNECTION_TIMEOUT_MS
DB_VALIDATION_TIMEOUT_MS
DB_IDLE_TIMEOUT_MS
DB_MAX_LIFETIME_MS
```

For local development, copy `.env.example` to `.env` and fill in local values. The real `.env` file is ignored and must not be committed.

Schema management:

- Flyway migrations live in `src/main/resources/db/migration`.
- Identity uses the shared/default PostgreSQL schema.
- Flyway uses `identity_flyway_schema_history` so Identity migrations do not collide with another service's Flyway versions.
- Flyway baselines on migrate because the shared/default schema may already contain objects from local development or other modules.
- Hibernate uses `ddl-auto=validate`.
- Open Session In View is disabled.

## Supabase Deployment Notes

Supabase should be used as a managed PostgreSQL database only for this service. Do not use Supabase Auth unless a future Identity ADR explicitly chooses it.

For IPv4-only deployment environments:

- Use the Supabase pooler connection string in `DB_URL`.
- Prefer Supavisor session mode for persistent Spring Boot services.
- Use transaction mode only for serverless or short-lived workloads.
- If transaction mode is used, disable prepared statements in the JDBC URL because transaction pooling does not support them.
- Keep Hikari pool sizes conservative and tune them with `DB_POOL_MAX_SIZE` and `DB_POOL_MIN_IDLE`.

Real Supabase credentials belong in local `.env` or production secret settings, not in Git.
