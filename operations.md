# Reliability, security, and operations

## Failure strategy

| Failure | User-visible behavior | Platform response |
|---|---|---|
| Payment provider timeout | Checkout remains pending; no duplicate retry by the client | Query provider by idempotency key, then retry or fail deterministically |
| Restaurant does not respond | Order expires with a clear message | Void authorization, emit timeout event, audit outcome |
| No courier accepts | ETA is revised; restaurant continues preparing under policy | Expand search radius, retry in waves, escalate to operations |
| Maps provider unavailable | Last known ETA is shown with reduced confidence | Use cached geocode/routes where safe, circuit-break provider calls |
| Notification provider unavailable | In-app state remains authoritative | Queue retry, fall back by channel, move poison messages to DLQ |
| Kafka consumer failure | Commands continue while projections may lag | Restart from committed offset, replay safely through inbox deduplication |
| Redis unavailable | Browse and live tracking degrade | Fall back to database for essential reads; never lose canonical state |
| Database primary failure | Short write interruption | Automatic failover to replica, then consistency and replay checks |

## Reliability controls

- Timeouts on every network call; bounded exponential backoff with jitter.
- Circuit breakers and concurrency limits around each third-party provider.
- Transactional outbox for produced events and inbox deduplication for consumers.
- Dead-letter queues with reason, first failure, last failure, and replay owner.
- Health checks distinguish process health, dependency readiness, and traffic eligibility.
- Graceful shutdown stops new work, drains requests, and commits consumer offsets.
- Reconciliation jobs for payments, refunds, courier payouts, and incomplete orders.
- Point-in-time database recovery, tested restore procedures, and cross-zone replicas.

## Security and privacy

- OIDC authorization code flow with PKCE for mobile and web clients.
- Short-lived access tokens and refresh-token rotation.
- RBAC plus object ownership: customers see their orders, restaurants see their orders, couriers see assigned delivery data only.
- TLS at every boundary; managed encryption at rest; secrets stored outside images and source control.
- Payment card details remain with the payment provider; FoodFlow stores provider tokens and non-sensitive metadata only.
- Signed webhooks, replay protection, rate limits, payload limits, and schema validation.
- PII redaction in logs and minimal courier-visible customer data.
- Audit records for privileged reads, manual state changes, refunds, and support overrides.
- Defined retention and deletion workflows per market before launch.

## Observability

Every request and event carries a correlation ID. OpenTelemetry traces connect client command, database transaction, outbox publication, consumer processing, and provider call.

### Golden signals

- Traffic: requests, orders created, event throughput, GPS update rate.
- Errors: failed commands, rejected state transitions, provider errors, DLQ depth.
- Latency: API p50/p95/p99, Kafka consumer lag, payment and map provider time.
- Saturation: database connections, CPU, memory, queue age, Redis memory, pod throttling.

### Business invariants

- Orders stuck in a non-terminal state beyond the expected window.
- Captured payment without restaurant confirmation.
- Delivered order without capture or assigned delivery.
- Refund total above captured amount.
- Courier assigned to conflicting active deliveries.

## Deployment model

- Containerized services deployed to Kubernetes across at least two availability zones.
- API gateway and realtime gateway scale independently from workers.
- PostgreSQL, Kafka, and Redis use managed offerings where available.
- Separate development, staging, and production environments and credentials.
- Infrastructure and alerts are versioned; schema changes use backward-compatible expand/migrate/contract steps.
- Deployments use canary or progressive rollout with automatic rollback on SLO regression.

## Recovery runbooks required before launch

1. Database failover and point-in-time restore.
2. Kafka consumer replay from a safe offset.
3. Payment reconciliation and manual refund approval.
4. DLQ inspection, repair, and replay.
5. Provider outage mode and traffic restoration.
6. Leaked credential rotation.
7. Stuck order diagnosis using one correlation ID.

