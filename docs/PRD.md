# Product Requirements Document (PRD): botfleet

Date: 2025-11-11
Version: 0.1 (MVP Draft)
Authors: boss (Product Owner)
Context: Greenfield, API-first fleet and order management platform

---

## 1. Overview

Botfleet is a web API platform to manage a fleet of cargo trucks and their delivery orders. It provides end-to-end order lifecycle management, role-based QR access to order data, and real-time operational visibility for customers, drivers, admins, and sales.

Goals:
- Centralize order information and workflows across roles
- Enable fast, accurate pricing approval and assignment
- Provide transparent, role-appropriate tracking via QR and web
- Shorten time-to-invoice with reliable proof of delivery

Out of Scope (MVP): advanced route optimization, complex contract pricing, deep 3rd-party integrations, native mobile apps

---

## 2. Scope (MVP)

Included:
- Order intake and validation
- Sales feasibility check and pricing approval
- Vehicle/driver assignment and pickup scheduling
- QR generation per order; role-based data views
- Status & check-ins at key stops (pickup, depot, delivery)
- Customer tracking link with ETA updates
- Proof of delivery capture (signature/photos/timestamp)
- Invoice generation and basic billing fields
- Role dashboards (customer, driver, admin/sales)
- API-first implementation with RBAC

Excluded (later phases): route optimization, analytics/forecasting, ERP/TMS integrations (beyond webhooks), native mobile apps

---

## 3. Users and Roles

- Customer (Shipper/Consignee): create requests, view order status/ETA, view/download POD & invoice
- Sales: evaluate feasibility, calculate/approve pricing, confirm service level
- Admin/Operations: assign vehicle/driver, schedule, monitor live status, manage exceptions
- Driver: view assigned stops, scan QR to access order/stop details, submit check-ins and POD
- Accounting (secondary): view invoices and payment status

RBAC Principle: least privilege; role-based filtering on API responses and QR views.

---

## 4. Functional Requirements (MVP)

4.1 Order Intake
- FR-01: Capture order with items: description, quantity, dimensions (L/W/H), weight, declared value, handling notes
- FR-02: Capture pickup: address, contact, time windows, access instructions
- FR-03: Capture delivery: address, recipient, preferences, signature requirement
- FR-04: Capture service level: express | standard | economy | scheduled date
- FR-05: Validate required fields and compute preliminary distance and charge basis

4.2 Sales Approval & Pricing
- FR-06: Sales can mark feasibility (feasible/unfeasible) with reasons
- FR-07: Sales can compute pricing (base + distance + weight + surcharges) and approve
- FR-08: Customer is notified of approval/quote and status updates

4.3 Assignment & Scheduling
- FR-09: Admin can assign vehicle and driver to approved order
- FR-10: System captures pickup appointment time and route plan reference

4.4 QR & Role-Based Views
- FR-11: System generates a per-order QR (tokenized secure link)
- FR-12: QR scan renders data based on authenticated/identified role:
  - Customer: current status, ETA, order summary
  - Driver: stop instructions, contact info, item handling notes
  - Admin/Sales: full operational details (except secrets)

4.5 Status, Check-Ins, and Tracking
- FR-13: Support status states: created, approved, scheduled, picked_up, in_transit, at_depot, out_for_delivery, delivered, exception
- FR-14: Drivers can submit check-ins at key stops with optional photos/notes
- FR-15: System records location/time for status events and computes ETA
- FR-16: Customers receive viewable timeline of status events

4.6 Proof of Delivery (POD) & Billing
- FR-17: Capture POD (signature photo, recipient name, timestamp, geotag optional)
- FR-18: Generate invoice (line items, taxes/surcharges, total) upon delivery
- FR-19: Expose invoice and payment status to authorized roles

4.7 Dashboards
- FR-20: Customer dashboard: order list, status, ETA, POD, invoices
- FR-21: Driver dashboard: today’s stops, QR scan access, check-ins, POD
- FR-22: Admin/Sales dashboard: approvals, assignments, live map/list status, exceptions queue

4.8 API & Security
- FR-23: REST endpoints for orders, items, assignments, status events, invoices, QR view
- FR-24: Authentication with role-based authorization and response filtering
- FR-25: Webhooks for key events: order.approved, order.assigned, status.updated, order.delivered

---

## 5. Non-Functional Requirements (MVP)

