# ADR-0004: REST API, Transactional Outbox, and Deferred Messaging Infrastructure

Status: Accepted

## Context
HealthOS needs a stable application API and future durable cross-module/integration events, but it does not yet need distributed streaming infrastructure.

## Decision
Use versioned REST endpoints under `/api/v1` with OpenAPI documentation. Use synchronous application contracts for immediate in-process interactions. For durable asynchronous events, use a transactional outbox written in the same PostgreSQL transaction as the business change. Add Redis/BullMQ only when actual background workloads appear. Do not introduce Kafka in HOS-000.

## Consequences
Benefits: simple operations, reliable event handoff, and a clear future path to external messaging. Trade-offs: outbox processing must be idempotent and eventual-consistency behavior must be explicit when asynchronous flows are added.