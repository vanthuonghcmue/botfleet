# 🔗 Shared Package Coding Standards

> **Package**: `@botfleet/shared`
> **Purpose**: Lightweight shared code for all packages
> **Last Updated**: November 2025

## 📋 Table of Contents

1. [Package Structure](#package-structure)
2. [Type Definitions](#type-definitions)
3. [DTOs (Data Transfer Objects)](#dtos-data-transfer-objects)
4. [Validation Schemas](#validation-schemas)
5. [Constants](#constants)
6. [Utility Functions](#utility-functions)
7. [Error Classes](#error-classes)
8. [Bundle Size Management](#bundle-size-management)
9. [Export Organization](#export-organization)
10. [Testing](#testing)

---

## 1. Package Structure

```
packages/shared/
├── src/
│   ├── types/           # TypeScript interfaces
│   │   ├── common.types.ts
│   │   ├── monitor.types.ts
│   │   ├── user.types.ts
│   │   └── index.ts
│   ├── dto/             # Request/Response shapes
│   │   ├── monitor.dto.ts
│   │   ├── auth.dto.ts
│   │   └── index.ts
│   ├── validation/      # Zod schemas
│   │   ├── monitor.schemas.ts
│   │   ├── auth.schemas.ts
│   │   └── index.ts
│   ├── constants/       # Shared constants
│   │   ├── api.constants.ts
│   │   ├── status.constants.ts
│   │   └── index.ts
│   ├── utils/          # Lightweight utilities
│   │   ├── date.utils.ts
│   │   ├── transform.utils.ts
│   │   └── index.ts
│   ├── errors/         # Error classes
│   │   ├── base.errors.ts
│   │   ├── business.errors.ts
│   │   └── index.ts
│   └── index.ts       # Main barrel export
├── tests/
└── package.json
```

### Critical Rule: Keep It Lightweight!
```json
// package.json - Minimal dependencies
{
  "name": "@botfleet/shared",
  "dependencies": {
    "zod": "^3.22.0"  // ONLY Zod for validation
  },
  "devDependencies": {
    // Dev dependencies are fine
  }
}
```

**❌ NEVER ADD:**
- Database drivers (mysql2, pg, mongodb)
- HTTP frameworks (express, fastify)
- Heavy utilities (lodash, moment)
- Node.js specific modules (fs, path, crypto)

---

## 2. Type Definitions

### Type Organization
```typescript
// types/common.types.ts
/**
 * Common types used across multiple entities
 */

// ID types
export type ID = number;
export type UUID = string;

// Status types
export type EntityStatus = 'active' | 'inactive' | 'deleted';

// Timestamps
export interface Timestamps {
  created_at: string;
  updated_at: string;
}

// Pagination
export interface PaginationMeta {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
  hasMore: boolean;
}

export interface PaginatedResult<T> {
  data: T[];
  meta: PaginationMeta;
}
```

### Entity Types
```typescript
// types/monitor.types.ts
import { ID, Timestamps, EntityStatus } from './common.types';

/**
 * Monitor entity types
 * Represents database structure
 */
export interface MonitorEntity extends Timestamps {
  id: ID;
  team_id: ID;
  name: string;
  url: string;
  type: MonitorType;
  status: MonitorStatus;
  interval: number;
  timeout: number;
  retry_interval: number;
  max_retries: number;
  active: boolean;
  worker_locations: string[];
}

export type MonitorType =
  | 'http'
  | 'https'
  | 'tcp'
  | 'udp'
  | 'dns'
  | 'ping'
  | 'keyword';

export type MonitorStatus = 'up' | 'down' | 'paused' | 'pending';

/**
 * Monitor configuration for creation
 */
export interface CreateMonitorInput {
  name: string;
  url: string;
  type: MonitorType;
  interval?: number;
  timeout?: number;
  worker_locations?: string[];
}
```

### Type Rules

#### ✅ DO:
```typescript
// Use specific types
export type MonitorStatus = 'up' | 'down' | 'paused';

// Use interfaces for objects
export interface Monitor {
  id: number;
  name: string;
}

// Export commonly used types
export type Nullable<T> = T | null;
export type Optional<T> = T | undefined;

// Document complex types
/**
 * Represents a monitor check result from any location
 * @property monitorId - The monitor that was checked
 * @property location - Worker location that performed the check
 */
export interface CheckResult {
  monitorId: ID;
  location: string;
}
```

#### ❌ DON'T:
```typescript
// Don't use 'any'
export interface BadType {
  data: any; // ❌
}

// Don't create overly generic types
export type Thing = any; // ❌

// Don't include implementation details
export interface Monitor {
  _internal: SomeInternalType; // ❌
}
```

---

## 3. DTOs (Data Transfer Objects)

DTOs define the shape of data sent over the network.

### DTO Patterns
```typescript
// dto/monitor.dto.ts
import { MonitorEntity, MonitorStatus, ID } from '../types';

/**
 * Monitor data sent to frontend
 * Transformed from entity, excludes sensitive fields
 */
export interface MonitorDto {
  id: ID;
  teamId: ID;
  name: string;
  url: string;
  type: string;
  status: MonitorStatus;
  interval: number;
  active: boolean;
  lastCheckAt?: string;
  createdAt: string;
  updatedAt: string;
  // Real-time data added by API
  realtimeStatus?: MonitorRealtimeStatusDto;
}

/**
 * Real-time status from InfluxDB
 */
export interface MonitorRealtimeStatusDto {
  monitorId: ID;
  overallStatus: MonitorStatus;
  locations: LocationStatusDto[];
  lastUpdated: string;
}

export interface LocationStatusDto {
  location: string;
  status: MonitorStatus;
  responseTime: number;
  lastCheck: string;
}

/**
 * Transform entity to DTO
 * This function can be used by both API and Frontend
 */
export function transformMonitorToDto(
  entity: MonitorEntity
): MonitorDto {
  return {
    id: entity.id,
    teamId: entity.team_id,
    name: entity.name,
    url: entity.url,
    type: entity.type,
    status: entity.status,
    interval: entity.interval,
    active: entity.active,
    createdAt: entity.created_at,
    updatedAt: entity.updated_at,
  };
}
```

### Request/Response DTOs
```typescript
// dto/auth.dto.ts
/**
 * Login request from frontend
 */
export interface LoginRequestDto {
  email: string;
  password: string;
  rememberMe?: boolean;
}

/**
 * Login response from API
 */
export interface LoginResponseDto {
  success: boolean;
  user: UserDto;
  token: string;
  expiresIn: number;
}

/**
 * Standard API response wrapper
 */
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: ApiError;
  meta?: Record<string, any>;
}

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}
```

---

## 4. Validation Schemas

Use **Zod** for runtime validation that works in both frontend and backend.

### Schema Patterns
```typescript
// validation/monitor.schemas.ts
import { z } from 'zod';
import { MonitorType } from '../types';

/**
 * Base monitor validation
 */
const monitorBaseSchema = z.object({
  name: z
    .string()
    .min(1, 'Name is required')
    .max(255, 'Name too long'),
  url: z
    .string()
    .url('Invalid URL')
    .max(2000, 'URL too long'),
  type: z.enum(['http', 'https', 'tcp', 'dns', 'ping']),
});

/**
 * Create monitor validation
 */
export const createMonitorSchema = monitorBaseSchema.extend({
  interval: z
    .number()
    .min(60, 'Minimum interval is 60 seconds')
    .max(86400, 'Maximum interval is 24 hours')
    .default(300),
  timeout: z
    .number()
    .min(1000, 'Minimum timeout is 1 second')
    .max(30000, 'Maximum timeout is 30 seconds')
    .default(5000),
  worker_locations: z
    .array(z.string())
    .default(['default']),
});

/**
 * Update monitor validation (all fields optional)
 */
export const updateMonitorSchema = createMonitorSchema.partial();

/**
 * Monitor filters for list endpoint
 */
export const monitorFiltersSchema = z.object({
  status: z.enum(['active', 'paused', 'deleted']).optional(),
  type: z.string().optional(),
  search: z.string().optional(),
  page: z.number().positive().default(1),
  limit: z.number().positive().max(100).default(50),
});

// Export types from schemas
export type CreateMonitorInput = z.infer<typeof createMonitorSchema>;
export type UpdateMonitorInput = z.infer<typeof updateMonitorSchema>;
export type MonitorFilters = z.infer<typeof monitorFiltersSchema>;
```

### Validation Helpers
```typescript
// validation/common.schemas.ts
import { z } from 'zod';

/**
 * Common validation schemas
 */
export const idSchema = z.number().positive('Invalid ID');

export const emailSchema = z
  .string()
  .email('Invalid email format')
  .toLowerCase();

export const passwordSchema = z
  .string()
  .min(8, 'Password must be at least 8 characters')
  .regex(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    'Password must contain uppercase, lowercase, and number'
  );

export const paginationSchema = z.object({
  page: z.number().positive().default(1),
  limit: z.number().positive().max(100).default(50),
});

/**
 * Helper to validate and transform data
 */
export function validate<T>(
  schema: z.ZodSchema<T>,
  data: unknown
): T {
  return schema.parse(data);
}

export function validateSafe<T>(
  schema: z.ZodSchema<T>,
  data: unknown
): { success: true; data: T } | { success: false; error: z.ZodError } {
  const result = schema.safeParse(data);
  if (result.success) {
    return { success: true, data: result.data };
  }
  return { success: false, error: result.error };
}
```

---

## 5. Constants

Define shared constants used across packages.

### API Constants
```typescript
// constants/api.constants.ts
/**
 * API version and endpoints
 */
export const API_VERSION = 'v2';
export const API_BASE = `/api/${API_VERSION}`;

export const API_ENDPOINTS = {
  // Auth
  LOGIN: `${API_BASE}/auth/login`,
  LOGOUT: `${API_BASE}/auth/logout`,
  REFRESH: `${API_BASE}/auth/refresh`,
  ME: `${API_BASE}/users/me`,

  // Monitors
  MONITORS: `${API_BASE}/teams/:teamId/monitors`,
  MONITOR_BY_ID: `${API_BASE}/teams/:teamId/monitors/:id`,
  MONITOR_STATS: `${API_BASE}/teams/:teamId/monitors/stats`,

  // Events
  SSE_EVENTS: `${API_BASE}/teams/:teamId/events`,
} as const;

/**
 * HTTP methods
 */
export const HTTP_METHODS = {
  GET: 'GET',
  POST: 'POST',
  PUT: 'PUT',
  PATCH: 'PATCH',
  DELETE: 'DELETE',
} as const;

/**
 * Default values
 */
export const DEFAULT_PAGINATION = {
  PAGE: 1,
  LIMIT: 50,
  MAX_LIMIT: 100,
} as const;
```

### Status Constants
```typescript
// constants/status.constants.ts
/**
 * Monitor statuses
 */
export const MONITOR_STATUS = {
  UP: 'up',
  DOWN: 'down',
  PAUSED: 'paused',
  PENDING: 'pending',
} as const;

/**
 * HTTP status codes
 */
export const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  NO_CONTENT: 204,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,
  INTERNAL_SERVER_ERROR: 500,
  SERVICE_UNAVAILABLE: 503,
} as const;

/**
 * Error codes
 */
export const ERROR_CODES = {
  // Auth errors
  INVALID_CREDENTIALS: 'INVALID_CREDENTIALS',
  TOKEN_EXPIRED: 'TOKEN_EXPIRED',
  UNAUTHORIZED: 'UNAUTHORIZED',

  // Validation errors
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  INVALID_INPUT: 'INVALID_INPUT',

  // Business errors
  QUOTA_EXCEEDED: 'QUOTA_EXCEEDED',
  RESOURCE_NOT_FOUND: 'RESOURCE_NOT_FOUND',
  DUPLICATE_RESOURCE: 'DUPLICATE_RESOURCE',
} as const;
```

---

## 6. Utility Functions

Only include **lightweight, pure functions** that work in all environments.

### Date Utilities
```typescript
// utils/date.utils.ts
/**
 * Format ISO date to relative time
 * Works in both Node.js and browser
 */
export function timeAgo(date: string | Date): string {
  const seconds = Math.floor(
    (new Date().getTime() - new Date(date).getTime()) / 1000
  );

  const intervals = [
    { label: 'year', seconds: 31536000 },
    { label: 'month', seconds: 2592000 },
    { label: 'day', seconds: 86400 },
    { label: 'hour', seconds: 3600 },
    { label: 'minute', seconds: 60 },
  ];

  for (const interval of intervals) {
    const count = Math.floor(seconds / interval.seconds);
    if (count >= 1) {
      return `${count} ${interval.label}${count > 1 ? 's' : ''} ago`;
    }
  }

  return 'just now';
}

/**
 * Format duration in milliseconds
 */
export function formatDuration(ms: number): string {
  if (ms < 1000) return `${ms}ms`;
  if (ms < 60000) return `${(ms / 1000).toFixed(1)}s`;
  if (ms < 3600000) return `${Math.floor(ms / 60000)}m`;
  return `${(ms / 3600000).toFixed(1)}h`;
}

/**
 * Parse ISO date safely
 */
export function parseDate(
  date: string | Date | null | undefined
): Date | null {
  if (!date) return null;
  const parsed = new Date(date);
  return isNaN(parsed.getTime()) ? null : parsed;
}
```

### Transform Utilities
```typescript
// utils/transform.utils.ts
/**
 * Convert snake_case to camelCase
 */
export function snakeToCamel(str: string): string {
  return str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
}

/**
 * Convert object keys from snake_case to camelCase
 */
export function transformKeys<T = any>(obj: any): T {
  if (Array.isArray(obj)) {
    return obj.map(transformKeys) as T;
  }

  if (obj !== null && typeof obj === 'object') {
    return Object.keys(obj).reduce((result, key) => {
      const camelKey = snakeToCamel(key);
      result[camelKey] = transformKeys(obj[key]);
      return result;
    }, {} as any) as T;
  }

  return obj;
}

/**
 * Pick specific keys from object
 */
export function pick<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Pick<T, K> {
  const result = {} as Pick<T, K>;
  keys.forEach((key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
  });
  return result;
}

/**
 * Omit specific keys from object
 */
export function omit<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Omit<T, K> {
  const result = { ...obj };
  keys.forEach((key) => {
    delete result[key];
  });
  return result as Omit<T, K>;
}
```

### Validation Utilities
```typescript
// utils/validation.utils.ts
/**
 * Check if value is a valid email
 * Simple regex for client-side validation
 */
export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Check if URL is valid
 */
export function isValidUrl(url: string): boolean {
  try {
    new URL(url);
    return true;
  } catch {
    return false;
  }
}

/**
 * Sanitize string for display
 */
export function sanitize(str: string): string {
  return str
    .trim()
    .replace(/[<>]/g, '') // Remove potential HTML
    .slice(0, 1000); // Limit length
}
```

---

## 7. Error Classes

Define error classes that can be used across packages.

### Base Errors
```typescript
// errors/base.errors.ts
/**
 * Base error class for all custom errors
 */
export abstract class BaseError extends Error {
  abstract readonly statusCode: number;
  abstract readonly code: string;

  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode,
    };
  }
}
```

### Business Errors
```typescript
// errors/business.errors.ts
import { BaseError } from './base.errors';

export class ValidationError extends BaseError {
  readonly statusCode = 400;
  readonly code = 'VALIDATION_ERROR';

  constructor(
    message: string,
    public errors?: Record<string, string[]>
  ) {
    super(message);
  }
}

export class NotFoundError extends BaseError {
  readonly statusCode = 404;
  readonly code = 'NOT_FOUND';
}

export class UnauthorizedError extends BaseError {
  readonly statusCode = 401;
  readonly code = 'UNAUTHORIZED';
}

export class ForbiddenError extends BaseError {
  readonly statusCode = 403;
  readonly code = 'FORBIDDEN';
}

export class ConflictError extends BaseError {
  readonly statusCode = 409;
  readonly code = 'CONFLICT';
}

export class QuotaExceededError extends BaseError {
  readonly statusCode = 429;
  readonly code = 'QUOTA_EXCEEDED';

  constructor(
    message: string,
    public limit?: number,
    public current?: number
  ) {
    super(message);
  }
}
```

---

## 8. Bundle Size Management

Keep the package lightweight for frontend usage.

### Size Monitoring
```typescript
// package.json
{
  "scripts": {
    "size": "size-limit",
    "analyze": "size-limit --why"
  },
  "size-limit": [
    {
      "path": "dist/index.js",
      "limit": "50 KB"
    }
  ]
}
```

### Tree-Shaking Support
```typescript
// index.ts - Use specific exports
// ✅ GOOD: Allows tree-shaking
export { MonitorDto, UserDto } from './dto';
export { createMonitorSchema } from './validation';
export { timeAgo, formatDuration } from './utils';

// ❌ BAD: Exports everything
export * from './dto';
export * from './validation';
export * from './utils';
```

### Import Optimization
```typescript
// ✅ GOOD: Import only what you need
import { MonitorDto } from '@botfleet/shared/dto';
import { timeAgo } from '@botfleet/shared/utils';

// ❌ BAD: Import entire package
import * as shared from '@botfleet/shared';
```

---

## 9. Export Organization

### Main Export File
```typescript
// index.ts
/**
 * @botfleet/shared - Lightweight shared code
 *
 * Organized exports for tree-shaking
 */

// Types
export type {
  // Common
  ID,
  UUID,
  Timestamps,
  PaginationMeta,
  PaginatedResult,

  // Monitor
  MonitorEntity,
  MonitorType,
  MonitorStatus,
  CreateMonitorInput,

  // User
  UserEntity,
  UserRole,
} from './types';

// DTOs
export {
  // Monitor
  type MonitorDto,
  type MonitorRealtimeStatusDto,
  transformMonitorToDto,

  // Auth
  type LoginRequestDto,
  type LoginResponseDto,

  // API
  type ApiResponse,
  type ApiError,
} from './dto';

// Validation
export {
  // Schemas
  createMonitorSchema,
  updateMonitorSchema,
  loginSchema,

  // Helpers
  validate,
  validateSafe,
} from './validation';

// Constants
export {
  API_ENDPOINTS,
  HTTP_STATUS,
  ERROR_CODES,
  MONITOR_STATUS,
  DEFAULT_PAGINATION,
} from './constants';

// Utils
export {
  // Date
  timeAgo,
  formatDuration,
  parseDate,

  // Transform
  transformKeys,
  pick,
  omit,

  // Validation
  isValidEmail,
  isValidUrl,
  sanitize,
} from './utils';

// Errors
export {
  BaseError,
  ValidationError,
  NotFoundError,
  UnauthorizedError,
  ForbiddenError,
  QuotaExceededError,
} from './errors';
```

---

## 10. Testing

### Unit Tests
```typescript
// tests/utils/date.utils.test.ts
import { timeAgo, formatDuration } from '@/utils/date.utils';

describe('Date Utils', () => {
  describe('timeAgo', () => {
    it('should format recent time as "just now"', () => {
      const now = new Date();
      expect(timeAgo(now)).toBe('just now');
    });

    it('should format minutes correctly', () => {
      const date = new Date(Date.now() - 5 * 60 * 1000);
      expect(timeAgo(date)).toBe('5 minutes ago');
    });

    it('should handle invalid dates', () => {
      expect(timeAgo('invalid')).toBe('Invalid date');
    });
  });

  describe('formatDuration', () => {
    it('should format milliseconds', () => {
      expect(formatDuration(500)).toBe('500ms');
    });

    it('should format seconds', () => {
      expect(formatDuration(5500)).toBe('5.5s');
    });

    it('should format minutes', () => {
      expect(formatDuration(90000)).toBe('1m');
    });
  });
});
```

### Schema Tests
```typescript
// tests/validation/monitor.schemas.test.ts
import { createMonitorSchema } from '@/validation';

describe('Monitor Schemas', () => {
  describe('createMonitorSchema', () => {
    it('should validate valid input', () => {
      const input = {
        name: 'Test Monitor',
        url: 'https://example.com',
        type: 'http',
      };

      const result = createMonitorSchema.parse(input);

      expect(result).toMatchObject({
        ...input,
        interval: 300, // Default
        timeout: 5000, // Default
      });
    });

    it('should reject invalid URL', () => {
      const input = {
        name: 'Test',
        url: 'not-a-url',
        type: 'http',
      };

      expect(() => createMonitorSchema.parse(input))
        .toThrow('Invalid URL');
    });

    it('should reject invalid interval', () => {
      const input = {
        name: 'Test',
        url: 'https://example.com',
        type: 'http',
        interval: 30, // Too small
      };

      expect(() => createMonitorSchema.parse(input))
        .toThrow('Minimum interval is 60 seconds');
    });
  });
});
```

---

## 📚 Additional Resources

- [Zod Documentation](https://zod.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Bundle Size Optimization](https://web.dev/reduce-javascript-payloads-with-code-splitting/)

---

**Remember**: The Shared package must remain lightweight! Every byte counts for the frontend bundle size.