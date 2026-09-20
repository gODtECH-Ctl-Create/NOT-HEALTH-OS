# ADR-0003: Provider-Agnostic Identity and Scoped RBAC

Status: Accepted

## Context
HealthOS requires secure authentication, MFA readiness, multi-organization membership, and branch/department authorization. Identity implementation should not be coupled to healthcare business modules.

## Decision
Use an OIDC/OAuth 2.0 compatible identity-provider boundary. HealthOS owns local user profiles, memberships, roles, permissions, and scoped role assignments. Authorization is deny-by-default and evaluates identity, organization membership, permission, scope, and module activation. Do not authorize by hard-coded role names. Select the concrete identity provider in HOS-001.

## Consequences
Benefits: provider flexibility, clean separation of authentication and authorization, and support for custom organization roles. Trade-offs: token identity and local membership state must be reconciled, and authorization requires strong negative tests.