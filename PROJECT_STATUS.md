# AIASPES Identity Module Status

Last updated: 2026-08-21

## Module Purpose

The Identity module is the AIASPES identity, access, and access-role service.

## Confirmed Repository

Repository:

- [aiaspes-identity](https://github.com/blissfuljuan/aiaspes-identity-project.git)

Local workspace:

```text
C:\Projects\AIASPES\aiaspes-identity
```

Main system documentation repository:

```text
C:\Projects\AIASPES\ai-assisted-software-project-eval-system
```

## Confirmed Architecture Context

This module is part of the AIASPES microservices architecture with an API Gateway and event-driven messaging.

Confirmed:

- Identity remains a separate repository and independently deployable service.
- The service owns identity, access, access-role business logic, and module-owned data.
- External/client access should be routed through the AIASPES API Gateway.
- Service-to-service workflows should not be forced through the API Gateway.
- Other services may call Identity synchronously when they need an immediate identity/access response.
- Identity may publish events/messages where asynchronous service-to-service communication is appropriate.
- Any message broker/event bus transports messages only and must not contain Identity business rules.
- Workflow orchestration should be introduced only when a genuine cross-service business process requires it.

## Confirmed Scope

Confirmed responsibilities:

- Identity management
- Access management
- Access-role management

Confirmed core roles:

- Student
- Instructor
- Admin

Confirmed optional roles:

- Evaluator
- Adviser
- Panelist

Optional roles are planned but deferred until an explicit workflow needs them.

## Phase 1 Repository Baseline

Status: completed for current scaffold.

Verified:

- Git branch: `main`
- Git remote: `https://github.com/blissfuljuan/aiaspes-identity-project.git`
- Git state during baseline audit: clean
- Spring Boot parent version: `4.1.1`
- Java release target: `17`
- Build tool: Maven
- Group ID: `com.blissfuljuan.aiaspes`
- Artifact ID: `aiaspes-identity`
- Base package: `com.blissfuljuan.aiaspes.identity`
- Current source surface: Spring Boot application class and one context-load test
- PostgreSQL/Flyway database setup branch: `feat/database-connection`

Baseline validation:

```powershell
cmd /c mvnw.cmd test
```

Result:

```text
Cannot index into a null array.
Cannot start maven from wrapper
```

Fallback validation:

```powershell
$env:JAVA_HOME='C:\Users\eric\.jdks\corretto-21.0.11'
& 'C:\Users\eric\.m2\wrapper\dists\apache-maven-3.9.16\0daed3be3ebd1c706f0e69e8b07c6b73f5cc4ea3dfce72a8d0ec2e849ca2ddb0\bin\mvn.cmd' test
```

Result:

```text
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

## Phase 2 Identity Module Foundation

Status: documentation started; feature implementation not started.

Foundation documents:

- `README.md`
- `docs/architecture/module-boundaries.md`
- `docs/architecture/identity-foundation.md`
- `docs/architecture/persistence-conventions.md`
- `docs/adr/ADR-001-identity-service-boundary-and-foundation.md`
- `docs/adr/ADR-002-postgresql-flyway-persistence.md`

Documented but not implemented:

- planned package boundaries
- service responsibility boundaries
- role/access foundation questions
- authentication strategy options
- cross-module authenticated user context questions
- event candidates
- PostgreSQL/Flyway persistence foundation

## Phase 3 Database Connection Foundation

Status: configured on `feat/database-connection`.

Confirmed:

- PostgreSQL is the main Identity database.
- Supabase may be used as the managed PostgreSQL provider for database hosting only.
- Supabase Auth is not part of the current Identity foundation.
- Database settings are supplied through environment variables.
- Local development may load database settings from `.env`.
- Real `.env` files are ignored by Git.
- `.env.example` is committed as the local template.
- Flyway owns schema migrations.
- Hibernate validates schema with `ddl-auto=validate`.
- Open Session In View is disabled.
- Identity uses the shared/default PostgreSQL schema.
- Flyway uses `identity_flyway_schema_history` so Identity migrations do not collide with another service's Flyway versions.
- Flyway uses `baseline-on-migrate=true` because the shared/default schema may already be non-empty.

Required environment variables:

- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`

Optional environment variables:

- `DB_POOL_MAX_SIZE`
- `DB_POOL_MIN_IDLE`
- `DB_CONNECTION_TIMEOUT_MS`
- `DB_VALIDATION_TIMEOUT_MS`
- `DB_IDLE_TIMEOUT_MS`
- `DB_MAX_LIFETIME_MS`

Supabase deployment notes:

- IPv4-only deployments should use a Supabase pooler connection string in `DB_URL`.
- Persistent Spring Boot service deployments should prefer Supavisor session mode.
- Transaction mode should be reserved for serverless or short-lived workloads.
- If transaction mode is used, prepared statements must be disabled in the JDBC URL.
- Hikari pool sizes should stay conservative across Identity and other AIASPES services.

Current migration state:

- `V1__initialize_identity_schema.sql` is intentionally comment-only.
- Flyway migration history is stored in `identity_flyway_schema_history`.
- In a non-empty shared/default schema, Flyway baselines Identity at version 1 and future application tables should start with V2.
- No application tables exist yet because the first persisted Identity model has not been finalized.

Validation evidence:

- Local `.env` exists and is ignored by Git.
- `mvn spring-boot:run` connected to PostgreSQL through `.env`.
- Flyway created `public.identity_flyway_schema_history` in the shared/default schema.
- Flyway baselined Identity at version 1 because the shared/default schema was already non-empty.
- The next Identity application migration should start at V2.
- `mvn test` passed with 1 test, 0 failures, 0 errors, 0 skipped.
- Non-escalated Maven commands still fail in this workspace because Maven Central access is sandboxed.

## Current Project State

Confirmed current state:

- This is a Spring Boot project.
- The module is at scaffold stage.
- No application identity features are implemented yet.
- No authentication approach has been finalized.
- No role persistence model has been finalized.
- No API contracts have been finalized.
- PostgreSQL connection configuration has been defined.
- No application database tables have been defined.

## Planned Items

Planned, not yet confirmed as implementation:

- User account model
- Authentication/login approach
- Role model
- Access-role model
- Student, Instructor, and Admin authorization rules
- Optional Evaluator, Adviser, and Panelist authorization rules
- Token or session strategy
- Cross-module authenticated user context for Project Intake
- Cross-service contracts with the API Gateway, Project Intake, and any event broker/message bus
- First application persistence migration
- API error/response convention

## Open Questions

- Should authentication use JWT, server sessions, an external identity provider, or another strategy?
- Should optional roles be implemented immediately or deferred?
- Should roles be single-role or multi-role per user?
- Should role assignments be global only, or scoped by course/project/module?
- Should Identity expose user profile data to other modules?
- How should Project Intake validate or consume identity/access context?
- Should backend services verify Identity-issued tokens directly or trust signed gateway-provided identity headers?
- Which Identity changes should emit events/messages for other services?
- Which Identity model should become the first persisted table?

## Next Recommended Step

Before feature implementation:

1. Decide and document the authentication/token strategy.
2. Decide and document the role/access-role model.
3. Add the first application persistence migration after the first persisted model is accepted.
4. Define the authenticated user context contract for Project Intake.
5. Add architecture tests when package boundaries are created.
6. Add or update tests alongside each implementation slice.
