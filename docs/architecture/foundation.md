# HealthOS Architecture Foundation

Status: Accepted for HOS-000 architecture stage

## Architectural goal

HealthOS starts as a secure, modular, multi-tenant healthcare platform that can serve a small clinic and grow to multi-branch organizations without an early microservices migration.

## Architecture style

Use a TypeScript monorepo with:
- `apps/web`: Next.js primary browser application
- `apps/api`: NestJS modular monolith
- `apps/worker`: background workers only when jobs are introduced
- `packages/contracts`: shared safe contracts
- `packages/config`: typed configuration helpers
- `packages/test-utils`: reusable test support

Use pnpm workspaces. Do not introduce microservices at the foundation stage. A module may be extracted later only for independent scale, isolation, ownership, or deployment needs.

## Core platform boundaries

HOS-000 defines shared platform boundaries only: identity/session integration, organizations, branches, departments, users and memberships, roles and permissions, module registry and activation, audit events, notification abstraction, settings, shared billing/reporting boundaries, health checks, and observability. Clinical workflows are excluded.

## Multi-tenancy

Use one PostgreSQL database with a shared schema initially. Every tenant-owned row carries an immutable `organization_id`. Branch- and department-scoped rows additionally carry `branch_id` and/or `department_id`, but those never replace the organization boundary.

The active organization is derived from authenticated membership and server-side authorization. It is never trusted solely from request data supplied by the client.

Tenant-owned data access requires tenant context at the application data-access boundary. PostgreSQL Row Level Security is the defense-in-depth baseline for tenant-owned tables. Cross-organization access is denied by default. Platform-super-admin operations use a separate privileged path and are always explicitly authorized and audited.

Do not use schema-per-tenant or database-per-tenant initially. Preserve the option for dedicated databases or tenant routing later where regulation, residency, enterprise isolation, or scale requires it.

## Organization hierarchy

`Organization -> Branch -> Department` is the core hierarchy. Users join organizations through memberships. Role assignments may be scoped to organization, branch, or department. A branch or department can never cross organization boundaries.

## Authentication and authorization

Authentication uses an OIDC/OAuth 2.0 compatible identity-provider boundary. The exact vendor is deferred to HOS-001. HealthOS owns the local user profile, memberships, roles, permissions, and authorization state.

Authorization uses scoped RBAC with deny-by-default behavior. Permissions are stable capability strings such as `organization.read`, `branch.manage`, `department.manage`, `user.manage`, `role.manage`, `audit.read`, and `module.manage`.

Authorization evaluates identity, organization membership, permission, scope, and module activation. Do not scatter hard-coded role-name checks throughout the codebase. Platform administration is a separate authorization domain from healthcare-organization administration.

## Module architecture

Each functional area is an explicit module containing its application/use-case logic, domain services, data access, controllers, authorization policies, events, and tests. Modules communicate through defined contracts or domain events and must not use arbitrary cross-module table access.

A central module registry defines module key, status, dependencies, capabilities, and core/optional classification. Organization module activation controls optional capability availability. UI visibility is never treated as an authorization control.

## Data architecture

PostgreSQL is the system of record. Use UUID identifiers, UTC timestamps, foreign keys, database constraints, reviewed migrations, and transactional writes for consistency-sensitive changes. Sensitive values are not duplicated unnecessarily.

For future offline/synchronization compatibility, externally synchronized entities should use stable identifiers, `updated_at`, and a version/concurrency field where needed.

## API and events

Initial application APIs are REST under `/api/v1`, with JSON bodies, explicit validation, OpenAPI documentation, consistent Problem Details-style errors, pagination, correlation IDs, and server-side authorization on every protected operation.

Use in-process domain events for immediate internal behavior and a transactional outbox for durable asynchronous events. Introduce Redis/BullMQ only when real background workloads such as notifications, exports, or retries exist. Do not introduce Kafka in HOS-000.

## Web architecture

Next.js is the primary operational UI. Treat the browser as untrusted; tenant isolation and authorization are enforced by the API. Avoid persistent browser storage of clinical data. Departmental workspaces should be task-oriented rather than mirror technical backend modules.

## Deployment baseline

Initial deployment target is containerized AWS:
- Docker images for web and API
- Amazon ECS on Fargate
- Application Load Balancer
- Amazon RDS for PostgreSQL
- Amazon S3 when object/document storage is needed
- AWS Secrets Manager for secrets
- AWS KMS-backed encryption
- CloudWatch plus OpenTelemetry for telemetry
- Redis/ElastiCache only when jobs or caching justify it

Use separate `dev`, `staging`, and `production` environments. Kubernetes/EKS is not required initially and remains a later option if scale or platform-operating needs justify it.

## CI/CD and testing

Use GitHub Actions. Pull requests run applicable linting, type checking, unit tests, integration tests, authorization/tenant-isolation tests, production builds, migration validation, and security/dependency checks.

Testing layers: unit, PostgreSQL integration, API boundary tests, browser end-to-end tests, migration tests, and mandatory negative tests proving cross-tenant access is denied.

## Observability

Use structured logs with request/correlation IDs and OpenTelemetry instrumentation. Logs must not contain patient records, credentials, tokens, or avoidable sensitive fields. Track availability, latency, error rate, database health, auth failures, audit pipeline health, job failures when applicable, and backup health.

## Audit architecture

Audit events are separate from ordinary logs and are append-oriented. Record actor ID, organization ID, branch/department scope where relevant, action, resource type and ID, outcome, timestamp, request/correlation ID, and controlled source metadata. Do not copy complete clinical payloads into audit records.

## Deliberate deferrals

HOS-000 does not finalize the identity vendor, offline conflict-resolution algorithm, regional compliance mapping, pricing implementation, organization verification policy details, mobile scope, or clinical module schemas/workflows.

## Future service extraction

Keep a module in the modular monolith unless it develops an independent scale profile, deployment cadence, materially stronger isolation requirement, separate operational ownership, or an integration workload that would destabilize the core application.