# Project overview

FoodFlow is a city-wide food delivery platform connecting customers, restaurants, and couriers through one reliable order workflow.

## Product goal

Let a customer discover food, pay, and follow a delivery in real time while giving restaurants and couriers clear operational tools. The platform must keep money, order status, and delivery ownership consistent even when mobile clients disconnect or external providers temporarily fail.

## Actors

- **Customer:** searches restaurants, creates and pays for an order, follows delivery progress, cancels when allowed, and requests support.
- **Restaurant employee:** accepts or rejects an order, updates preparation time, manages availability, and marks an order ready.
- **Courier:** receives delivery offers, navigates to pickup and drop-off, and records delivery milestones.
- **Support operator:** inspects the audit trail, issues permitted refunds, and resolves exceptions.
- **Platform operator:** monitors service health, replays failed events, and manages configuration safely.

## In scope

- Customer, restaurant, and courier applications.
- Identity, restaurant catalog, cart and checkout, order orchestration, payments and refunds.
- Automatic courier matching using location, availability, distance, and ETA.
- Real-time order and courier tracking.
- Push, SMS, and email notifications.
- Auditing, reconciliation, observability, backup, and recovery.

## Deferred beyond the first release

- Restaurant advertising marketplace and sponsored ranking.
- Multi-restaurant carts.
- Scheduled and group orders.
- Loyalty points and subscription plans.
- Advanced route batching for multiple concurrent deliveries.

## Definition of success

- Users can complete the happy path from discovery to delivery without manual intervention.
- Duplicate requests, events, or provider webhooks do not create duplicate charges or orders.
- An external payment, map, or notification outage degrades gracefully and can be reconciled.
- Every order transition is attributable, timestamped, and observable.
- Access is restricted by role and resource ownership.
