# Implementation roadmap

This is a planning proposal. Team size, provider onboarding, compliance review, and capacity assumptions must be confirmed before dates are committed.

## Delivery slices

### Phase 0 - Foundations

- Repository structure, CI, environments, secrets, and observability baseline.
- OIDC integration and role model.
- PostgreSQL migrations, outbox/inbox primitives, and API standards.
- Provider sandboxes for payment, maps, and notifications.
- Architecture decision records for unresolved choices.

**Exit gate:** one traced, authenticated command reaches a service, database, event bus, and test consumer in staging.

### Phase 1 - Order core

- Restaurant and menu management.
- Customer discovery, quote, checkout, and order creation.
- Restaurant accept/reject flow.
- Payment authorize, capture, void, webhook, and reconciliation.
- Order audit trail and customer status projection.

**Exit gate:** a customer can place an order, the restaurant can accept or reject it, and all payment compensations pass integration tests.

### Phase 2 - Delivery and realtime

- Courier availability and offer lifecycle.
- PostGIS candidate lookup and ETA-aware dispatch.
- Pickup and delivery milestones.
- GPS ingestion, retention, WebSocket subscriptions, and reconnect behavior.
- Notification templates and channel fallback.

**Exit gate:** an end-to-end delivery completes across all three applications under normal and retry conditions.

### Phase 3 - Production hardening

- Load, soak, chaos, restore, and failover tests.
- Security review, privacy controls, and abuse-rate limits.
- Operations dashboards, alerts, runbooks, and on-call ownership.
- Progressive rollout, support tooling, and launch rehearsal.

**Exit gate:** SLO tests pass, critical runbooks are rehearsed, and launch owners approve rollback criteria.

## Test strategy

| Layer | Focus |
|---|---|
| Unit | State transitions, pricing, cancellation, refund, and dispatch rules |
| Contract | REST schemas, Kafka event compatibility, and webhook signatures |
| Integration | PostgreSQL transactions, outbox/inbox, Redis fallback, and provider sandboxes |
| End-to-end | Customer to restaurant to courier, including reconnect and duplicate requests |
| Resilience | Dependency latency, dropped events, consumer restart, database failover, and replay |
| Performance | Peak order creation, menu reads, WebSocket fan-out, GPS ingestion, and consumer lag |
| Security | Authorization boundaries, token misuse, webhook replay, rate limiting, and PII leakage |

## Highest-risk assumptions

1. Restaurant response time and courier supply are sufficient for the promised ETA.
2. Payment authorization and capture rules fit every launch market.
3. GPS frequency balances tracking quality, battery use, privacy, and event cost.
4. One-delivery-per-courier is adequate for MVP economics.
5. Provider quotas and rate limits support peak demand.

Each assumption needs an owner, validation method, and due date before production scope is locked.

## Open decisions

- Monorepo versus service repositories.
- Shared PostgreSQL cluster with isolated schemas versus separate databases at launch.
- Dispatch scoring weights and fallback radius policy.
- Customer cancellation and restaurant compensation policy.
- Delivery proof methods by market and order value.
- GPS retention period and anonymization process.
- Primary/fallback providers for maps and notifications.
