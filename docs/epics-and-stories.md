# Epics and Stories — botfleet MVP

Date: 2025-11-30
Author: PM/Architect
Status: Draft for Sprint Planning

---

## Epic 1: Core Data & Auth (RBAC Foundation)
- Goal: Establish users/roles, auth, and base entities to support all flows.
- Stories:
  - ST-101: Define initial DB schema for Users, Roles, Customers, Vehicles, Drivers, Addresses, Orders, OrderItems, Stops, StatusEvents, CheckIns, QRCodes, Invoices, ProofOfDelivery
  - ST-102: Choose runtime + framework (e.g., Node/Express or Python/FastAPI) and Postgres version; document in `architecture.md`
  - ST-103: Implement auth (JWT/OAuth2) with RBAC middleware and role-based response filtering
  - ST-104: Seed scripts for roles and sample users (customer, driver, admin, sales)
  - ST-105: Add audit fields and basic structured logging

## Epic 2: Order Intake & Pricing Approval
- Goal: Capture orders and enable sales feasibility and pricing approval.
- Stories:
  - ST-201: Implement `POST /orders` with validations (FR-01..FR-05)
  - ST-202: Implement `GET /orders` and `GET /orders/{orderId}` with RBAC filtering
  - ST-203: Implement `POST /orders/{orderId}/quote` and pricing calculation baseline
  - ST-204: Implement `POST /orders/{orderId}/approve` with feasibility reasons
  - ST-205: Webhook trigger for approval (`order.approved`)

## Epic 3: Assignment & Scheduling
- Goal: Assign vehicle/driver and schedule pickup.
- Stories:
  - ST-301: Implement `POST /orders/{orderId}/assignments` (vehicleId, driverId)
  - ST-302: Implement `POST /orders/{orderId}/schedule` (pickupTime)
  - ST-303: Update order status to `scheduled` on success; emit `order.assigned`

## Epic 4: Status & Tracking
- Goal: Manage status lifecycle and driver check-ins with timeline.
- Stories:
  - ST-401: Implement `POST /orders/{orderId}/status` (codes per FR-13)
  - ST-402: Implement `POST /orders/{orderId}/checkins` (stopId, photos[], notes)
  - ST-403: Implement `GET /orders/{orderId}/timeline` (aggregate StatusEvents)
  - ST-404: Minimal ETA computation stub; pluggable mapping provider interface

## Epic 5: QR & Role-Based Views
- Goal: Generate per-order QR and render role-aware views.
- Stories:
  - ST-501: Implement `POST /orders/{orderId}/qr` to generate tokenized link (short TTL)
  - ST-502: Implement `GET /qr/{token}` with role-aware filtering and secure token validation
  - ST-503: Short-link fallback and token rotation policy

## Epic 6: POD & Billing
- Goal: Capture proof of delivery and generate invoice.
- Stories:
  - ST-601: Implement `POST /orders/{orderId}/pod` (signature/photo/timestamp)
  - ST-602: Implement `GET /orders/{orderId}/invoice` and generation logic
  - ST-603: Store photos in object storage (stub), signed URL access

## Epic 7: Observability & Webhooks
- Goal: Ensure logs/metrics and event publishing for external systems.
- Stories:
  - ST-701: Structured JSON logging across services
  - ST-702: Webhooks dispatcher for `order.approved`, `order.assigned`, `status.updated`, `order.delivered`
  - ST-703: Transactional outbox pattern for reliability (MVP simplified)

## Epic 8: Dashboards (MVP Web UI)
- Goal: Basic role dashboards to validate flows.
- Stories:
  - ST-801: Customer dashboard (orders list, status/ETA, POD, invoices)
  - ST-802: Driver dashboard (today’s stops, QR scan access, check-ins, POD)
  - ST-803: Admin/Sales dashboard (approvals, assignments, live status, exceptions)

---

## Sequencing Guidance (Initial)
1) Epic 1 → Epic 2 → Epic 3 → Epic 4 → Epic 5 → Epic 6 → Epic 7; Epic 8 in parallel after core APIs exist.
2) Prioritize endpoints and minimal UI to exercise end-to-end flow.

## Acceptance Mapping
- AC-01 → ST-201
- AC-02 → ST-203, ST-204, ST-205
- AC-03 → ST-301, ST-302, ST-303
- AC-04 → ST-501, ST-502
- AC-05 → ST-401, ST-402
- AC-06 → ST-403, ST-404, ST-502
- AC-07 → ST-601
- AC-08 → ST-602
- AC-09 → ST-103, ST-202, ST-502

## Open Decisions to Resolve
- Multi-tenant: MVP single-tenant; document in `architecture.md`
- Pricing baseline: Define formula and example; add to PRD and tech spec
- Token policies: JWT/QR TTLs; rotation and refresh behavior
- Error handling: Retries for webhooks; idempotency keys for status updates

## Next Steps
- Review and adjust epics/stories with stakeholders
- Generate Technical Spec per epic (interfaces, models, endpoints, error cases)
- Produce `sprint-status.yaml` with Sprint 1 story selection