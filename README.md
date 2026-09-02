# FoodFlow system design

FoodFlow is a city-wide food delivery platform for customers, restaurants, and couriers. This repository contains a complete high-level design package: product scope, application flows, service architecture, data model, integration contracts, operational requirements, and an implementation plan.

## Design package

| Area | Artifact | Purpose |
|---|---|---|
| Product | [overview.md](overview.md) | Goals, actors, scope, and success criteria |
| Requirements | [requirements.md](requirements.md) | Functional requirements, business rules, lifecycle, and SLO targets |
| Architecture | [foodflow-system-architecture.excalidraw](foodflow-system-architecture.excalidraw) | Logical services, trust boundaries, data infrastructure, and integrations |
| Data | [database-model.excalidraw](database-model.excalidraw) | Core relational entities and relationships |
| Contracts | [service-contracts.md](service-contracts.md) | REST APIs, domain events, ownership, and idempotency |
| Operations | [operations.md](operations.md) | Reliability, security, observability, deployment, and recovery |
| Delivery plan | [roadmap.md](roadmap.md) | MVP slices, testing strategy, rollout gates, and open decisions |
| Customer UX | [applications/customer-app.excalidraw](applications/customer-app.excalidraw) | Customer journeys and tracking state |
| Restaurant UX | [applications/restaurant-portal.excalidraw](applications/restaurant-portal.excalidraw) | Order intake and preparation workflow |
| Courier UX | [applications/courier-app.excalidraw](applications/courier-app.excalidraw) | Offer, pickup, navigation, and delivery workflow |
| UI concept | [output/pdf/foodflow-ui-concept.pdf](output/pdf/foodflow-ui-concept.pdf) | English desktop and mobile interface concept |
| Presentation | [output/presentation/foodflow-system-design.pptx](output/presentation/foodflow-system-design.pptx) | Stakeholder-ready system design deck |

## Architecture principles

1. The order state machine is the source of truth for the end-to-end workflow.
2. Each command is idempotent and every external callback is safe to replay.
3. PostgreSQL protects money and order state; Kafka distributes durable domain events.
4. Redis accelerates reads and stores ephemeral courier positions, but is never the only system of record.
5. Third-party failures are isolated with timeouts, retries, circuit breakers, and reconciliation jobs.
6. Personally identifiable information is minimized by role and retained only for defined operational needs.

## Suggested review order

Start with `overview.md` and `requirements.md`, then review the architecture and data diagrams. Use `service-contracts.md` to validate integration boundaries, `operations.md` for production readiness, and `roadmap.md` to plan delivery.

