# Botfleet Source Tree Architecture

> **Updated**: November 2025

## 🎯 Project Structure Overview

```
botfleet/
 ⭐
├── packages/                 # Monorepo packages (to be scaffolded)
│   ├── api/                  # Fastify REST API (core domain + services)
│   ├── frontend/             # Next.js 15 dashboards (role-based)
│   ├── worker/               # BullMQ background jobs (webhooks/outbox, async tasks)
│   └── shared/               # Shared types, DTOs, validation schemas, constants
├── infra/                    # Infrastructure IaC & docker compose (future)
├── docs/                     # Product & technical documentation
│   └── coding-standards/     # Comprehensive standards
├── scripts/                  # Dev utility scripts (migrations, seeding)
└── pnpm-workspace.yaml       # Workspace manifest
```
---

**Detailed Documentation**: See [ Source Tree](source-tree.md) in comprehensive docs

### Quick Overview

```
├── packages/
│   ├── api/                               # Backend API (Fastify)
│   │   └── src/
│   │       ├── loaders/                   # App/bootstrap (DI container, plugins)
│   │       ├── config/                    # Env, constants, token policies
│   │       ├── middleware/                # Auth, RBAC, validation, error handling
│   │       ├── routes/                    # Route registration modules
│   │       ├── controllers/               # HTTP handlers (thin) 
│   │       ├── services/                  # Business logic (pricing, assignment, status)
│   │       ├── repositories/              # Data access (Sequelize)
│   │       ├── models/                    # Sequelize models & associations
│   │       ├── domain/                    # Domain objects/value types (optional)
│   │       ├── validation/                # Zod schemas & schema adapters
│   │       ├── mappers/                   # DTO/entity mapping utilities
│   │       ├── events/                    # Webhook & outbox producers
│   │       ├── webhooks/                  # Dispatch logic & retry policies
│   │       ├── security/                  # JWT, QR token creation, policies
│   │       ├── logging/                   # Pino logger factories
│   │       ├── observability/             # OpenTelemetry instrumentation
│   │       └── lib/                       # Small utilities (dates, pricing calc helpers)
│   │
│   ├── frontend/                          # Frontend (Next.js 15)
│   │   └── src/
│   │       ├── app/                       # App Router segments
│   │       │   ├── (auth)/                # Auth pages & flows
│   │       │   ├── (orders)/              # Customer/Admin order views
│   │       │   ├── (pricing)/             # Sales pricing approvals
│   │       │   ├── (assignment)/          # Vehicle / driver assignment
│   │       │   ├── (tracking)/            # Live status timelines
│   │       │   ├── (pod)/                 # POD & invoice views
│   │       │   └── (dashboard)/           # Role landing dashboards
│   │       ├── components/
│   │       │   ├── ui/                    # shadcn/ui wrappers
│   │       │   ├── orders/                # Order-specific components
│   │       │   ├── status/                # Status timeline & badges
│   │       │   ├── pricing/               # Pricing quote displays
│   │       │   ├── assignment/            # Driver/vehicle widgets
│   │       │   ├── tracking/              # Map/ETA components (stub)
│   │       │   ├── pod/                   # POD & invoice components
│   │       │   ├── layout/                # Shell, navigation, responsive layout
│   │       │   └── common/                # Shared generic components
│   │       ├── hooks/                     # React hooks (auth, polling, SSE)
│   │       ├── stores/                    # Zustand slices (session, orders)
│   │       ├── lib/                       # Client utilities (API client, formatting)
│   │       ├── validation/                # Client-side Zod schemas
│   │       └── sse/                       # SSE subscription utilities
│   │
│   ├── worker/                            # Background workers (BullMQ)
│   │   └── src/
│   │       ├── queues/                    # Queue definitions & registration
│   │       ├── processors/                # Job processors (webhook delivery, cleanup)
│   │       ├── schedulers/                # Recurring task scheduling
│   │       ├── outbox/                    # Transactional outbox polling & dispatch
│   │       ├── adapters/                  # External provider adapters (mapping, storage)
│   │       ├── config/                    # Worker-specific configuration
│   │       ├── instrumentation/           # Telemetry for jobs
│   │       └── lib/                       # Reusable worker utilities
│   │
│   └── shared/                            # Shared code
│       └── src/
│           ├── types/                     # TypeScript types & interfaces
│           ├── dto/                       # Request/response DTOs
│           ├── validation/                # Zod schemas (canonical)
│           ├── constants/                 # Status codes, role enums, token TTLs
│           ├── pricing/                   # Pricing formula helpers
│           ├── permissions/               # RBAC permission maps
│           └── utils/                     # Generic utilities
│
└── docs/
    └── coding-standards/                  # ⭐ Comprehensive standards
        ├── README.md                      # Overview
        ├── index.md                       # Navigation hub
        ├── api-standards.md               # API patterns (auth, RBAC, responses)
        ├── frontend-standards.md          # Frontend patterns
        ├── worker-standards.md            # Worker patterns
        └── shared-standards.md            # Shared patterns
```

---

## File Naming Conventions

### Quick Reference

| Package | Pattern | Example |
|---------|---------|---------|
| **API Controllers** | `orders.controller.ts` | `orders.controller.ts` |
| **API Services** | `pricing.service.ts` | `pricing.service.ts` |
| **API Repositories** | `order.repository.ts` | `order.repository.ts` |
| **API Routes** | `ordersRoute.ts` | `ordersRoute.ts` |
| **Frontend Components** | `order-card.tsx` | `order-card.tsx` |
| **Frontend Hooks** | `use-orders.ts` | `use-orders.ts` |
| **Frontend Pages** | `page.tsx` | `app/(orders)/page.tsx` |
| **Worker Processors** | `webhook-delivery.processor.ts` | `webhook-delivery.processor.ts` |
| **Shared Types** | `order.types.ts` | `order.types.ts` |

**Detailed conventions**: [docs/coding-standards/](../../docs/coding-standards/)

---

## Layering Guidelines (API)
1. Controller: Parse input → delegate to service → shape output DTO.
2. Service: Orchestrate business logic, enforce invariants, call repositories.
3. Repository: Data persistence only (no domain decisions).
4. Validation: Zod schemas reused across controller + service boundary.
5. Security: Auth (JWT) and RBAC middleware executed before controller.
6. Events: Service emits domain events → outbox → worker dispatch.


---