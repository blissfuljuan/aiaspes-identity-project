# ADR-001: Identity Service Boundary And Foundation

Date: 2026-08-21

Status: Accepted

## Context

AIASPES uses a microservices architecture with an API Gateway and event-driven messaging. The Identity module is a separate repository and independently deployable service.

The service is currently at starter/scaffold stage. No authentication, role model, persistence model, or API contract has been implemented.

The main system documentation confirms that Identity owns identity, access, and access-role management. Project Intake should use Identity for submitting student identity and instructor/admin authorization, but Project Intake should not own login, registration, or role assignment.

## Decision

Identity will remain a separate service repository responsible for:

- user identity records
- authentication approach and implementation, once selected
- role and access-role assignment
- access checks based on Identity-owned data
- identity context exposed to other modules
- Identity-owned events when asynchronous communication is introduced

The service will use the base package:

```text
com.blissfuljuan.aiaspes.identity
```

The planned internal package structure is:

```text
api
application
domain
infrastructure
integration
```

Feature implementation remains deferred until the role model, authentication approach, access-control rules, and cross-module user context are documented and accepted.

## Consequences

Positive:

- Identity remains independently deployable.
- Project Intake can depend on Identity for identity/access context without owning Identity rules.
- The service has a clear boundary before feature implementation starts.
- Future API, persistence, and security implementation can be tested against documented package boundaries.

Tradeoffs:

- Initial progress is documentation-first rather than feature-first.
- Authentication and persistence implementation must wait for explicit decisions.
- Some details, such as single-role vs multi-role and JWT vs session, remain open until follow-up ADRs.

## Follow-Up Decisions

Planned ADRs:

- Authentication and token/session strategy
- Role and access-role model
- Persistence and migration strategy
- Cross-module authenticated user context
- API error/response convention, if shared conventions are needed
