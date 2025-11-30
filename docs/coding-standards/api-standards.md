# 🚀 API Package Coding Standards

> **Package**: `@botfleet/api`
> **Framework**: Fastify v5
> **Last Updated**: November 2025

## 📋 Table of Contents

1. [General Coding Standards](#general-coding-standards)
2. [Naming Conventions](#naming-conventions)
3. [Import/Export Standards](#importexport-standards)
4. [Package Structure](#package-structure)
5. [BaseController Pattern](#basecontroller-pattern)
6. [Controller Patterns](#controller-patterns)
7. [Service Patterns](#service-patterns)
8. [Repository Patterns](#repository-patterns)
9. [Dependency Injection](#dependency-injection)
10. [Validation](#validation)
11. [SSE Implementation](#sse-implementation)
12. [Database Standards](#database-standards)
13. [Error Handling](#error-handling)
14. [Testing](#testing)
15. [Logging Standards](#logging-standards)
16. [Architecture Improvements](#architecture-improvements)

---

## 1. General Coding Standards

### Basic Principles:

- Use English for all code and documentation.
- Always declare the type of each variable and function (parameters and return value).
- Avoid using any.
- Create necessary types.
- Use JSDoc to document public classes and methods.
- Don't leave blank lines within a function.
- One export per file.

Repository Pattern:
Database Query → returns MonitorAbnormallyEntity (snake_case)
↓
transformToCamelCase() → converts to Anomaly (camelCase API type)
↓
Service/Controller Layer → uses Anomaly type

### TypeScript Best Practices

#### ✅ DO:

```typescript
// Use explicit return types for public methods
async createMonitor(data: CreateMonitorDto): Promise<MonitorDto> {
  return await this.monitorRepository.create(data);
}

// Use type-only imports when possible
import type { Logger } from '@/lib/logger';
import type { FastifyRequest, FastifyReply } from 'fastify';

// Use strict TypeScript settings
interface UserProfile {
  readonly id: number;
  readonly email: string;
  firstName?: string;  // Use optional properties, not undefined unions
}

// Prefer const assertions for immutable data
const STATUS_CODES = {
  OK: 200,
  CREATED: 201,
  NOT_FOUND: 404,
} as const;

// Use meaningful generic constraints
interface Repository<T extends BaseEntity> {
  findById(id: number): Promise<T | null>;
}
```

#### ❌ DON'T:

```typescript
// Don't use 'any' type
function processData(data: any): any {
    // ❌
    return data.someProperty;
}

// Don't mix union types with undefined
interface User {
    name: string | undefined; // ❌ Use optional instead
}

// Don't use function declarations in modules
function helper() {} // ❌ Use const helper = () => {}

// Don't use default exports in services/repositories
export default class UserService {} // ❌ Use named exports
```

### Function and Method Standards

#### ✅ DO:

```typescript
// Keep functions under 20 lines
async validateAndCreateUser(data: CreateUserDto): Promise<User> {
  await this.validateUserData(data);
  return await this.createUser(data);
}

// Single responsibility principle
private async validateUserData(data: CreateUserDto): Promise<void> {
  if (!data.email) throw new BadRequestError('Email required');
  if (await this.userExists(data.email)) {
    throw new ConflictError('User already exists');
  }
}

// Use descriptive parameter names
async findUsersByRole(
  organizationId: number,
  role: UserRole,
  includeInactive = false
): Promise<User[]> {
  // Implementation
}

// Document complex business logic
/**
 * Calculates monitor health score based on:
 * - Uptime percentage (70% weight)
 * - Response time consistency (20% weight)
 * - Error rate (10% weight)
 */
calculateHealthScore(metrics: MonitorMetrics): number {
  // Implementation
}
```

#### ❌ DON'T:

```typescript
// Don't create overly long functions
async processUserRegistration(data: any) { // ❌ 50+ lines
  // Validation logic
  // Email sending logic
  // Database operations
  // Audit logging
  // Cache updates
  // External API calls
  // ... many more responsibilities
}

// Don't use unclear parameter names
async find(id: number, x: boolean, y: string): Promise<any> { // ❌
}

// Don't skip error handling
async createUser(data: CreateUserDto) {
  const user = await this.userRepository.create(data); // ❌ No error handling
  return user;
}
```

### Code Organization Principles

#### SOLID Principles Application:

```typescript
// Single Responsibility ✅
class EmailService {
    async sendEmail(to: string, subject: string, body: string): Promise<void> {
        // Only handles email sending
    }
}

// Dependency Inversion ✅
class UserService {
    constructor(
        private userRepository: IUserRepository, // Depends on interface
        private emailService: IEmailService
    ) {}
}

// Interface Segregation ✅
interface Readable {
    read(): Promise<string>;
}

interface Writable {
    write(data: string): Promise<void>;
}

// Don't force clients to depend on unused methods
class FileHandler implements Readable, Writable {
    // Both interfaces are used
}
```

---

## 2. Naming Conventions

### File Naming

- Use UPPERCASE for environment variables.
- Avoid magic numbers and define constants.
- Start each function with a verb.
- Use verbs for boolean variables. Example: isLoading, hasError, canDelete, etc.
- Use complete words instead of abbreviations and correct spelling.
    - Except for standard abbreviations like API, URL, etc.
    - Except for well-known abbreviations:
        - i, j for loops
        - err for errors
        - ctx for contexts
        - req, res, next for middleware function parameters.

```bash
# Controllers: kebab-case with .controller.ts suffix
monitor.controller.ts
two-fa.controller.ts
user-profile.controller.ts

# Services: kebab-case with .service.ts suffix
monitor.service.ts
email.service.ts
user-context.service.ts

# Repositories: kebab-case with .repository.ts suffix
monitor.repository.ts
user.repository.ts

# Routes: kebab-case with Route/Routes suffix
monitors-route.ts
auth-routes.ts

# Middleware: kebab-case with .middleware.ts suffix
authentication.middleware.ts
rate-limiting.middleware.ts

# Types: kebab-case with .types.ts suffix
monitor.types.ts
user.types.ts

# Utilities: kebab-case
date.utils.ts
crypto.utils.ts
```

### Class Naming

```typescript
// Controllers: PascalCase + Controller suffix
export class MonitorController extends BaseController {}
export class TwoFAController extends BaseController {}
export class UserProfileController extends BaseController {}

// Services: PascalCase + Service suffix
export class MonitorService extends BaseService {}
export class EmailService {}
export class UserContextService {}

// Repositories: PascalCase + Repository suffix
export class MonitorRepository extends BaseRepository {}
export class UserRepository extends BaseRepository {}

// Models: PascalCase (entity name)
export class Monitor extends Model {}
export class User extends Model {}
export class OrganizationTeam extends Model {}

// Error Classes: PascalCase + Error suffix
export class ValidationError extends AppError {}
export class MonitorNotFoundError extends NotFoundError {}

// Middleware: PascalCase + Middleware suffix
export class AuthenticationMiddleware {}
export class RateLimitingMiddleware {}
```

### Interface and Type Naming

```typescript
// Interfaces: PascalCase + descriptive suffix
interface CreateMonitorRequest {}
interface MonitorFilters {}
interface PaginationOptions {}
interface ApiSuccessResponse<T> {}

// Types: PascalCase
type MonitorStatus = 'active' | 'paused' | 'deleted';
type UserRole = 'admin' | 'member' | 'viewer';

// Constants: SCREAMING_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_TIMEOUT_MS = 5000;
const SUPPORTED_MONITOR_TYPES = ['http', 'tcp', 'dns'] as const;

// Enums: PascalCase
enum MonitorType {
    HTTP = 'http',
    TCP = 'tcp',
    DNS = 'dns',
}
```

### Variable and Function Naming

- Write short functions with a single purpose. Less than 20 instructions.
- Name functions with a verb and something else.
    - If it returns a boolean, use isX or hasX, canX, etc.
    - If it doesn't return anything, use executeX or saveX, etc.
- Avoid nesting blocks by:
    - Early checks and returns.
    - Extraction to utility functions.
- Use higher-order functions (map, filter, reduce, etc.) to avoid function nesting.
- Use arrow functions for simple functions (less than 3 instructions).
- Use named functions for non-simple functions.
- Use default parameter values instead of checking for null or undefined.
- Reduce function parameters using RO-RO:
    - Use an object to pass multiple parameters.
    - Use an object to return results.
- Declare necessary types for input arguments and output.
- Use a single level of abstraction.

```typescript
// Variables: camelCase
const userId = request.user.id;
const monitorFilters = { status: 'active' };
const isEmailVerified = user.emailVerifiedAt !== null;

// Functions: camelCase + descriptive verbs
async function createMonitor(data: CreateMonitorDto): Promise<Monitor> {}
function validateEmailFormat(email: string): boolean {}
function calculateUptime(heartbeats: Heartbeat[]): number {}

// Boolean variables: is/has/can/should prefix
const isActive = monitor.status === 'active';
const hasPermission = user.role === 'admin';
const canEditMonitor = checkEditPermissions(user, monitor);
const shouldSendAlert = monitor.alertsEnabled && incident.severity === 'critical';

// Database fields: snake_case (matches SQL conventions)
interface UserEntity {
    user_id: number;
    created_at: Date;
    updated_at: Date;
    email_verified_at: Date | null;
}

// API fields: camelCase (matches JavaScript conventions)
interface UserDto {
    userId: number;
    createdAt: string;
    updatedAt: string;
    emailVerifiedAt: string | null;
}
```

### Route and Endpoint Naming

```typescript
// REST endpoints: kebab-case, plural nouns for collections
GET    /api/teams/:teamId/monitors
POST   /api/teams/:teamId/monitors
GET    /api/teams/:teamId/monitors/:id
PUT    /api/teams/:teamId/monitors/:id
DELETE /api/teams/:teamId/monitors/:id

// Action endpoints: kebab-case verbs
POST   /api/teams/:teamId/monitors/:id/pause
POST   /api/teams/:teamId/monitors/:id/resume
POST   /api/auth/reset-password
GET    /api/teams/:teamId/monitors/:id/health-check

// Nested resources: logical hierarchy
GET    /api/teams/:teamId/monitors/:id/incidents
GET    /api/teams/:teamId/monitors/:id/heartbeats
GET    /api/teams/:teamId/incidents/:id/timeline
```

---

## 3. Import/Export Standards

### Import Organization

```typescript
// 1. Node.js built-in modules
import { randomUUID } from 'crypto';
import { readFile } from 'fs/promises';

// 2. External dependencies (alphabetical)
import { FastifyReply, FastifyRequest } from 'fastify';
import { Model, DataTypes } from 'sequelize';
import { z } from 'zod';

// 3. Internal shared packages
import { BadRequestError, MonitorFilters, CreateMonitorDto, validateMonitorData } from '@botfleet/shared';

// 4. Internal relative imports (@ alias preferred)
import { MonitorService } from '@/services';
import { MonitorRepository } from '@/repositories';
import type { Logger } from '@/lib/logger';

// 5. Type-only imports (separate and last)
import type { AuthenticatedUser } from '@/types';
```

### Export Standards

```typescript
// ✅ Prefer named exports
export class MonitorController extends BaseController {}
export class MonitorService {}
export class MonitorRepository {}

// ✅ Export types and interfaces
export interface CreateMonitorRequest {
    name: string;
    url: string;
    interval: number;
}

export type MonitorStatus = 'active' | 'paused' | 'deleted';

// ✅ Export constants
export const MAX_MONITORS_PER_TEAM = 100;
export const DEFAULT_CHECK_INTERVAL = 300;

// ✅ Use default export only for main module entry
// app.ts, server.ts, or single-purpose utilities
export default function createServer() {
    // Application factory function
}

// ❌ Avoid default exports for classes/services
export default class MonitorService {} // ❌ Use named export
```

### Re-export Patterns

```typescript
// services/index.ts - Clean barrel exports
export { MonitorService } from './monitor.service';
export { UserService } from './user.service';
export { EmailService } from './email.service';
export { SSEService } from './sse.service';

// Don't export everything with wildcard
export * from './monitor.service'; // ❌ Too broad

// types/index.ts - Export commonly used types
export type { CreateMonitorRequest, MonitorFilters, MonitorStatus } from './monitor.types';

export type { User, CreateUserRequest } from './user.types';
```

---

## 4. Package Structure

```
packages/api/
├── src/
│   ├── controllers/         # HTTP handlers (thin layer)
│   ├── services/           # Business logic
│   ├── repositories/       # Data access layer
│   ├── models/            # Sequelize models
│   ├── middleware/        # Fastify hooks/middleware
│   ├── routes/           # Route definitions
│   ├── loaders/         # App initialization
│   ├── lib/             # Utilities and helpers
│   └── types/          # TypeScript definitions
├── migrations/         # Database migrations
├── seeds/             # Database seeds
└── tests/            # Test files
```

### File Naming Convention

```typescript
// Controllers: [entity].controller.ts
monitor.controller.ts;
auth.controller.ts;

// Services: [entity].service.ts
monitor.service.ts;
auth.service.ts;

// Repositories: [entity].repository.ts
monitor.repository.ts;
user.repository.ts;

// Routes: [entity]Route.ts or [entity]Routes.ts
monitorsRoute.ts;
authRoute.ts;
```

---

## 5. BaseController Pattern

All controllers **MUST** extend `BaseController` to ensure consistency and reduce code duplication.

### BaseController Benefits

- **Consistency**: All controllers use the same patterns for common operations
- **Type Safety**: Proper TypeScript typing for request/response handling
- **Error Handling**: Standardized error responses and validation
- **Logging**: Structured logging with correlation IDs
- **Code Reduction**: ~60% less boilerplate code per controller
- **Maintainability**: Changes to common patterns only need to be made once

### Using BaseController

```typescript
// monitors.controller.ts
import { MonitorService } from '@/services';
import { MonitorFilters } from '@botfleet/shared';
import { FastifyReply, FastifyRequest } from 'fastify';
import { BaseController } from './base.controller';

export class MonitorController extends BaseController {
    private monitorService: MonitorService;

    constructor({ monitorService }: { monitorService: MonitorService }) {
        super(); // ✅ REQUIRED: Call super()
        this.monitorService = monitorService;
    }

    async list(
        request: FastifyRequest<{
            Params: { teamId: string };
            Querystring: { page?: number; limit?: number; status?: string };
        }>,
        reply: FastifyReply
    ) {
        // ✅ Use BaseController methods
        const teamId = this.getTeamId(request);
        const { page, limit, status } = request.query;

        this.logOperation(request, 'list', { teamId, filters: request.query });

        const filters: MonitorFilters = {
            ...(status && { status }),
        };

        const { page: pageNum, limit: limitNum } = this.parsePagination({ page, limit });

        const result = await this.monitorService.listMonitors(teamId, filters, pageNum, limitNum);

        return this.sendPaginatedSuccess(reply, result);
    }

    async create(
        request: FastifyRequest<{
            Params: { teamId: string };
            Body: CreateMonitorRequest;
        }>,
        reply: FastifyReply
    ) {
        const teamId = this.getTeamId(request);
        const userId = this.getUserId(request);
        const data = this.getRequestBody<CreateMonitorRequest>(request);

        this.logOperation(request, 'create', { teamId, userId });

        const monitor = await this.monitorService.createMonitor(teamId, data);

        return this.sendCreated(reply, monitor, 'Monitor created successfully');
    }
}
```

### Migration Checklist

When creating a new controller or updating an existing one:

- [ ] Extend `BaseController`
- [ ] Call `super()` in constructor
- [ ] Use `this.getTeamId(request)` instead of manual parsing
- [ ] Use `this.getUserId(request)` for authentication
- [ ] Use `this.logOperation()` for structured logging
- [ ] Use `this.sendSuccess()` or `this.sendPaginatedSuccess()` for responses
- [ ] Use `this.parsePagination()` for pagination parameters
- [ ] Use `this.getNumericRouteParam()` for route parameter validation

---

## 6. Controller Patterns

Controllers should be **thin** - only handle HTTP concerns, delegate business logic to services.

### Controller Template

```typescript
/**
 * MonitorController - HTTP handlers for monitor endpoints
 *
 * ARCHITECTURE: Controllers are thin HTTP handlers that:
 * 1. Parse request parameters
 * 2. Call service methods
 * 3. Format responses
 *
 * NO business logic in controllers!
 */
import { FastifyRequest, FastifyReply } from 'fastify';
import { MonitorService } from '@/services';

export default class MonitorController {
    private monitorService: MonitorService;

    constructor({ monitorService }: { monitorService: MonitorService }) {
        this.monitorService = monitorService;
    }

    /**
     * List monitors with filters
     * GET /api/teams/:teamId/monitors
     */
    async list(
        request: FastifyRequest<{
            Params: { teamId: number };
            Querystring: { page?: number; limit?: number };
        }>,
        reply: FastifyReply
    ) {
        const { teamId } = request.params;
        const { page = 1, limit = 50 } = request.query;

        // NO try-catch! Fastify handles errors
        const result = await this.monitorService.listMonitors(teamId, page, limit);

        return reply.send({
            success: true,
            data: result.data,
            meta: result.meta,
        });
    }
}
```

### Controller Rules

#### ✅ DO:

```typescript
// Let Fastify handle errors - no try-catch
async getById(request: FastifyRequest, reply: FastifyReply) {
  const monitor = await this.monitorService.findById(request.params.id);
  return reply.send({ success: true, data: monitor });
}

// Use typed request/reply parameters
async create(
  request: FastifyRequest<{ Body: CreateMonitorDto }>,
  reply: FastifyReply
) {
  const monitor = await this.monitorService.create(request.body);
  return reply.code(201).send({ success: true, data: monitor });
}

// Log at entry points
async delete(request: FastifyRequest, reply: FastifyReply) {
  request.log.info({ monitorId: request.params.id }, 'Deleting monitor');
  await this.monitorService.delete(request.params.id);
  return reply.code(204).send();
}
```

#### ❌ DON'T:

```typescript
// Don't use try-catch in controllers
async getById(request: FastifyRequest, reply: FastifyReply) {
  try { // ❌ Unnecessary
    const monitor = await this.monitorService.findById(request.params.id);
    return reply.send(monitor);
  } catch (error) {
    reply.code(500).send({ error: error.message });
  }
}

// Don't put business logic in controllers
async create(request: FastifyRequest, reply: FastifyReply) {
  // ❌ This belongs in service layer
  const existingMonitor = await Monitor.findOne({
    where: { url: request.body.url }
  });
  if (existingMonitor) {
    throw new Error('Monitor already exists');
  }
  // ... more business logic
}
```

---

## 7. Service Patterns

Services contain **business logic** and orchestrate operations.

### Service Template

```typescript
/**
 * MonitorService - Business logic for monitor operations
 *
 * Services orchestrate:
 * - Multiple repository calls
 * - Business rule validation
 * - External service integration
 * - Event emissions
 */
import { Logger } from '@/lib/logger';
import { MonitorRepository, TeamRepository } from '@/repositories';
import { SSEService, InfluxService } from '@/services';
import { MonitorDto, CreateMonitorDto } from '@botfleet/shared';

export default class MonitorService {
    private monitorRepository: MonitorRepository;
    private teamRepository: TeamRepository;
    private sseService: SSEService;
    private influxService: InfluxService;
    private logger: Logger;

    constructor({ monitorRepository, teamRepository, sseService, influxService, logger }: ServiceDependencies) {
        this.monitorRepository = monitorRepository;
        this.teamRepository = teamRepository;
        this.sseService = sseService;
        this.influxService = influxService;
        this.logger = logger;
    }

    /**
     * Create a new monitor with business validation
     */
    async createMonitor(teamId: number, data: CreateMonitorDto): Promise<MonitorDto> {
        this.logger.info('Creating monitor', { teamId, data });

        // Business validation
        const team = await this.teamRepository.findById(teamId);
        if (!team) {
            throw new NotFoundError('Team not found');
        }

        // Check quotas
        const monitorCount = await this.monitorRepository.countByTeam(teamId);
        if (monitorCount >= team.maxMonitors) {
            throw new QuotaExceededError('Monitor limit reached');
        }

        // Create monitor
        const monitor = await this.monitorRepository.create({
            ...data,
            team_id: teamId,
        });

        // Emit real-time update
        await this.sseService.emit('monitor:created', {
            teamId,
            monitor: monitor,
        });

        // Initialize metrics
        await this.influxService.initializeMonitorMetrics(monitor.id);

        this.logger.info('Monitor created', { monitorId: monitor.id });
        return monitor;
    }

    /**
     * List monitors with real-time status
     */
    async listMonitors(teamId: number, page: number, limit: number): Promise<PaginatedResult<MonitorDto>> {
        // Get monitors from database
        const result = await this.monitorRepository.listByTeam(teamId, page, limit);

        // Enrich with real-time status
        const monitorIds = result.data.map((m) => m.id);
        const statusMap = await this.influxService.getBatchMonitorStatus(monitorIds);

        // Combine data
        const enrichedMonitors = result.data.map((monitor) => ({
            ...monitor,
            realtimeStatus: statusMap[monitor.id] || null,
        }));

        return {
            data: enrichedMonitors,
            meta: result.meta,
        };
    }
}
```

### Service Rules

#### ✅ DO:

- Orchestrate multiple operations
- Implement business rules
- Handle transactions
- Emit events for real-time updates
- Log important business operations

#### ❌ DON'T:

- Access HTTP request/response directly
- Handle HTTP status codes
- Parse query parameters
- Implement data access logic (use repositories)

---

## 8. Repository Patterns

Repositories handle **data access** - all database queries go here.

### Repository Template

```typescript
/**
 * MonitorRepository - Data access layer for monitors
 *
 * Repositories handle:
 * - Database queries
 * - Query optimization
 * - Data transformation
 * - NO business logic!
 */
import { Monitor } from '@/models';
import { Op, WhereOptions } from 'sequelize';
import { PaginatedResult, MonitorEntity } from '@botfleet/shared';

export default class MonitorRepository {
    /**
     * Find monitor by ID with relations
     */
    async findById(monitorId: number, teamId?: number): Promise<MonitorEntity | null> {
        const where: WhereOptions = { id: monitorId };
        if (teamId) {
            where.team_id = teamId;
        }

        return await Monitor.findOne({
            where,
            include: [
                {
                    association: 'team',
                    attributes: ['id', 'name'],
                },
            ],
        });
    }

    /**
     * List monitors with pagination
     */
    async listByTeam(
        teamId: number,
        page: number = 1,
        limit: number = 50,
        filters?: MonitorFilters
    ): Promise<PaginatedResult<MonitorEntity>> {
        const offset = (page - 1) * limit;

        // Build where clause
        const where: WhereOptions = { team_id: teamId };

        if (filters?.status) {
            where.status = filters.status;
        }

        if (filters?.search) {
            where[Op.or] = [
                { name: { [Op.like]: `%${filters.search}%` } },
                { url: { [Op.like]: `%${filters.search}%` } },
            ];
        }

        // Execute query with count
        const { rows, count } = await Monitor.findAndCountAll({
            where,
            limit,
            offset,
            order: [['created_at', 'DESC']],
            distinct: true, // Important for accurate count with joins
        });

        return {
            data: rows,
            meta: {
                page,
                limit,
                total: count,
                totalPages: Math.ceil(count / limit),
                hasMore: offset + rows.length < count,
            },
        };
    }

    /**
     * Batch operations for performance
     */
    async updateBatch(monitorIds: number[], updates: Partial<MonitorEntity>): Promise<number> {
        const [affectedRows] = await Monitor.update(updates, {
            where: { id: { [Op.in]: monitorIds } },
        });
        return affectedRows;
    }
}
```

### Repository Rules

#### ✅ DO:

- Use Sequelize models and operators
- Include related data when needed
- Handle pagination properly
- Use transactions for consistency
- Optimize queries (indexes, select specific fields)

#### ❌ DON'T:

- Implement business logic
- Throw business errors
- Call other services
- Handle authentication/authorization

---

## 9. Dependency Injection

Use **Awilix** for clean dependency injection.

### Container Setup

```typescript
// loaders/container.ts
import { diContainer } from '@fastify/awilix';
import { asClass, asFunction } from 'awilix';

export async function createContainer() {
    // Register repositories
    await diContainer.register({
        monitorRepository: asClass(MonitorRepository).singleton(),
        userRepository: asClass(UserRepository).singleton(),
    });

    // Register services
    await diContainer.register({
        monitorService: asClass(MonitorService).singleton(),
        authService: asClass(AuthService).singleton(),
        sseService: asClass(SSEService).singleton(),
    });

    // Register controllers
    await diContainer.register({
        monitorController: asClass(MonitorController).singleton(),
        authController: asClass(AuthController).singleton(),
    });
}
```

### Awilix Constructor Pattern

```typescript
// ✅ CORRECT: Destructured constructor parameters (works with Awilix)
export class AuthController {
    constructor({ authService, userRepository }: {
        authService: AuthService;
        userRepository: UserRepository;
    }) {
        this.authService = authService;
        this.userRepository = userRepository;
    }
}

// ❌ WRONG: TypeScript parameter properties are not supported by Awilix
export class BadController {
    constructor(private authService: AuthService) {}
}
```

### Using DI in Classes

```typescript
// ✅ GOOD: Constructor injection
export default class MonitorService {
    constructor({ monitorRepository, logger }: { monitorRepository: MonitorRepository; logger: Logger }) {
        this.monitorRepository = monitorRepository;
        this.logger = logger;
    }
}

// ❌ BAD: Service locator anti-pattern
export default class MonitorService {
    constructor(container: AwilixContainer) {
        this.monitorRepository = container.resolve('monitorRepository');
        this.logger = container.resolve('logger');
    }
}
```

---

## 10. Validation

Use **Zod** schemas from shared package for validation.

### Request Validation

```typescript
// routes/monitorsRoute.ts
import { createMonitorSchema } from '@botfleet/shared';

export default async function monitorsRoute(app: FastifyInstance) {
    app.post('/monitors', {
        schema: {
            body: createMonitorSchema,
        },
        handler: app.diContainer.resolve('monitorController').create,
    });
}
```

### Response Schema Note (Fastify JSON Schema)

```typescript
// Critical: allow dynamic objects in responses to avoid data stripping
schema: {
    response: {
        200: {
            type: 'object',
            properties: {
                success: { type: 'boolean' },
                data: { type: 'object', additionalProperties: true }, // IMPORTANT
                message: { type: 'string' },
            },
        },
    },
}
```

### Service-Level Validation

```typescript
// services/monitor.service.ts
import { z } from 'zod';
import { createMonitorSchema } from '@botfleet/shared';

async createMonitor(data: unknown): Promise<Monitor> {
  // Parse and validate
  const validatedData = createMonitorSchema.parse(data);

  // Additional business validation
  if (validatedData.interval < 60) {
    throw new ValidationError('Interval must be at least 60 seconds');
  }

  return await this.monitorRepository.create(validatedData);
}
```

---

## 11. SSE Implementation

Server-Sent Events for real-time updates.

### SSE Service

```typescript
// services/sse.service.ts
export default class SSEService {
    private connections: Map<string, Set<FastifyReply>>;

    constructor() {
        this.connections = new Map();
    }

    /**
     * Add SSE connection for a team
     */
    addConnection(teamId: string, reply: FastifyReply): void {
        if (!this.connections.has(teamId)) {
            this.connections.set(teamId, new Set());
        }
        this.connections.get(teamId)!.add(reply);

        // Setup heartbeat
        const interval = setInterval(() => {
            this.sendToClient(reply, { type: 'ping' });
        }, 30000);

        // Cleanup on disconnect
        reply.raw.on('close', () => {
            clearInterval(interval);
            this.connections.get(teamId)?.delete(reply);
        });
    }

    /**
     * Emit event to team members
     */
    emit(event: string, data: any): void {
        const teamId = data.teamId;
        const connections = this.connections.get(teamId);

        if (!connections) return;

        const message = {
            type: event,
            data,
            timestamp: new Date().toISOString(),
        };

        connections.forEach((reply) => {
            this.sendToClient(reply, message);
        });
    }

    private sendToClient(reply: FastifyReply, data: any): void {
        reply.raw.write(`data: ${JSON.stringify(data)}\n\n`);
    }
}
```

### SSE Route

```typescript
// routes/eventsRoute.ts
export default async function eventsRoute(app: FastifyInstance) {
    app.get('/events', async (request, reply) => {
        reply.raw.writeHead(200, {
            'Content-Type': 'text/event-stream',
            'Cache-Control': 'no-cache',
            Connection: 'keep-alive',
        });

        const teamId = request.user.teamId;
        app.diContainer.resolve('sseService').addConnection(teamId, reply);

        // Keep connection open
        request.raw.on('close', () => {
            reply.raw.end();
        });
    });
}
```

---

## 12. Database Standards

### Migrations (Sequelize CLI)

```typescript
// migrations/2025.11.30-create-monitor.ts
import { DataTypes, QueryInterface } from 'sequelize';

export async function up(queryInterface: QueryInterface) {
    await queryInterface.createTable('monitor', {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true,
            allowNull: false,
        },
        team_id: {
            type: DataTypes.INTEGER,
            allowNull: false,
            references: { model: 'team', key: 'id' },
            onDelete: 'CASCADE',
        },
        name: { type: DataTypes.STRING(255), allowNull: false },
        url: { type: DataTypes.STRING(2000), allowNull: false },
        status: { type: DataTypes.ENUM('active', 'paused', 'deleted'), defaultValue: 'active' },
        interval: { type: DataTypes.INTEGER, defaultValue: 300 },
        created_at: { type: DataTypes.DATE, allowNull: false, defaultValue: DataTypes.NOW },
        updated_at: { type: DataTypes.DATE, allowNull: false, defaultValue: DataTypes.NOW },
    });

    await queryInterface.addIndex('monitor', ['team_id']);
    await queryInterface.addIndex('monitor', ['team_id', 'status']);
}

export async function down(queryInterface: QueryInterface) {
    await queryInterface.dropTable('monitor');
}
```

### Model Definition

```typescript
// models/monitor.model.ts
import { Model, DataTypes } from 'sequelize';
import { sequelize } from '@/lib/database';

export class Monitor extends Model {
    declare id: number;
    declare team_id: number;
    declare name: string;
    declare url: string;
    declare status: 'active' | 'paused' | 'deleted';
    declare interval: number;
    declare created_at: Date;
    declare updated_at: Date;
}

Monitor.init(
    {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true,
        },
        team_id: {
            type: DataTypes.INTEGER,
            allowNull: false,
        },
        name: {
            type: DataTypes.STRING(255),
            allowNull: false,
        },
        url: {
            type: DataTypes.STRING(2000),
            allowNull: false,
        },
        status: {
            type: DataTypes.ENUM('active', 'paused', 'deleted'),
            defaultValue: 'active',
        },
        interval: {
            type: DataTypes.INTEGER,
            defaultValue: 300,
        },
    },
    {
        sequelize,
        modelName: 'Monitor',
        tableName: 'monitor',
        underscored: true,
        timestamps: true,
        createdAt: 'created_at',
        updatedAt: 'updated_at',
    }
);
```

### Database Rules

#### ✅ DO:

- Use migrations for all schema changes
- Add indexes for frequently queried columns
- Use foreign keys for referential integrity
- Use transactions for multi-table operations
- Use Sequelize timestamps for `created_at`/`updated_at` (PostgreSQL)
- **CRITICAL**: Always use parameterized queries with named replacements
- **CRITICAL**: Escape SQL wildcards in LIKE queries to prevent injection

#### ❌ DON'T:

- Modify database directly in production
- Use raw SQL unless absolutely necessary
- Store sensitive data unencrypted
- Use `SELECT *` in production queries
- **NEVER** concatenate user input directly into SQL queries
- **NEVER** use unescaped user input in LIKE patterns

### SQL Injection Prevention

**CRITICAL SECURITY REQUIREMENT**: All database queries MUST be safe from SQL injection.

**PREFERRED APPROACH**: Use Sequelize ORM - it automatically prevents SQL injection.

#### ✅ BEST - Use Sequelize ORM with Op.like (Automatic Protection):

```typescript
import { Op } from 'sequelize';
import { Monitor } from '@/models/monitor.model';

// ✅ SAFE - Sequelize automatically escapes wildcards and prevents SQL injection
// NO manual escaping needed!
const monitors = await Monitor.findAll({
    where: {
        organization_team_id: teamId,
        [Op.or]: [
            { name: { [Op.like]: `%${filters.search}%` } }, // Automatically safe!
            { url: { [Op.like]: `%${filters.search}%` } },
        ],
    },
    order: [['created_at', 'DESC']],
    limit: 50,
});
```

**Why This Is Better:**

- ✅ Sequelize uses parameterized queries automatically
- ✅ Wildcards are automatically escaped
- ✅ Type-safe and easier to maintain
- ✅ Less code = fewer bugs

#### ⚠️ ACCEPTABLE - Raw SQL with manual escaping (Only When Necessary):

Use raw SQL only for complex queries that Sequelize can't express (multi-table JOINs or DB-specific features). Prefer Sequelize on PostgreSQL.

```typescript
// For LIKE queries - ALWAYS escape wildcards to prevent SQL injection
if (filters.search) {
    // Escape backslash first, then wildcards (order matters!)
    const escapedSearch = filters.search
        .replace(/\\/g, '\\\\') // Escape backslashes
        .replace(/%/g, '\\%') // Escape % wildcard
        .replace(/_/g, '\\_'); // Escape _ wildcard

    whereConditions.push("(name LIKE :search OR url LIKE :search ESCAPE '\\')");
    replacements.search = `%${escapedSearch}%`;
}

// Execute with parameterized query
const results = await this.sequelize.query(query, {
    replacements, // Named parameters prevent SQL injection
    type: QueryTypes.SELECT,
});
```

#### ❌ WRONG - Vulnerable to SQL injection:

```typescript
// ❌ NEVER concatenate user input into SQL
const query = `SELECT * FROM monitors WHERE name LIKE '%${filters.search}%'`;

// ❌ WRONG - Parameterized but unescaped wildcards allow injection
replacements.search = `%${filters.search}%`; // Attacker can inject: "%'; DROP TABLE--"
whereConditions.push('name LIKE :search');

// ❌ WRONG - Missing ESCAPE clause
whereConditions.push('name LIKE :search');
replacements.search = `%${filters.search.replace(/%/g, '\\%')}%`;
```

#### Why Wildcard Escaping Is Critical

**Attack Vector**: Without escaping, attackers can manipulate LIKE patterns:

```typescript
// Malicious input: "%'; DROP TABLE monitors; --"
// Without escaping: LIKE '%'; DROP TABLE monitors; --%'
// Result: SQL injection - table deleted!

// With proper escaping: LIKE '%\%\'; DROP TABLE monitors; --\%'
// Result: Safe - searches for literal string
```

**Escape Order Matters**:

1. Escape `\` first (escape character itself)
2. Then escape `%` (matches any characters)
3. Finally escape `_` (matches single character)

**Defense in Depth**:

- Use parameterized queries (prevents direct injection)
- Escape SQL wildcards (prevents pattern-based injection)
- Add `ESCAPE '\\'` clause (explicitly defines escape character)

#### Helper Function Pattern

Create a reusable helper to ensure consistency:

```typescript
// lib/sql-utils.ts
/**
 * Escape SQL wildcards for safe LIKE queries
 * SECURITY: Prevents SQL injection via pattern manipulation
 *
 * @param input - User-provided search string
 * @returns Escaped string safe for LIKE patterns
 */
export function escapeSqlWildcards(input: string): string {
    return input
        .replace(/\\/g, '\\\\') // Escape backslashes first
        .replace(/%/g, '\\%') // Escape % wildcard
        .replace(/_/g, '\\_'); // Escape _ wildcard
}

// Usage in repositories
import { escapeSqlWildcards } from '@/lib/sql-utils';

if (filters.search) {
    const escapedSearch = escapeSqlWildcards(filters.search);
    whereConditions.push("(name LIKE :search ESCAPE '\\')");
    replacements.search = `%${escapedSearch}%`;
}
```

#### Testing SQL Injection Prevention

**Test Cases** (add to repository tests):

```typescript
describe('MonitorRepository - SQL Injection Prevention', () => {
    it('should escape % wildcard in search', async () => {
        // Attacker tries to match all records with %
        const result = await repository.listByTeam(1, { search: '%' });
        // Should search for literal '%' character, not match all
        expect(result.data.length).toBe(0);
    });

    it('should escape _ wildcard in search', async () => {
        const result = await repository.listByTeam(1, { search: 'test_injection' });
        // Should search for literal 'test_injection', not 'test*injection'
        expect(result.data).toMatchSnapshot();
    });

    it('should escape backslash in search', async () => {
        const result = await repository.listByTeam(1, { search: 'test\\escape' });
        // Should search for literal backslash
        expect(result.data).toMatchSnapshot();
    });

    it('should prevent SQL injection via comment injection', async () => {
        // Attacker tries SQL comment injection
        const result = await repository.listByTeam(1, { search: "%'; DROP TABLE monitors; --" });
        // Should be treated as literal search string
        expect(result.data).toBeDefined();
        // Verify table still exists
        const count = await repository.countByTeam(1);
        expect(count).toBeGreaterThanOrEqual(0);
    });
});
```

---

## 13. File Upload Security

**CRITICAL SECURITY REQUIREMENT**: All file upload endpoints MUST implement multiple layers of security to prevent attacks.

### Security Threats in File Uploads

1. **Path Traversal**: Malicious filenames like `../../etc/passwd` can write to unauthorized locations
2. **Memory Exhaustion**: Large files can consume server memory (DoS attack)
3. **File Type Abuse**: Executable files disguised as images
4. **Filename Injection**: Special characters in filenames can cause command injection

### Required Security Layers

#### ✅ CORRECT - Secure File Upload Pattern:

```typescript
import path from 'node:path';
import { sanitizeFilename, validatePathWithinDirectory } from '@/lib/security-validation';
import { BadRequestError } from '@botfleet/shared';

async uploadFile(request: FastifyRequest, reply: FastifyReply) {
  const data = await request.file();

  // LAYER 1: Validate file exists
  if (!data) {
    throw new BadRequestError('No file uploaded', 'NO_FILE');
  }

  // LAYER 2: Check file size BEFORE loading into memory
  const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB (matches multipart config)
  if (data.file.bytesRead > MAX_FILE_SIZE) {
    throw new BadRequestError(
      `File too large. Maximum size is ${MAX_FILE_SIZE / 1024 / 1024}MB`,
      'FILE_TOO_LARGE'
    );
  }

  // LAYER 3: Validate MIME type (whitelist approach)
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  if (!allowedTypes.includes(data.mimetype)) {
    throw new BadRequestError(
      `Invalid file type. Allowed: ${allowedTypes.join(', ')}`,
      'INVALID_FILE_TYPE'
    );
  }

  // LAYER 4: Sanitize filename (removes path traversal attempts)
  const safeFilename = sanitizeFilename(data.filename);

  // LAYER 5: Construct path safely
  const tmpFilePath = path.join('/tmp', `${Date.now()}-${safeFilename}`);

  // LAYER 6: Verify path stays within allowed directory
  validatePathWithinDirectory(tmpFilePath, '/tmp');

  // Now safe to process file
  const buffer = await data.toBuffer();
  await fs.writeFile(tmpFilePath, buffer);

  // Upload to S3/storage...
  const result = await this.uploadService.upload(tmpFilePath, safeFilename);

  // LAYER 7: Clean up temporary file
  await fs.unlink(tmpFilePath).catch(() => {/* ignore */});

  return { url: result.url };
}
```

#### ❌ WRONG - Vulnerable File Upload:

```typescript
// ❌ WRONG - Missing size check (memory exhaustion attack)
const buffer = await data.toBuffer();

// ❌ WRONG - Unsanitized filename (path traversal attack)
const tmpFilePath = `/tmp/${data.filename}`;
// Attacker sends filename: "../../etc/cron.d/evil"
// Result: Writes to /etc/cron.d/evil instead of /tmp

// ❌ WRONG - No MIME type validation (executable upload)
// Attacker uploads "image.jpg" that is actually a .exe file

// ❌ WRONG - No path verification
const tmpFilePath = `/tmp/../../../etc/passwd`;
// Path traversal succeeds
```

### Security Validation Utilities

Use the centralized security validation library:

```typescript
import { sanitizeFilename, validatePathWithinDirectory } from '@/lib/security-validation';

// Sanitize filename - removes directory paths and unsafe characters
const safe = sanitizeFilename('../../etc/passwd'); // Returns: 'passwd'
const safe = sanitizeFilename('file<>:"/\\|?*.txt'); // Returns: 'file_________.txt'

// Validate path stays within allowed directory
const filePath = path.join('/tmp', safeFilename);
validatePathWithinDirectory(filePath, '/tmp'); // Throws if path escapes /tmp
```

### Fastify Multipart Configuration

Enforce size limits at the plugin level (configured in `plugin.ts`):

```typescript
{
  plugin: multipart,
  name: 'Multipart',
  options: {
    limits: {
      fieldNameSize: 100,      // Max field name size
      fieldSize: 100,          // Max field value size
      fields: 10,              // Max number of non-file fields
      fileSize: 1024 * 1024 * 5, // 5MB max file size
      files: 1,                // Max number of file fields
      headerPairs: 2000,       // Max number of header key-value pairs
    },
  },
}
```

### File Upload Testing

**Test Cases** (add to controller tests):

```typescript
describe('UploadController - Security', () => {
    it('should reject files larger than 5MB', async () => {
        const largeFile = Buffer.alloc(6 * 1024 * 1024); // 6MB
        await expect(uploadFile(largeFile)).rejects.toThrow('FILE_TOO_LARGE');
    });

    it('should sanitize path traversal in filename', async () => {
        const result = await uploadFile('../../etc/passwd');
        expect(result.url).not.toContain('../');
        expect(result.url).toContain('passwd'); // Only basename kept
    });

    it('should reject non-image files', async () => {
        const exe = createFile('malware.exe', 'application/x-msdownload');
        await expect(uploadFile(exe)).rejects.toThrow('INVALID_FILE_TYPE');
    });

    it('should clean up temp files on upload failure', async () => {
        // Mock S3 upload to fail
        jest.spyOn(uploadService, 'upload').mockRejectedValue(new Error('S3 error'));

        await expect(uploadFile(validFile)).rejects.toThrow();

        // Verify temp file was deleted
        const files = await fs.readdir('/tmp');
        expect(files).not.toContain(expect.stringContaining('test-file'));
    });
});
```

### File Storage Best Practices

1. **Never store uploaded files in application directory** - Use `/tmp` or dedicated storage service
2. **Use unique filenames** - Prevent overwrites: `${timestamp}-${uuid}-${sanitized}`
3. **Scan files for malware** - Integrate ClamAV or cloud scanning service
4. **Set Content-Disposition header** - Force download instead of inline rendering: `attachment; filename="safe.jpg"`
5. **Use signed URLs** - Time-limited access to uploaded files (S3 pre-signed URLs)
6. **Implement upload quotas** - Prevent disk space exhaustion attacks

---

## 14. Error Handling

### Error Class vs API Response Interface Distinction

**IMPORTANT**: Distinguish between error classes and API response interfaces:

- **`AppError` (Class)**: Base class for throwing exceptions in application code
- **`ApiErrorResponse` (Interface)**: JSON response format sent to clients

```typescript
// ✅ CORRECT - Use AppError for exceptions
throw new BadRequestError('Invalid input', 'VALIDATION_ERROR');

// ✅ CORRECT - Use ApiErrorResponse for response typing
interface ApiErrorResponse {
    success: false;
    error: {
        code: string;
        message: string;
        details?: any;
    };
}
```

### Custom Error Classes

```typescript
// Use the standardized AppError hierarchy from @botfleet/shared
import { AppError, BadRequestError, UnauthorizedError, NotFoundError, InternalError } from '@botfleet/shared';

// Example usage in controllers
export class NotFoundError extends AppError {
    constructor(message = 'Resource not found') {
        super(message, 404, 'NOT_FOUND');
    }
}

export class ValidationError extends AppError {
    constructor(
        message: string,
        public errors?: any
    ) {
        super(message, 400, 'VALIDATION_ERROR');
    }
}

export class UnauthorizedError extends AppError {
    constructor(message = 'Unauthorized') {
        super(message, 401, 'UNAUTHORIZED');
    }
}
```

### Standard Error Classes to Use

Always use the pre-defined error classes from `@botfleet/shared`:

```typescript
// HTTP Error Classes
import {
    BadRequestError, // 400 - Validation/input errors
    UnauthorizedError, // 401 - Authentication required
    ForbiddenError, // 403 - Access denied
    NotFoundError, // 404 - Resource not found
    ConflictError, // 409 - Resource already exists
    UnprocessableEntityError, // 422 - Business validation failed
    InternalError, // 500 - Unexpected server errors
} from '@botfleet/shared';

// Business Logic Error Classes
import {
    UserNotFoundError,
    TeamNotFoundError,
    MonitorNotFoundError,
    EmailAlreadyExistsError,
    InvalidCredentialsError,
    MonitorLimitExceededError,
} from '@botfleet/shared';
```

**Usage Examples:**

```typescript
// ✅ CORRECT - Use specific error classes
throw new BadRequestError('Invalid team ID', 'INVALID_TEAM_ID');
throw new UnauthorizedError('Authentication required', 'AUTH_REQUIRED');
throw new UserNotFoundError(userId);

// ❌ WRONG - Don't create custom Error instances
throw new Error('User not found'); // Generic, no status code
throw new AppError('Invalid input'); // Abstract class
```

### Global Error Handler

```typescript
// middleware/errorHandler.ts
export function errorHandler(error: Error, request: FastifyRequest, reply: FastifyReply): void {
    request.log.error({
        error: error.message,
        stack: error.stack,
        url: request.url,
        method: request.method,
    });

    if (error instanceof AppError) {
        reply.status(error.statusCode).send({
            success: false,
            error: {
                code: error.code,
                message: error.message,
                ...(error instanceof ValidationError && { errors: error.errors }),
            },
        });
        return;
    }

    // Unhandled errors
    reply.status(500).send({
        success: false,
        error: {
            code: 'INTERNAL_ERROR',
            message: 'An unexpected error occurred',
        },
    });
}
```

---

## 14. Testing

### Unit Tests

```typescript
// tests/services/monitor.service.test.ts
import { MonitorService } from '@/services';
import { createMockRepository } from '../mocks';

describe('MonitorService', () => {
    let service: MonitorService;
    let mockRepository: jest.Mocked<MonitorRepository>;

    beforeEach(() => {
        mockRepository = createMockRepository();
        service = new MonitorService({
            monitorRepository: mockRepository,
            logger: createMockLogger(),
        });
    });

    describe('createMonitor', () => {
        it('should create monitor with valid data', async () => {
            const data = { name: 'Test', url: 'https://example.com' };
            const expectedMonitor = { id: 1, ...data };

            mockRepository.create.mockResolvedValue(expectedMonitor);

            const result = await service.createMonitor(1, data);

            expect(result).toEqual(expectedMonitor);
            expect(mockRepository.create).toHaveBeenCalledWith({
                ...data,
                team_id: 1,
            });
        });

        it('should throw QuotaExceededError when limit reached', async () => {
            mockRepository.countByTeam.mockResolvedValue(100);

            await expect(service.createMonitor(1, { name: 'Test', url: 'https://example.com' })).rejects.toThrow(
                QuotaExceededError
            );
        });
    });
});
```

### Integration Tests

```typescript
// tests/integration/monitors.test.ts
import { build } from '../helper';

describe('Monitor Endpoints', () => {
    let app: FastifyInstance;

    beforeAll(async () => {
        app = await build();
    });

    afterAll(async () => {
        await app.close();
    });

    describe('POST /api/monitors', () => {
        it('should create a monitor', async () => {
            const response = await app.inject({
                method: 'POST',
                url: '/api/teams/1/monitors',
                headers: {
                    authorization: 'Bearer valid-token',
                },
                payload: {
                    name: 'API Monitor',
                    url: 'https://api.example.com',
                    interval: 300,
                },
            });

            expect(response.statusCode).toBe(201);
            expect(response.json()).toMatchObject({
                success: true,
                data: {
                    name: 'API Monitor',
                    url: 'https://api.example.com',
                },
            });
        });
    });
});
```

---

## 15. Logging Standards

### Logger Sources by Layer

- **Services**: Use injected `this.logger` with structured objects
- **Controllers**: Use `request.log` (includes correlation IDs automatically)
- **Middleware**: Use `fastify.log` for infrastructure concerns
- **Format**: Always `{ structured_data }, 'readable_message'`
- **Security**: Automatic sensitive data redaction built into logger
- **Performance**: Include `operation`, `phase`, and timing data for monitoring

### Logger Type Import

```typescript
// ✅ CORRECT - Type-only import
import type { Logger } from '@/lib/logger';

// ❌ WRONG - Runtime import for typing
import { Logger } from '@/lib/logger';
```

### Structured Logging Format (Pino Standard)

```typescript
// ✅ CORRECT - Object first, message second
logger.info(
    {
        userId: 123,
        operation: 'create_monitor',
        phase: 'start',
    },
    'Creating monitor for user'
);

// ❌ WRONG - Message first (not pino-compatible)
logger.info('Creating monitor', { userId: 123 });

// ❌ WRONG - Only message (loses structured data)
logger.info('Creating monitor for user');
```

### Log Levels Usage

```typescript
// DEBUG - Development debugging, query details
this.logger.debug({ query, params }, 'Database query executed');

// INFO - Business operations, user actions
this.logger.info({ userId, monitorId, operation: 'create' }, 'Monitor created');

// WARN - Recoverable issues, validation failures
this.logger.warn({ userId, error: 'invalid_url' }, 'Invalid monitor URL provided');

// ERROR - System errors, exceptions
this.logger.error({ error: err.message, stack: err.stack }, 'Database connection failed');
```

### Context Fields Standards

```typescript
// User operations
{ userId: number, correlationId?: string, operation: string, phase?: string }

// Team operations
{ userId: number, teamId: number, operation: string, phase?: string }

// Error logging
{ error: string, stack?: string, correlationId?: string }

// Performance logging
{ operation: string, phase: string, duration?: number, recordCount?: number }
```

### Layer-Specific Patterns

#### Service Layer

```typescript
export default class MonitorService {
    private logger: Logger;

    async createMonitor(teamId: number, data: CreateMonitorDto) {
        this.logger.info(
            {
                teamId,
                operation: 'create_monitor',
                phase: 'start',
            },
            'Starting monitor creation'
        );

        // ... business logic ...

        this.logger.info(
            {
                teamId,
                monitorId: monitor.id,
                operation: 'create_monitor',
                phase: 'complete',
            },
            'Monitor creation completed'
        );
    }
}
```

#### Controller Layer

```typescript
async create(request: FastifyRequest, reply: FastifyReply) {
  request.log.info({
    userId: user.id,
    teamId,
    operation: 'create_monitor_request'
  }, 'Monitor creation requested');

  const result = await this.monitorService.createMonitor(teamId, request.body);
  return reply.send({ success: true, data: result });
}
```

#### Middleware Layer

```typescript
export async function registerAuthenticationMiddleware(fastify: FastifyInstance) {
    fastify.decorate('authenticate', async (request: any, reply: any) => {
        fastify.log.info(
            {
                userId: session.userId,
                sessionId: decoded.sessionId,
                operation: 'authenticate',
            },
            'User authenticated successfully'
        );
    });

    fastify.log.info({ middleware: 'authentication' }, 'Authentication middleware registered');
}
```

### Anti-Patterns to Avoid

```typescript
// ❌ DON'T use console.log (except pre-logger setup)
console.log('User logged in:', userId);

// ❌ DON'T mix logger sources within same layer
this.logger.info('Starting operation');
fastify.log.info('Operation in progress');
request.log.info('Operation completed');

// ❌ DON'T log sensitive data
logger.info({ password, token, creditCard }, 'Processing user data');

// ❌ DON'T use inconsistent message formats
logger.info('ServiceName.methodName');
logger.info('ServiceName.methodName - Success');
logger.info('ServiceName.methodName - Error occurred');
```

---

## 16. Architecture Improvements (Based on Code Review)

### Base Classes Pattern

To reduce code duplication and improve consistency, implement base classes for each layer:

```typescript
// controllers/base.controller.ts
export abstract class BaseController {
    protected getUserId(request: FastifyRequest): number {
        const userId = (request as any).user?.id;
        if (!userId) throw new Error('Authentication required');
        return userId;
    }

    protected sendSuccess<T>(reply: FastifyReply, data: T, message?: string) {
        return reply.send({
            success: true,
            data,
            ...(message && { message }),
        });
    }
}

// services/base.service.ts
export abstract class BaseService {
    protected logger: Logger;

    constructor(dependencies: { logger: Logger }) {
        this.logger = dependencies.logger;
    }

    protected logOperation(operation: string, data: Record<string, any>) {
        this.logger.info({ operation, ...data }, `${this.constructor.name}.${operation}`);
    }
}

// repositories/base.repository.ts
export abstract class BaseRepository<T extends Model> {
    protected logger: Logger;
    protected model: ModelStatic<T>;

    async findById(id: number): Promise<T | null> {
        this.logger.debug({ id }, `${this.constructor.name}.findById`);
        return await this.model.findByPk(id);
    }

    async create(data: any): Promise<T> {
        this.logger.info(data, `${this.constructor.name}.create`);
        return await this.model.create(data);
    }
}
```

### Middleware Extraction

Split app.ts middleware into focused files:

```typescript
// middleware/auth/authentication.middleware.ts
export async function registerAuthenticationMiddleware(fastify: FastifyInstance) {
    fastify.decorate('authenticate', async (request: any, reply: any) => {
        // Move auth logic from app.ts here
    });
}

// middleware/auth/team-authorization.middleware.ts
export async function registerTeamAuthorizationMiddleware(fastify: FastifyInstance) {
    fastify.decorate('checkTeamAccess', async (request: any, reply: any) => {
        // Move team auth logic from app.ts here
    });
}
```

### Health Monitoring

Add connection monitoring for better observability:

```typescript
// lib/database.ts
export const getDatabaseHealth = async () => {
    const pool = sequelize.connectionManager.pool;
    return {
        total: pool._allObjects.length,
        used: pool._allObjects.filter((obj) => obj._owner).length,
        available: pool._availableObjects.length,
        state: await sequelize
            .authenticate()
            .then(() => 'healthy')
            .catch(() => 'unhealthy'),
    };
};

// lib/redis.ts - Add circuit breaker
class CircuitBreaker {
    private failures = 0;
    private state: 'closed' | 'open' | 'half-open' = 'closed';

    async execute<T>(operation: () => Promise<T>): Promise<T> {
        if (this.state === 'open') {
            throw new Error('Circuit breaker is open');
        }
        // Implementation details
    }
}
```

### Metrics Integration

```typescript
// lib/metrics.ts
import { Counter, Histogram, register } from 'prom-client';

export class MetricsCollector {
    private readonly requestDuration = new Histogram({
        name: 'http_request_duration_seconds',
        help: 'Duration of HTTP requests in seconds',
        labelNames: ['method', 'route', 'status_code'],
    });

    recordDuration(method: string, route: string, statusCode: number, duration: number) {
        this.requestDuration.labels(method, route, statusCode.toString()).observe(duration);
    }
}

// middleware/metrics.middleware.ts
export function metricsMiddleware(metrics: MetricsCollector) {
    return async (request: FastifyRequest, reply: FastifyReply) => {
        const startTime = Date.now();

        reply.addHook('onSend', async () => {
            const duration = (Date.now() - startTime) / 1000;
            metrics.recordDuration(request.method, request.routerPath || request.url, reply.statusCode, duration);
        });
    };
}
```

---

## 📚 Additional Resources

- [Fastify Documentation](https://www.fastify.io/)
- [Sequelize Documentation](https://sequelize.org/)
- [Awilix Documentation](https://github.com/jeffijoe/awilix)
- [Zod Documentation](https://zod.dev/)
- [Pino Logging](https://getpino.io/)
- [Prometheus Metrics](https://prometheus.io/)

---

**Remember**: The API package owns the database. All migrations and schema changes happen here!
