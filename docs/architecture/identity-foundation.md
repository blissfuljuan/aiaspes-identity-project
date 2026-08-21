# Identity Foundation

Last updated: 2026-08-21

## Purpose

This document records the Identity service foundation before application features are implemented.

## Confirmed Scope

Identity owns:

- Identity management
- Access management
- Access-role management

Core roles:

- Student
- Instructor
- Admin

Optional roles:

- Evaluator
- Adviser
- Panelist

Optional roles are planned but should be deferred until a workflow explicitly needs them.

## Planned Identity Model

Planned concepts:

- User account
- Login credential or external authentication reference
- Role
- Access-role assignment
- Account status

Initial account statuses to consider:

- Active
- Disabled
- Pending activation

Open questions:

- Should a user have exactly one role or multiple roles?
- Should role assignment be global only, or can it be scoped to a course, project, panel, or module?
- Should Identity store profile fields, or only identity/account fields?

## Authentication Strategy

No authentication strategy is finalized yet.

Options to evaluate:

- JWT issued by Identity and verified by the API Gateway and backend services.
- Server-side sessions managed by Identity and mediated through the gateway.
- External identity provider with Identity owning AIASPES-specific roles and access rules.

Recommended next decision:

- Start with a JWT-based strategy unless project requirements require server-side session revocation as a first-class feature.

Rationale:

- Backend services need an authenticated user context for authorization decisions.
- Project Intake needs immediate access to submitting student identity and instructor/admin authorization context.
- JWT works naturally with an API Gateway and independently deployable backend services.

JWT details are not confirmed until documented in an ADR.

## Access-Control Foundation

Core role intent:

- Student: creates and submits proposal drafts for self or team.
- Instructor: reviews and evaluates assigned or accessible proposals.
- Admin: manages system-level users, access roles, and administrative review operations.

Planned rule style:

- Use coarse roles for broad access.
- Add scoped permissions only when a workflow requires more precision than a global role.
- Keep authorization business rules in the owning service.

Open questions:

- Can instructors view all submitted proposals or only assigned/course-scoped proposals?
- Can admins impersonate or act on behalf of users?
- Should student team membership authorization be checked by Project Intake after Identity confirms user identity?

## Cross-Module User Context

Project Intake needs a reliable current-user context.

Planned context fields:

- User ID
- Email or username
- Display name
- Roles
- Account status

Possible transport:

- Client request includes an access token.
- API Gateway validates or forwards authentication context according to the selected gateway strategy.
- Backend services verify the token or trusted gateway-provided identity context.
- Project Intake may call Identity synchronously when it needs fresh user or role data.

Open questions:

- Should backend services verify Identity-issued JWTs directly?
- Should the gateway pass signed identity headers to backend services?
- What token lifetime and refresh-token strategy should be used?
- What user fields are safe to expose across module boundaries?

## Events

Potential Identity-owned events:

- UserRegistered
- UserActivated
- UserDisabled
- UserRoleAssigned
- UserRoleRevoked

Eventing is planned but not required for the first Identity implementation slice.

Rules:

- Identity emits events only for Identity-owned state changes.
- Consumers apply their own business rules after receiving events.
- Event broker configuration must not contain business rules.

## First Implementation Candidate

After foundation decisions are accepted, the first vertical slice should be narrow:

```text
Create user account
-> assign one or more roles
-> expose current-user/read model endpoint
-> validate with unit and application tests
```

Before coding that slice, decide:

- single-role vs multi-role
- JWT vs session vs external provider
- persistence database and migration approach
- initial API response/error convention