- NFR-01 Reliability: 99.5% uptime target for API
- NFR-02 Performance: p95 API latency < 300ms for core reads (< 1k orders)
- NFR-03 Security: JWT/OAuth2 support; signed tokens for QR links; audit logs for key actions
- NFR-04 Privacy: limit PII exposure by role; encrypt sensitive fields at rest
- NFR-05 Observability: structured logging; event audit; minimal metrics (requests, errors, queues)
- NFR-06 Portability: relational DB (e.g., Postgres) baseline; infrastructure-agnostic design

---

## 6. Data Model (Initial)

Entities (key fields):
- User(id, name, email, role, phone, org_id, status)
- Role(id, name: customer|driver|admin|sales|accounting, permissions[])
- Customer(id, org_name, billing_info, contacts[])
- Vehicle(id, plate, capacity_weight, capacity_volume, status)
- Driver(id, user_id, license_no, vehicle_id, status)
- Address(id, line1, line2, city, state, postal_code, country, geo)
- Order(id, customer_id, service_level, status, quote_amount, approved_by, assigned_vehicle_id, assigned_driver_id, created_at, updated_at)
- OrderItem(id, order_id, description, qty, weight, dims_lwh, value, handling_notes)
- Stop(id, order_id, type: pickup|delivery|depot, address_id, window_start, window_end, contact)
- StatusEvent(id, order_id, code, notes, at, location)
- CheckIn(id, order_id, stop_id, driver_id, at, location, photos[])
- QRCode(id, order_id, token, expires_at, last_scanned_at)
- Invoice(id, order_id, subtotal, surcharges, tax, total, currency, issued_at, paid_status)
- ProofOfDelivery(id, order_id, recipient_name, signature_photo_url, at, location, notes)

Notes:
- Keep history through StatusEvent/CheckIn; derive current state from latest event
- Tokenized QR grants time-bound, role-filtered access

---

## 7. API Surface (High-Level)

Authentication
- POST /auth/token

Orders
- POST /orders
- GET /orders
- GET /orders/{orderId}
- PATCH /orders/{orderId}

Items
- POST /orders/{orderId}/items
- DELETE /orders/{orderId}/items/{itemId}

Sales & Pricing
- POST /orders/{orderId}/approve
- POST /orders/{orderId}/quote

Assignment & Scheduling
- POST /orders/{orderId}/assignments { vehicleId, driverId }
- POST /orders/{orderId}/schedule { pickupTime }

Status & Check-Ins
- POST /orders/{orderId}/status { code, notes }
- POST /orders/{orderId}/checkins { stopId, photos[], notes }
- GET  /orders/{orderId}/timeline

QR & Tracking
- POST /orders/{orderId}/qr
- GET  /qr/{token}  (role-aware view)

POD & Invoices
- POST /orders/{orderId}/pod
- GET  /orders/{orderId}/invoice

Webhooks
- POST /webhooks (register)

---

## 8. Acceptance Criteria (MVP)

- AC-01: A customer can submit an order with required details and receive an order ID
- AC-02: Sales can approve pricing and the customer can see the approved quote
- AC-03: Admin can assign a driver/vehicle and schedule pickup
- AC-04: Driver can scan the order’s QR and see role-appropriate information
- AC-05: Driver can post check-ins and status updates from stops
- AC-06: Customer can view live status/ETA via tracking view
- AC-07: POD captured with signature/photo and timestamp at delivery
- AC-08: Invoice is generated and visible to authorized roles
- AC-09: All API responses are filtered by role; unauthorized fields are not returned

---

## 9. Risks & Mitigations

- Driver adoption of check-ins → prioritize ultra-simple flows and offline-friendly design
- QR reliability in field conditions → include short-link fallback, retry guidance
- Data accuracy/consistency → validation rules, clear role responsibilities, audit

---

## 10. Dependencies & Integrations (Future)

- Mapping/Geocoding provider (ETA, distance)
- Payment processor (post-MVP)
- ERP/Accounting export (webhooks/CSV initially)

---

## 11. Open Questions

- Which pricing formula variations are required for MVP vs later?
- Any compliance constraints (e.g., hazmat) to reflect at MVP?
- Preferred auth model (pure OAuth2/JWT, SSO later)?
- Do we need multi-tenant org boundaries at MVP?
