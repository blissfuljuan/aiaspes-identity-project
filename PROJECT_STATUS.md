# AIASPES Identity Module Status

Last updated: 2026-08-21

## Module Purpose

The Identity module is the AIASPES identity, access, and access-role module.

## Confirmed Repository

Repository:

- [aiaspes-identity](https://github.com/blissfuljuan/aiaspes-identity-project.git)

Local workspace:

```text
C:\Projects\AIASPES\aiaspes-identity
```

## Confirmed Architecture Context

This module is part of the AIASPES Microservices Architecture with an API Gateway and Event-Driven Messaging.

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

## Current Project State

Confirmed current state:

- This is a Spring Boot project.
- The module is currently at starter/scaffold stage.
- No application identity features are implemented yet.
- No authentication approach has been finalized.
- No role persistence model has been finalized.
- No API contracts have been finalized.

Observed technical baseline:

- Spring Boot parent version: 4.1.1
- Java version: 17
- Maven project
- Group ID: `com.blissfuljuan.aiaspes`
- Artifact ID: `aiaspes-identity`
- Base package: `com.blissfuljuan.aiaspes.identity`

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

## Open Questions

- Should authentication use JWT, server sessions, or another strategy?
- Should optional roles be implemented immediately or deferred?
- Should roles be single-role or multi-role per user?
- Should Identity expose user profile data to other modules?
- How should Project Intake validate or consume identity/access context?
- Which Identity changes should emit events/messages for other services?

## Continuation Notes

Before implementing features:

- Finalize the identity/access/access-role model.
- Document the selected authentication strategy.
- Define role and permission rules.
- Define the contract other modules use to identify the current user.
- Add or update tests alongside each vertical slice.
