# ADR-0001: Start HealthOS as a Modular Monolith

Status: Accepted

## Context
HealthOS will eventually contain many clinical, administrative, financial, and operational capabilities, but the product is at foundation stage with no application code yet. Starting with microservices would add distributed transactions, service discovery, network failure modes, deployment overhead, and duplicated platform concerns before those costs are justified.

## Decision
Build the initial backend as a NestJS modular monolith inside a TypeScript monorepo. Preserve explicit module boundaries, defined contracts, domain events, and a transactional outbox so future extraction remains possible. Do not allow arbitrary cross-module table access.

## Consequences
Benefits: simpler development and deployment, easier domain evolution, and lower operational cost. Trade-off: the API initially scales as one deployable unit and boundary discipline must be enforced in code review and tests.

## Revisit when
A module develops materially different scale, isolation, deployment, or ownership requirements.