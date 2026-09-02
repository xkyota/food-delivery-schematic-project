# Product and system requirements

## Core journeys

### Customer order

1. Authenticate or continue with an established session.
2. Resolve a delivery address and serviceability zone.
3. Browse an available restaurant and current menu.
4. Add items to a single-restaurant cart.
5. Revalidate price, availability, fees, address, and ETA at checkout.
6. Create the order and authorize payment with an idempotency key.
7. Ask the restaurant to accept within a configurable timeout.
8. Capture payment after restaurant acceptance.
9. Dispatch a courier and stream preparation and delivery status.
10. Complete delivery, send the receipt, and reconcile payment and courier payout.

### Restaurant fulfillment

1. Receive an audible and visible new-order alert.
2. Accept with a preparation estimate or reject with a reason.
3. Update preparation status and item availability.
4. Mark the order ready only after acceptance and payment capture.
5. Hand the order to the assigned courier and record pickup.

### Courier delivery

1. Go online and publish coarse location and availability.
2. Receive one time-limited offer with pickup, drop-off distance, and estimated earnings.
3. Accept exactly one active assignment.
4. Confirm arrival, pickup, start of delivery, and delivery completion.
5. Share location only while online and during an active assignment.

## Canonical order lifecycle

```text
PENDING_PAYMENT
  -> PAYMENT_AUTHORIZED
  -> RESTAURANT_CONFIRMED
  -> PREPARING
  -> READY_FOR_PICKUP
  -> PICKED_UP
  -> IN_TRANSIT
  -> DELIVERED
```

Terminal exception states are `PAYMENT_FAILED`, `REJECTED`, and `CANCELLED`. A refund is modeled as a payment workflow linked to the order, not as a replacement for the order's terminal state.

Every transition requires:

- an allowed source state;
- an authorized actor or service;
- an idempotency key or unique provider event identifier;
- a timestamp, reason, and correlation ID in the audit trail;
- a domain event written through the transactional outbox.

## Business rules

- A cart contains items from one restaurant only.
- Menu price and item name are snapshotted into `ORDER_ITEM` at checkout.
- The delivery address and fee are snapshotted on the order.
- A restaurant cannot accept an order after its acceptance deadline.
- A courier cannot hold more active assignments than the configured capacity.
- Customers may self-cancel before restaurant confirmation. Later cancellation requires a policy decision and may trigger a partial or full refund.
- Payment authorization expires or is voided when the restaurant rejects or times out.
- Payment capture occurs once, after restaurant confirmation.
- Delivery completion requires an allowed proof method: customer confirmation, PIN, photo, or support override.
- Refund amount cannot exceed captured amount minus previous successful refunds.

## Target service levels

These are design targets to validate during load and resilience testing.

| Capability | Target |
|---|---|
| Browse and catalog availability | 99.95% monthly |
| Order command availability | 99.90% monthly |
| Order creation latency | p95 below 500 ms excluding provider time |
| Catalog read latency | p95 below 300 ms |
| Visible tracking freshness | 95% of active updates visible within 3 seconds |
| Duplicate financial effects | Zero by invariant |
| Recovery point objective | 5 minutes for transactional data |
| Recovery time objective | 30 minutes for critical ordering capability |

## Capacity inputs still to confirm

- Daily and peak orders per city.
- Peak concurrent customers, couriers, and restaurants.
- GPS update frequency and retention window.
- Average menu size and catalog update rate.
- Payment and map provider quotas.
- Legal retention periods by market.

The first performance model must be recalculated when these inputs are approved. Until then, horizontal scaling boundaries and queue-based load shedding remain mandatory.

## Acceptance criteria for MVP

- Happy-path delivery completes across all three applications.
- Retry of any command or webhook produces the same final result without duplication.
- Restaurant timeout voids authorization and notifies the customer.
- Courier rejection or timeout returns the job to dispatch without losing the order.
- Temporary notification failure never blocks the order state machine.
- Operators can trace one order across services using a correlation ID.
- Backup restore, event replay, and provider reconciliation are rehearsed before production traffic.

