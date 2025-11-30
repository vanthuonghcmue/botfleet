# Botfleet Backend Architecture
## Overview


## Core Design Principles

### 1. Clean Architecture Pattern
- **Domain-Driven Design**: Business logic separated from infrastructure concerns
- **Dependency Injection**: Awilix container manages service dependencies
- **Interface Segregation**: Clear contracts between layers
- **Testability**: Loose coupling enables comprehensive testing

### 2. Package-Based Architecture
- **Independent Packages**: Each package has clear responsibility boundaries
- **Shared Dependencies**: Common types and utilities in `packages/shared`
- **Service Isolation**: API, Worker, and Frontend packages operate independently
- **Scalable Deployment**: Packages can be deployed and scaled independently

## Technology Stack

### Core Technologies
- **Runtime**: Node.js with TypeScript
- **API Framework**: Fastify for high-performance REST APIs
- **Database**: PostgreSQL with Sequelize ORM
- **Queue System**: BullMQ with Redis for background processing
- **Real-time Communication**: Server-Sent Events (SSE) for efficient data streaming
- **Dependency Injection**: Awilix for clean service management
- **Observability**: Pino structured logging with OpenTelemetry tracing

### Versions (Target for Botfleet)
- **Node.js**: `25.x LTS`
- **TypeScript**: `5.6`
- **Fastify**: `5.x`
- **Sequelize**: `6.x`
- **PostgreSQL**: `15`
- **BullMQ**: `5.x`
- **Redis**: `7.x`
- **Awilix**: `8.x`
- **Zod**: `3.x`
- **Pino**: `9.x`
- **OpenTelemetry JS**: `1.x` (SDK + API)
- **Next.js**: `15.x` (Frontend package)
- **Jest**: `29.x`
- **Playwright**: `1.49`

## System Components

### 1. API Package (`~/api`)
The Fastify-based REST API serving as the system's database owner:

```typescript
// API package responsibilities
├── Authentication & Authorization (JWT + OAuth)
├── Database Operations (Sequelize models/migrations)
├── REST API Endpoints (/api/*)
├── Real-time Updates (SSE endpoints)
├── Service Layer (business logic)
├── Repository Pattern (data access)
└── Validation & Security (Zod schemas)
```

#### Key Features
- **Fastify Plugins**: Modular architecture with plugin system
- **Database Ownership**: Single source of truth for all data operations
- **Automatic DI**: Zero-configuration dependency injection with Awilix
- **Shared Types**: Type-safe API contracts using shared TypeScript interfaces
- **SSE Streaming**: Real-time monitor updates without WebSocket complexity
- **Comprehensive Validation**: Zod schemas for request/response validation
- **API Documentation**: Automated Swagger/OpenAPI documentation

#### Critical API Implementation Patterns (Sprint 1 Lessons)

**1. Fastify-Awilix Dependency Injection Requirements:**
```typescript
// ✅ CORRECT: Destructured constructor pattern
export default class AuthController {
  constructor({ authService, userRepository }: {
    authService: AuthService;
    userRepository: UserRepository;
  }) {
    this.authService = authService;
    this.userRepository = userRepository;
  }
}

// ❌ WRONG: TypeScript private parameters don't work with Awilix
export class AuthController {
  constructor(private authService: AuthService) {} // Fails at runtime
}
```

**2. Shared Type Interface Usage:**
```typescript
import { ApiSuccessResponse, LoginResponse } from '@bubobot/shared';

// ✅ Use shared interfaces for type consistency
async login(req, reply): Promise<ApiSuccessResponse<LoginResponse>> {
  return {
    success: true,
    data: result as LoginResponse,
    message: 'Login successful'
  };
}
```

**3. Route Schema Configuration:**
```typescript
// ⚠️ CRITICAL: additionalProperties required for dynamic data
schema: {
  response: {
    200: {
      type: 'object',
      properties: {
        success: { type: 'boolean' },
        data: { type: 'object', additionalProperties: true }, // REQUIRED!
        message: { type: 'string' },
      },
    },
  },
}
```

**Why This Matters:**
- Without `additionalProperties: true`, Fastify strips dynamic response properties
- This causes empty `data: {}` responses even when controller returns complete data
- JSON Schema validation is extremely strict by default

