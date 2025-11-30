# Botfleet Technology Stack

> **Updated**: November 2025
> **Status**: Botfleet is the active platform

## 🎯 Quick Overview

**Platform (Active Development):**
- **API**: Fastify v5 + Sequelize v6 + Awilix v10 + Pino v9 + OpenTelemetry 1.x
- **Frontend**: Next.js 15 + React 19 + TanStack Query + TailwindCSS
- **Worker**: BullMQ (Redis-backed, location-based)
- **Shared**: TypeScript 5.6 + Zod validation
- **Database**: PostgreSQL 15
- **Cache/Queue**: Redis 7.x
- **Real-time**: Server-Sent Events (SSE)

---

## V2 Technology Stack (Primary)

### Package Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   @botfleet/    │    │   @botfleet/    │    │   @botfleet/    │
│    frontend     │────│      api        │────│     worker      │
│                 │    │                 │    │                 │
│ Next.js 15      │    │ Fastify v5      │    │ BullMQ (Redis)  │
│ React 19        │    │ Sequelize v6    │    │ Location-based  │
│ TanStack Query  │    │ Awilix DI       │    │ Processing      │
│ shadcn/ui       │    │ Pino + OTel     │    │ Maintenance     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   @botfleet/    │
                    │     shared      │
                    │                 │
                    │ TypeScript      │
                    │ Zod Validation  │
                    │ < 50KB Bundle   │
                    └─────────────────┘
```

### Frontend Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | Next.js | 15.x | React framework with App Router |
| **UI Library** | React | 19.x | Component library |
| **UI Components** | shadcn/ui | Latest | Accessible component library |
| **Styling** | TailwindCSS | 4.x | Utility-first CSS |
| **State (Server)** | TanStack Query | 5.x | Server state management |
| **State (Client)** | Zustand | 5.x | Minimal client state |
| **Forms** | React Hook Form | 7.x | Form state management |
| **Validation** | Zod | 3.x | Schema validation |
| **Icons** | Lucide React | Latest | Icon library |

### Backend API Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | Fastify | 5.x | High-performance web framework |
| **ORM** | Sequelize | 6.x | Database ORM |
| **DI Container** | Awilix | 10.x | Dependency injection |
| **Validation** | Zod | 3.x | Runtime validation |
| **Logging** | Pino | 9.x | Structured JSON logging |
| **Observability** | OpenTelemetry | 1.x | Tracing and metrics |
| **JWT** | @fastify/jwt | Latest | Token authentication |
| **Cookies** | @fastify/cookie | Latest | HTTP-only cookies |
| **CORS** | @fastify/cors | Latest | Cross-origin support |
| **Rate Limit** | @fastify/rate-limit | Latest | API rate limiting |

### Worker Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Queue** | BullMQ | 5.x | Redis-based job queue |
| **Redis Client** | ioredis | 5.x | Redis connection |
| **Database** | Sequelize | 6.x | Shared with API |
| **Logging** | Pino | 9.x | Structured logging |

### Infrastructure

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Database** | PostgreSQL | 15 | Relational database |
| **Cache** | Redis | 7.x | Session + queue storage |
| **Metrics** | InfluxDB | 2.x | Time-series monitoring data |
| **Containerization** | Docker | 24.x | Application containers |
| **Package Manager** | pnpm | 9.x | Fast package management |
| **Monorepo** | pnpm workspaces + optional Turborepo | Latest | Monorepo management |

---



---

## Development Tools

### Code Quality
- **TypeScript** - Strict mode enabled
- **ESLint** - Code linting
- **Prettier** - Code formatting
- **Husky** - Git hooks

### Testing
- **Jest** - Unit + integration tests
- **Playwright** - E2E testing
- **Supertest** - API testing

### DevOps
- **Docker** - Containerization
- **Docker Compose** - Local development
- **GitHub Actions** - CI/CD

---

## Version Requirements

```json
{
  "engines": {
    "node": ">=25.0.0",
    "pnpm": ">=9.0.0",
    "postgres": ">=15.0",
    "redis": ">=7.0.0"
  }
}
```

---

## Browser Support (Frontend)

- **Chrome**: 100+
- **Firefox**: 100+
- **Safari**: 16+
- **Edge**: 100+
- **Mobile Safari**: iOS 16+
- **Chrome Mobile**: Android 12+

---

## 📚 Detailed Documentation

### Documentation
- **Coding Standards**: [/docs/coding-standards/](/docs/coding-standards/)
