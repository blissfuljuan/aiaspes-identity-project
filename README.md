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
- Persistence schema
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
- `docs/adr/ADR-001-identity-service-boundary-and-foundation.md` records the first architecture decision.
