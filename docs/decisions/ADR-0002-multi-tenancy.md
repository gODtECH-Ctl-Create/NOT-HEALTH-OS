# ADR-0002: Shared-Database Multi-Tenancy with Defense in Depth

Status: Accepted

## Context
HealthOS must support many healthcare organizations while preventing cross-organization exposure. Database-per-tenant and schema-per-tenant would add provisioning and migration complexity too early.

## Decision
Use one PostgreSQL database and shared schema initially. Every tenant-owned row carries immutable `organization_id`. Resolve tenant context from authenticated membership. Enforce tenancy in application data access and with PostgreSQL Row Level Security on tenant-owned tables. Branch and department scopes remain subordinate to organization scope. Platform-super-admin access uses an explicit privileged and audited path.

## Consequences
Benefits: efficient onboarding, centralized migrations, lower operating cost, and testable isolation. Trade-offs: transaction/session context and RLS must be implemented carefully and tested aggressively.

## Revisit when
Regulation, residency, very large tenants, or performance isolation justify tenant routing or dedicated databases.