### 2. Worker Package (`~/worker`)
Background job processing with location-based distribution:

```typescript
// Worker package responsibilities
├── Monitor Execution (HTTP, TCP, DNS checks)
├── Incident Detection & Management
├── Notification Delivery (Email, SMS, Webhooks)
├── Anomaly Detection (AI/ML processing)
├── Data Cleanup & Maintenance
└── Location-Specific Processing
```

### 3. Frontend Package (`~/frontend`)
Next.js application with modern React patterns:

```typescript
// Frontend package responsibilities
├── Dashboard UI (React components)
├── Authentication Flow (OAuth + JWT)
├── Real-time Updates (SSE consumption)
├── Monitor Management (CRUD operations)
├── Team Collaboration (role-based access)
└── Status Pages (public monitoring pages)
```

### 4. Shared Package (`~/shared`)
Common types, DTOs, and validation schemas:

```typescript
// Shared package structure
├── types/
│   ├── user.types.ts
│   └── common.types.ts
├── dto/
│   ├── auth.dto.ts
│   └── validation schemas
├── constants/
│   └── error-codes.ts
└── errors/
    ├── base.error.ts
    ├── validation.error.ts
    └── business.error.ts
```

## Database Architecture

### Database Ownership Model
- **Single Owner**: API package exclusively owns database access
- **Migration Management**: Sequelize CLI handles schema evolution
- **Connection Pooling**: Optimized connections with automatic scaling
- **Transaction Management**: ACID compliance with proper transaction boundaries

### Migration Strategy
```bash
# Database operations
pnpm db:migrate           # Apply pending migrations
pnpm db:migrate:undo     # Rollback last migration
pnpm db:create           # Create database
pnpm db:seed             # Seed initial data
```

## Service Architecture

### Dependency Injection Container (Awilix)
```typescript
// container/config.ts
export const configureContainer = (container: AwilixContainer) => {
  container.register({
    // Repositories
    userRepository: asClass(UserRepository).singleton(),
    monitorRepository: asClass(MonitorRepository).singleton(),

    // Services
    authService: asClass(AuthService).singleton(),
    monitorService: asClass(MonitorService).singleton(),

    // External services
    emailService: asClass(EmailService).singleton(),
    notificationService: asClass(NotificationService).singleton(),
  });
};
```

### Service Layer Pattern
```typescript
// services/monitor.service.ts
export class MonitorService {
  constructor(
    private monitorRepository: MonitorRepository,
    private heartbeatRepository: HeartbeatRepository,
    private logger: Logger
  ) {}

  async createMonitor(data: CreateMonitorDto): Promise<Monitor> {
    this.logger.info('Creating monitor', { name: data.name });

    const monitor = await this.monitorRepository.create(data);

    // Emit event for worker processing
    await this.queueService.scheduleMonitor(monitor.id);

    return monitor;
  }
}
```

## Background Processing System

### BullMQ Queue Architecture
```typescript
// Queue structure and responsibilities
├── monitor-heartbeat-{location}
├── notification-delivery
│   ├── Email notifications
│   ├── SMS/Voice alerts
│   ├── Webhook dispatch
│   └── Integration updates
└── maintenance-tasks
    ├── Data cleanup
    ├── Report generation
    ├── Metric aggregation
    └── Health checks
```

### Queue Processing Flow
```typescript
// Job lifecycle management
```

## Real-time Communication (SSE)

### Server-Sent Events Implementation
```typescript
// SSE architecture benefits over WebSockets
├── Simpler Implementation (HTTP-based)
├── Automatic Reconnection (browser native)
├── Better Scalability (stateless)
├── Firewall Friendly (HTTP/HTTPS)
├── Built-in Browser Support
└── Load Balancer Compatible
```

### SSE Event Types
```typescript
// Real-time event streaming
interface SSEEvents {
  'notification.sent': NotificationEvent;
}
```

## Observability & Monitoring

### Structured Logging (Pino)
```typescript
// Logging configuration
├── Development: Pretty-printed console output
├── Production: JSON structured logs
├── Log Levels: trace, debug, info, warn, error, fatal
├── Request Tracing: Unique request IDs
├── Performance Metrics: Request duration tracking
└── Error Context: Stack traces with context
```

