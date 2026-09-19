# HealthOS

> A secure, modular healthcare operating system for connecting clinical, administrative, financial, and operational workflows across healthcare organizations.

> Built with **gODtECH FORGE** — Framework for Orchestrated Reasoning, Governance & Engineering.

## What is HealthOS?

HealthOS is a planned B2B healthcare SaaS platform for hospitals, clinics, laboratories, pharmacies, maternity homes, specialist centers, and future public healthcare organizations. It is designed as one configurable platform rather than separate products for each organization type.

## Current status

**Architecture foundation in progress.** The repository currently contains FORGE governance, product context, the HOS-000 architecture foundation, and accepted architecture decision records. Application code has not yet been scaffolded and no clinical modules are implemented.

## Product principles

- Modular rather than organization-specific forks
- Department-driven and workflow-driven experiences
- Security and auditability by design
- Configurable workflows instead of custom code per customer
- Scalable from clinics to multi-branch organizations
- Web first, with desktop/offline and mobile compatibility preserved for later phases

## Architecture

```text
Next.js web
    |
    | HTTPS / REST /api/v1
    v
NestJS modular-monolith API
    |
    +--> PostgreSQL (tenant-owned data, RLS defense in depth)
    +--> Transactional outbox
    +--> Redis/BullMQ when background work is introduced
    +--> S3/object storage when required

Authentication boundary: OIDC/OAuth 2.0 identity provider
Deployment baseline: Docker on AWS ECS/Fargate + RDS
```

## Foundation decisions

- TypeScript monorepo using pnpm workspaces
- Next.js for the primary web interface
- NestJS modular monolith for the initial backend
- PostgreSQL shared-schema multi-tenancy with immutable `organization_id` ownership
- Scoped, deny-by-default RBAC across organization, branch, and department scopes
- PostgreSQL Row Level Security as defense in depth for tenant-owned tables
- REST `/api/v1` with OpenAPI
- Transactional outbox for durable asynchronous events
- GitHub Actions for CI/CD
- OpenTelemetry as the telemetry boundary
- AWS ECS/Fargate as the initial container deployment target

See `docs/architecture/foundation.md` and `docs/decisions/` for the recorded design and trade-offs.

## Planned repository map

```text
apps/
  web/
  api/
  worker/        # only when background jobs are needed
packages/
  contracts/
  config/
  test-utils/
docs/
  architecture/
  decisions/
  specs/
.forge/
AGENTS.md
README.md
```

## Scope of HOS-000

HOS-000 establishes the application architecture, tenancy model, identity/authorization boundaries, module rules, audit strategy, API/data conventions, testing, CI/CD, observability, and deployment baseline. It does not implement patient, reception, appointment, consultation, laboratory, pharmacy, billing, or other clinical workflows.

## Roadmap

After the foundation is secured and verified, implementation proceeds through authentication, organization core, branches/departments, RBAC, audit infrastructure, notifications, module activation, onboarding/workspace provisioning, and then Phase 1 clinical modules.

## Local development

Application bootstrap commands will be documented after the HOS-000 scaffold exists. No unverified run command is documented yet.