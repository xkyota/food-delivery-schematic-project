# Service contracts

## Service ownership

| Service | Owns | Key responsibility |
|---|---|---|
| Identity and Access | identities, roles, sessions | OIDC, token validation, RBAC, ownership claims |
| Restaurant and Catalog | restaurants, menus, availability | Searchable menu read model and restaurant operations |
| Order Orchestrator | orders, order items, state transitions, audit | Canonical workflow, invariants, saga coordination |
| Payment | payments, refunds, reconciliation records | Authorize, capture, void, refund, and provider webhooks |
| Dispatch and Delivery | couriers, offers, deliveries, payout intent | Match, reserve, assign, and manage delivery milestones |
| Tracking | active GPS samples and public tracking projection | Ingest location, enforce retention, publish live state |
| Notification | templates and delivery attempts | Push, SMS, email, retry, and delivery status |

Logical ownership is required even if the MVP starts in one PostgreSQL cluster. Services do not write another service's tables directly.

## Public API surface

All mutating requests accept `Idempotency-Key`. Every response includes `X-Correlation-ID`. APIs are versioned under `/v1`.

### Customer API

| Method | Path | Purpose |
|---|---|---|
| GET | `/v1/restaurants?lat=&lng=&query=` | Discover serviceable restaurants |
| GET | `/v1/restaurants/{id}/menu` | Read current menu and availability |
| POST | `/v1/checkout/quote` | Revalidate items, price, fee, and ETA |
| POST | `/v1/orders` | Create order and start payment authorization |
| GET | `/v1/orders/{id}` | Read customer-safe order projection |
| POST | `/v1/orders/{id}/cancel` | Request cancellation under current policy |
| GET | `/v1/orders/{id}/tracking` | Get tracking snapshot and WebSocket subscription token |

### Restaurant API

| Method | Path | Purpose |
|---|---|---|
| GET | `/v1/restaurant/orders?state=` | List incoming and active orders |
| POST | `/v1/restaurant/orders/{id}/accept` | Accept with preparation minutes |
| POST | `/v1/restaurant/orders/{id}/reject` | Reject with a reason |
| POST | `/v1/restaurant/orders/{id}/ready` | Mark ready for pickup |
| PATCH | `/v1/restaurant/menu-items/{id}` | Update price or availability |

### Courier API

| Method | Path | Purpose |
|---|---|---|
| POST | `/v1/couriers/me/availability` | Go online or offline |
| GET | `/v1/couriers/me/offers` | Read current time-limited offer |
| POST | `/v1/delivery-offers/{id}/accept` | Atomically claim an offer |
| POST | `/v1/deliveries/{id}/status` | Record arrival, pickup, transit, or delivery |
| POST | `/v1/deliveries/{id}/locations` | Ingest bounded, authenticated GPS samples |

### Provider callbacks

| Method | Path | Protection |
|---|---|---|
| POST | `/v1/webhooks/payments/{provider}` | Signature, event ID deduplication, timestamp tolerance |
| POST | `/v1/webhooks/messaging/{provider}` | Signature or shared-secret validation, deduplication |

## Domain event catalog

Events use an immutable envelope with `event_id`, `event_type`, `occurred_at`, `aggregate_id`, `aggregate_version`, `correlation_id`, `causation_id`, and `payload`. Consumers store processed `event_id` values in an inbox table.

| Topic | Event examples | Primary consumers |
|---|---|---|
| `order.lifecycle.v1` | `OrderCreated`, `RestaurantConfirmed`, `OrderCancelled`, `OrderDelivered` | payment, dispatch, tracking, notification, analytics |
| `payment.lifecycle.v1` | `PaymentAuthorized`, `PaymentCaptured`, `PaymentFailed`, `RefundCompleted` | order, notification, reconciliation |
| `delivery.lifecycle.v1` | `CourierAssigned`, `OrderPickedUp`, `DeliveryCompleted`, `AssignmentExpired` | order, tracking, notification, payout |
| `catalog.changes.v1` | `MenuItemChanged`, `RestaurantAvailabilityChanged` | search index, cache invalidation |
| `tracking.updates.v1` | `CourierLocationUpdated`, `EtaUpdated` | realtime gateway, tracking projection |
| `notification.requests.v1` | `NotificationRequested` | notification workers |

## Consistency strategy

- Local state and outbox event commit in one database transaction.
- Kafka delivery is at least once; consumers must be idempotent.
- Optimistic aggregate versions reject stale order transitions.
- Financial operations use a stable business idempotency key mapped to a provider idempotency key.
- The order orchestrator uses a saga: compensate by voiding authorization or issuing a refund when later steps fail.
- Reconciliation jobs compare internal payment state with provider balance and event records.

## Data model additions beyond the conceptual ER diagram

The existing ER diagram is intentionally conceptual. Production tables also require:

- `currency`, `tax_amount`, `delivery_fee`, `discount_amount`, and price snapshots;
- structured address snapshot plus geospatial point;
- `version`, `updated_at`, and soft-delete or archival policy where applicable;
- unique idempotency keys and provider event IDs;
- delivery offer expiry and assignment attempt history;
- outbox and inbox tables;
- encrypted PII fields or references to a protected customer profile;
- audit actor, source, reason, correlation ID, and metadata schema version.