### OpenTelemetry Tracing
```typescript
// Distributed tracing setup
├── HTTP Request Tracing
├── Database Query Tracing
├── Queue Job Tracing
├── External Service Calls
├── Error Attribution
└── Performance Bottleneck Detection
```

### Health Monitoring
```typescript
// Health check endpoints
GET /health              # Overall system health
GET /health/detailed     # Component-specific status
GET /health/ready        # Kubernetes readiness probe
GET /health/live         # Kubernetes liveness probe
```

## Security Architecture

### Authentication & Authorization
```typescript
// Security stack implementation
├── JWT Tokens (stateless authentication)
├── OAuth 2.0 (Google, Microsoft integrations)
├── RBAC (role-based access control)
├── API Rate Limiting (per-user/IP limits)
├── Input Validation (Zod schema validation)
└── Security Headers (Helmet.js protection)
```

### Data Protection
```typescript
// Security measures
├── Password Hashing (bcrypt with salt rounds)
├── SQL Injection Prevention (parameterized queries)
├── XSS Protection (input sanitization)
├── CORS Configuration (restrictive policies)
├── Environment Security (secret management)
└── Audit Logging (security event tracking)
```

## Deployment Architecture

### Docker Containerization
```dockerfile
# Multi-stage build pattern
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

FROM node:18-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN pnpm install --production --frozen-lockfile
EXPOSE 4000
CMD ["node", "dist/index.js"]
```

### Service Configuration
```yaml
# docker-compose.yml structure
version: '3.8'
services:
  api:
    build: ./packages/api
    ports: ["4000:4000"]
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}

  worker-us:
    build: ./packages/worker
    environment:
      - WORKER_LOCATION=us-east-1
      - NODE_ENV=production

  frontend:
    build: ./packages/frontend
    ports: ["4001:4001"]
    environment:
      - NEXT_PUBLIC_API_URL=http://api:4000
```

## Performance Optimizations

### Database Performance
```typescript
// Optimization strategies
├── Connection Pooling (configurable pool sizes)
├── Query Optimization (indexes on frequently queried columns)
├── Eager Loading (reduce N+1 query problems)
├── Database Sharding (organization-based partitioning)
├── Read Replicas (separate read/write traffic)
└── Query Caching (Redis-based query result caching)
```

### API Performance
```typescript
// Fastify performance features
├── Schema Compilation (JSON schema pre-compilation)
├── Response Caching (intelligent cache headers)
├── Compression (gzip/brotli response compression)
├── Keep-Alive Connections (HTTP connection reuse)
├── Worker Threads (CPU-intensive task offloading)
└── Memory Management (garbage collection optimization)
```

## Scalability Patterns

### Horizontal Scaling
```typescript
// Scaling strategies
├── Stateless Design (no server-side session storage)
├── Package Independence (scale components separately)
├── Queue Distribution (location-based job routing)
├── Database Partitioning (shard by organization)
├── CDN Integration (static asset distribution)
└── Auto-scaling (container orchestration)
```

### Load Distribution
```typescript
// Traffic distribution patterns
├── API Load Balancing (round-robin/least-connections)
├── Worker Load Balancing (job queue distribution)
├── Database Load Balancing (read replica routing)
├── Geographic Distribution (multi-region deployment)
├── CDN Edge Caching (global content delivery)
└── Queue Partitioning (location-based processing)
```

## Development Workflow

### Package Development
```bash
# Development commands
pnpm dev                 # Start all packages in development
pnpm dev:api            # API package only
pnpm dev:worker         # Worker package only
pnpm dev:frontend       # Frontend package only

# Testing
pnpm test               # Run all tests
pnpm test:api           # API tests only
pnpm test:worker        # Worker tests only
pnpm test:e2e           # End-to-end tests

# Building
pnpm build              # Build all packages
pnpm build:api          # Build API package
pnpm build:worker       # Build worker package
pnpm build:frontend     # Build frontend package
```

### Quality Assurance
```typescript
// Code quality pipeline
├── TypeScript Compilation (strict mode enabled)
├── ESLint (coding standards enforcement)
├── Prettier (code formatting)
├── Jest Testing (unit and integration tests)
├── Playwright (end-to-end testing)
└── SonarQube (code quality metrics)
```