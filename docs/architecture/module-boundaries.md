# Identity Module Boundaries

Last updated: 2026-08-21

## Purpose

This document defines the initial module boundaries for the AIASPES Identity service.

## Confirmed System Context

AIASPES uses a microservices architecture with an API Gateway and event-driven messaging.

Confirmed:

- Identity is a separate repository and independently deployable service.
- Identity owns identity, access, access-role business logic, and module-owned data.
- External/client requests should enter through the API Gateway.
- Service-to-service workflows should not be forced through the API Gateway.
- Other services may call Identity synchronously when an immediate identity or access response is required.
- Identity may publish events/messages when other services should observe Identity-owned state changes.
- The message broker/event bus transports messages only and must not contain Identity business rules.

## Confirmed Responsibilities

Identity owns:

- User identity records
- Authentication decisions, once the authentication approach is selected
- Role and access-role assignment
- Access checks that depend on Identity-owned data
- Identity context exposed to other services
- Identity-owned events, when eventing is introduced

Identity does not own:

- Project proposal drafts, submissions, or lifecycle decisions
- Project Intake team membership rules beyond identifying users and roles
- AI proposal evaluation or recommendation rules
- Instructor/Admin proposal review decisions
- Gateway routing or edge policy implementation
- Broker-level workflow or business rules

## Package Boundary Plan

The current base package is:

```text
com.blissfuljuan.aiaspes.identity
```

Planned package structure:

```text
com.blissfuljuan.aiaspes.identity
  api
  application
  domain
  infrastructure
  integration
```

Package responsibilities:

- `api`: HTTP controllers, request/response DTOs, and API error mapping.
- `application`: use cases and transaction-level orchestration inside the service.
- `domain`: Identity business concepts, invariants, and domain services.
- `infrastructure`: database, persistence, security implementation, configuration, and adapters.
- `integration`: service-to-service clients and event publishing/listening adapters.

## Dependency Direction

Planned direction:

```text
api -> application -> domain
infrastructure -> application/domain
integration -> application/domain
```

Rules:

- `domain` must not depend on `api`, `infrastructure`, or `integration`.
- `application` must not depend on `api`.
- `api` should delegate business behavior to `application`.
- `infrastructure` and `integration` should adapt external technology to service-owned application/domain contracts.
- Shared backend source from other services should not be imported into Identity.

## Enforcement Plan

Before feature implementation grows, add architecture tests to enforce:

- No outward dependencies from `domain`.
- No dependency from `application` to `api`.
- Controllers stay in `api`.
- Persistence/security adapters stay in `infrastructure`.
- Cross-service clients/events stay in `integration`.

This enforcement should be added with the first package-structure implementation.
