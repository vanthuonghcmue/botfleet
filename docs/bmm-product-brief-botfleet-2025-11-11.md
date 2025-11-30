# Product Brief: botfleet

**Date:** 2025-11-11
**Author:** boss
**Context:** Greenfield software project (fleet management API)

---

## Executive Summary

Botfleet is a web API platform for managing a fleet of cargo trucks and their delivery orders. It provides complete order lifecycle management (from customer request to billing), role-based information access via QR codes, and real-time operational visibility for customers, drivers, admins, and sales. The MVP focuses on core order intake and pricing approval, vehicle/driver assignment and scheduling, QR-based order tracking, and role-specific dashboards.

---

## Core Vision

### Problem Statement

Small and mid-size logistics operators lack a simple, API-first system that unifies order intake, pricing approval, assignment, and real-time tracking across all stakeholders. Existing tools are fragmented or heavyweight, leading to manual coordination, missed information, and poor customer visibility.

### Problem Impact

- Time lost coordinating between sales, operations, drivers, and customers
- Data inconsistencies and errors (addresses, item details, pricing)
- Poor customer experience (limited tracking, unclear status/ETA)
- Inefficient fleet utilization and delayed billing

### Why Existing Solutions Fall Short

- Generic TMS tools are complex, expensive, and not developer-friendly
- Few systems provide role-based, QR-driven contextual access to order data
- Customer- and driver-facing experiences are often bolt-ons, not first-class
- Real-time status is typically GPS-driven only; operational check-ins are underused

### Proposed Solution

An API-first logistics platform enabling:
- Complete order lifecycle: request → sales approval → assignment → pickup → transit → delivery → billing
- Role-based QR codes so each stakeholder sees appropriate information on scan
- Driver/location check-ins at stops to update status with minimal friction
- Multi-role dashboards (customer, driver, admin, sales) for clear workflows
- Foundation for optimization (routing, capacity, pricing) in future phases

### Key Differentiators

- Role-based QR access: same code shows different data by role
- Driver-initiated location check-ins that complement GPS with operational truth
- Clean, API-first design to integrate with external portals and apps
- Pragmatic, modular MVP that grows into optimization and analytics later

---

## Target Users

### Primary Users

- Customers (shippers/consignees): Create/track orders, receive ETAs, proofs of delivery
- Drivers: View assigned stops, scan QR for order details, submit check-ins and POD
- Admin/Operations: Assign vehicles/drivers, monitor status, handle exceptions

### Secondary Users

- Sales: Evaluate feasibility, calculate/approve pricing, confirm service levels
- Accounting: Access invoice data, payment status, proofs of delivery
- Depot/Warehouse: Support pickup/drop-off, documentation and handoffs

### User Journey

1) Customer submits order request (items, pickup, delivery, service level)
2) Sales evaluates feasibility and pricing; approves or suggests alternatives
3) Operations assigns vehicle/driver and schedules pickup
4) Driver performs pickup, scans QR, and checks in at key stops
5) Customer tracks progress via link/QR; receives status and ETA updates
6) Delivery completed with POD; invoice generated and shared

---

## Success Metrics

### Business Objectives

- Improve on-time delivery rate and customer satisfaction
- Reduce coordination time between roles by 30%+
- Accelerate billing cycle time (delivery → invoice) by 40%+

### Key Performance Indicators

- Order cycle time (request → delivery)
- On-time pickup/delivery %
- Exceptions resolved within SLA %
- Time to invoice and DSO (days sales outstanding)
- User adoption/engagement by role (drivers/customers/admin/sales)

---

## MVP Scope

### Core Features

- Order Management
  - Create/update orders with full details (items, pickup, delivery, service level)
  - Sales feasibility/pricing approval workflow; pricing breakdown
- Assignment & Scheduling
  - Vehicle/driver assignment; pickup scheduling
- QR & Tracking
  - Generate per-order QR; role-based views on scan
  - Status updates: created, approved, scheduled, picked up, in transit, out for delivery, delivered
  - Driver check-ins at stops with optional photo/notes
- Proof of Delivery & Billing
  - POD capture (signature/photos/timestamp)
  - Invoice generation with final charges; payment status field
- Dashboards
  - Customer: order list/status, ETAs, POD, invoices
  - Driver: today’s stops, order details via QR, check-ins, POD
  - Admin/Sales: approvals, assignment, live status, exceptions
- API-First
  - Well-defined REST endpoints and auth with role-based access control (RBAC)

### Out of Scope for MVP

- Advanced route optimization and auto-rebalancing
- Complex pricing engines and contract management
- Deep 3rd-party integrations (ERP/TMS) beyond basic webhooks
- Mobile apps; initial UX may be responsive web + QR

### MVP Success Criteria

- End-to-end flow works for top use cases (request → delivery → invoice)
- Drivers and customers can complete tasks without training
- 90%+ of orders flow without manual coordination outside the system

### Future Vision

- Optimization (routing, capacity planning, load sharing)
- Analytics and forecasting dashboards
- Deeper integrations (ERP, accounting, telematics)
- Mobile apps for drivers/customers

---

## Technical Preferences

- API-first, modular service with clean RBAC and audit trails
- Standards-based QR generation and secure tokenized access
- Event-driven updates (webhooks/queues) for status changes
- Portable datastore (relational core for orders; consider spatial indexing later)

## Risks and Assumptions

- Driver adoption for check-ins (mitigation: ultra-simple flows, offline-friendly)
- QR scanning reliability in varied conditions (mitigation: short URLs as fallback)
- Data accuracy depends on clear role responsibilities
- Assumption: MVP can start with manual route planning before optimization

## Timeline

- MVP: 8–10 weeks (core flows + dashboards)
- Phase 2: Optimization and analytics (post-MVP)

## Supporting Materials

- Brainstorming results: docs/brainstorming-session-results-2025-11-11.md

---

_This Product Brief captures the vision and requirements for botfleet._

_It was created through collaborative discovery and reflects the unique needs of this Greenfield software project._

_Next: PRD will transform this brief into detailed product requirements._
