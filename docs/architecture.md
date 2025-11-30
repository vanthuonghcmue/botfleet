# Architecture: botfleet

Date: 2025-11-30
Author: Architect Agent
Context: Greenfield API-first logistics platform

---

## 1. Architectural Overview

Botfleet is a modular, API-first system composed of services and components that implement order lifecycle management, role-based QR access, and real-time status tracking. The design emphasizes clear boundaries, RBAC enforcement, auditability, and extensibility toward route optimization and analytics.

### Core Components
- API Gateway & Auth
- Order Service
- Pricing & Approval Service
- Assignment & Scheduling Service
- Status & Events Service
- QR View Service
- POD & Billing Service
- Webhooks Dispatcher
- Dashboards (web UI) for Customer, Driver, Admin/Sales

### Cross-Cutting Concerns
- RBAC & Policy Enforcement
- Tokenized QR Security (short-lived, role-scoped)
- Audit Logging & Event Sourcing (status/check-ins)
- Observability (logs/metrics/traces)

---

## 2. Component Responsibilities

### API Gateway & Auth
- Accepts all external requests, handles authentication (OAuth2/JWT)
- Routes to internal services; enforces coarse-grained RBAC

### Order Service
- CRUD for orders, items, stops
- Validations and preliminary charge basis data

### Pricing & Approval Service
- Feasibility checks
- Pricing calculation (base + distance + weight + surcharges)
- Approval workflow and customer notification trigger

### Assignment & Scheduling Service
- Vehicle/driver assignment
- Pickup scheduling and route reference

### Status & Events Service
- Status lifecycle handling (created → delivered)
- Driver check-ins, photos/notes attachment
- ETA computation via mapping provider
- Timeline generation for customer tracking

### QR View Service
- Generate secure tokenized QR per order
- Role-aware rendering for customers, drivers, admin/sales

### POD & Billing Service
- Capture proof-of-delivery (signature/photo/timestamp)
- Generate invoice and expose payment status fields

### Webhooks Dispatcher
- Publish key events: order.approved, order.assigned, status.updated, order.delivered

### Dashboards (Web UI)
- Customer: orders, status/ETA, POD, invoices
- Driver: today’s stops, QR scan, check-ins, POD
- Admin/Sales: approvals, assignments, live status, exceptions

---

## 3. Data Architecture

### Primary Database (Relational, e.g., Postgres)
- Core tables per PRD data model (Users, Roles, Customers, Vehicles, Drivers, Addresses, Orders, OrderItems, Stops, StatusEvents, CheckIns, QRCodes, Invoices, ProofOfDelivery)
- Indexes: Orders(status, customer_id), StatusEvents(order_id, at), Stops(order_id, type), QRCodes(order_id, token)
- Constraints: FKs, not-null, enum types for status codes
- Auditing: created_at/updated_at, actor_id on mutating actions

### Object Storage (Optional)
- Store photos (POD, check-ins), signed URLs

### Caching (Optional)
- Cache common read views (order summaries, timelines)

---

## 4. Security Model

- OAuth2/JWT for API; service tokens internally
- RBAC policies per role; field-level filtering on responses
- Tokenized QR: short-lived tokens scoped to role; refresh via authenticated sessions
- Audit logs for approvals, assignments, status changes, POD/invoice generation
- PII minimization: role-based exposure; at-rest encryption for sensitive fields

---

## 5. Integration & Events

- Mapping/Geocoding provider for ETA/distance
- Webhooks for external systems
- Event topics: order.approved, order.assigned, status.updated, order.delivered

---

## 6. Deployment & Ops

- Containerized services (API, services, dashboards)
- Environment configuration via secrets manager
- CI/CD with linting/tests; canary or rolling deploy
- Observability: structured logs + metrics; error alerting

---

## 7. Sequence Diagrams (Narrative)

### Order Lifecycle (MVP)
1. Customer creates order → Order Service validates and stores
2. Sales approves pricing → Pricing Service updates order; Webhooks notify customer
3. Admin assigns vehicle/driver → Assignment Service persists
4. Driver scans QR at pickup → QR View Service shows role data; Status Service sets picked_up
5. Driver check-ins at stops → Status Service updates events; ETA recalculated
6. Delivery completed with POD → POD & Billing Service stores signature/photo and generates invoice

---

## 8. Risks & Mitigations (Architecture)
- QR token leakage → short TTL, rotate tokens, minimal exposed data
- Event consistency → transactional outbox for webhooks/events
- Photo storage reliability → use object storage with signed URLs and retries

---

## 9. Roadmap Considerations
- Route optimization and capacity planning (Phase 2)
- Analytics dashboards and forecasting
- Deeper integrations (ERP/TMS, payments)
- Mobile apps (driver/customer)

---

This architecture translates the PRD into concrete components, data structures, and cross-cutting patterns ready for implementation.
