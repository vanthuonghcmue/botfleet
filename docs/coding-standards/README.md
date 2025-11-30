# 📖 Botfleet Coding Standards

> **Version**: 1.1.0
> **Last Updated**: November 2025
> **Status**: Active

## 🎯 Purpose

This document establishes coding standards for the Botfleet monorepo to ensure:
- **Consistency** across all packages
- **Maintainability** as the codebase grows
- **Quality** through automated enforcement
- **Collaboration** with clear guidelines

## 📋 Table of Contents

1. [General Principles](#general-principles)
2. [File Organization](#file-organization)
3. [Naming Conventions](#naming-conventions)
4. [TypeScript Standards](#typescript-standards)
5. [Import Organization](#import-organization)
6. [Error Handling](#error-handling)
7. [Logging Standards](#logging-standards)
8. [Testing Standards](#testing-standards)
9. [Documentation](#documentation)
10. [Git Conventions](#git-conventions)
11. [Code Review Checklist](#code-review-checklist)

---

## 1. General Principles

### Core Values
```typescript
// ✅ GOOD: Clear, simple, and maintainable
async function getMonitorById(id: number): Promise<Monitor> {
  return await monitorRepository.findById(id);
}

// ❌ BAD: Clever but hard to understand
async function gM(i: number) {
  return await mR.fBI(i);
}
```

### Rules
1. **Clarity over cleverness** - Code should be self-explanatory
2. **Consistency over personal preference** - Follow team patterns
3. **Explicit over implicit** - Make intentions clear
4. **Simple over complex** - Start simple, add complexity only when needed

---

## 2. File Organization

### Directory Structure
```
packages/
├── api/
│   ├── src/
│   │   ├── controllers/     # HTTP request handlers
│   │   ├── services/        # Business logic
│   │   ├── repositories/    # Data access layer
│   │   ├── models/          # Database models
│   │   ├── middleware/      # HTTP middleware
│   │   ├── routes/          # Route definitions
│   │   ├── lib/            # Utilities
│   │   └── types/          # TypeScript declarations
│   └── tests/              # Test files
├── frontend/
│   ├── src/
│   │   ├── app/            # Next.js pages
│   │   ├── components/     # React components
│   │   ├── hooks/          # Custom hooks
│   │   ├── lib/           # Utilities
│   │   └── types/         # TypeScript types
│   └── tests/             # Test files
├── worker/
│   ├── src/
│   │   ├── processors/    # Job processors
│   │   ├── services/      # Business logic
│   │   └── lib/          # Utilities
│   └── tests/            # Test files
└── shared/
    └── src/
        ├── types/        # Shared TypeScript types
        ├── dto/          # Data transfer objects
        ├── validation/   # Validation schemas
        └── utils/        # Shared utilities
```

### File Naming
- **Files**: `kebab-case.ts` (e.g., `monitor-service.ts`)
- **Test files**: `*.test.ts` or `*.spec.ts`
- **Type files**: `*.types.ts`
- **Constants**: `*.constants.ts`
- **Configs**: `*.config.ts`

### File Size
- **Maximum lines**: 500 per file (split if larger)
- **Single responsibility**: One main export per file

---

## 3. Naming Conventions

### Variables & Functions
```typescript
// ✅ GOOD: Descriptive camelCase
const monitorCheckInterval = 60000;
function calculateResponseTime(start: number, end: number): number {
  return end - start;
}

// ❌ BAD: Unclear or wrong case
const mci = 60000;
function calc_time(s: number, e: number): number {
  return e - s;
}
```

### Classes & Interfaces
```typescript
// ✅ GOOD: PascalCase with clear names
class MonitorService {
  // Implementation
}

interface MonitorConfig {
  interval: number;
  timeout: number;
}

// ❌ BAD: Wrong case or unclear
class monitor_service {
  // Implementation
}
```

### Constants
```typescript
// ✅ GOOD: SCREAMING_SNAKE_CASE for true constants
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_TIMEOUT_MS = 5000;

// ✅ GOOD: camelCase for config objects
const defaultMonitorConfig = {
  interval: 60000,
  timeout: 5000
};
```

### Enums
```typescript
// ✅ GOOD: PascalCase enum, PascalCase values
enum MonitorStatus {
  Active = 'active',
  Paused = 'paused',
  Deleted = 'deleted'
}

// For string unions (preferred over enums)
type MonitorStatus = 'active' | 'paused' | 'deleted';
```

### Boolean Variables
```typescript
// ✅ GOOD: Clear boolean names with is/has/can prefix
const isActive = true;
const hasPermission = false;
const canEdit = true;

// ❌ BAD: Ambiguous boolean names
const active = true;
const permission = false;
```

---

## 4. TypeScript Standards

### Type Safety
```typescript
// ✅ GOOD: Explicit types
function processMonitor(monitor: Monitor): MonitorResult {
  return {
    id: monitor.id,
    status: monitor.status
  };
}

// ❌ BAD: Using 'any'
function processMonitor(monitor: any): any {
  return monitor;
}
```

### Interface vs Type
```typescript
// ✅ Use interfaces for objects that can be extended
interface MonitorBase {
  id: number;
  name: string;
}

interface HttpMonitor extends MonitorBase {
  url: string;
  method: string;
}

// ✅ Use types for unions, intersections, and utilities
type MonitorStatus = 'active' | 'paused' | 'deleted';
type MonitorWithStatus = Monitor & { status: MonitorStatus };
```

### Strict Mode Rules
- **No implicit any**: All parameters must have types
- **Strict null checks**: Handle null/undefined explicitly
- **No unused variables**: Remove or prefix with `_`
- **Consistent return types**: All code paths must return same type

### Utility Types
```typescript
// ✅ GOOD: Use built-in utility types
type PartialMonitor = Partial<Monitor>;
type ReadonlyMonitor = Readonly<Monitor>;
type MonitorKeys = keyof Monitor;

// Common patterns
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
```

---

## 5. Import Organization

### Import Order
```typescript
// 1. Node.js built-ins
import path from 'path';
import { promises as fs } from 'fs';

// 2. External packages
import fastify from 'fastify';
import { z } from 'zod';

// 3. Internal packages (monorepo)
import { MonitorDto, UserDto } from '@botfleet/shared';

// 4. Absolute imports (same package)
import { MonitorService } from '@/services';
import { logger } from '@/lib/logger';

// 5. Relative imports (same module)
import { validateMonitor } from './utils';
import type { MonitorConfig } from './types';
```

### Import Rules
- **Group imports** with blank lines between groups
- **Sort alphabetically** within each group
- **Use type imports** for type-only imports
- **Prefer named imports** over default imports
- **Use absolute imports** for cross-module imports

---

## 6. Error Handling

### Error Classes
```typescript
// ✅ GOOD: Custom error classes with context
export class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: unknown
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Usage
throw new ValidationError('Invalid URL format', 'url', url);
```

### Error Handling Patterns

#### API Package (Fastify)
```typescript
// ✅ GOOD: Let Fastify handle errors
async function getMonitor(request: FastifyRequest, reply: FastifyReply) {
  const monitor = await monitorService.findById(request.params.id);
  if (!monitor) {
    throw new NotFoundError('Monitor not found');
  }
  return monitor; // Fastify handles the response
}
```

#### Frontend Package
```typescript
// ✅ GOOD: Handle errors at boundaries
function MonitorList() {
  const { data, error, isLoading } = useMonitors();

  if (error) {
    return <ErrorBoundary error={error} />;
  }

  if (isLoading) {
    return <LoadingSpinner />;
  }

  return <MonitorGrid monitors={data} />;
}
```

#### Worker Package
```typescript
// ✅ GOOD: Structured error handling with retries
async function processJob(job: Job) {
  try {
    const result = await performCheck(job.data);
    return result;
  } catch (error) {
    logger.error('Job processing failed', {
      jobId: job.id,
      error: error.message,
      stack: error.stack
    });
    throw error; // BullMQ will retry based on config
  }
}
```

### Never Swallow Errors
```typescript
// ❌ BAD: Silent failure
try {
  await doSomething();
} catch (error) {
  // Don't do this!
}

// ✅ GOOD: Always handle or propagate
try {
  await doSomething();
} catch (error) {
  logger.error('Operation failed', error);
  throw new OperationError('Failed to complete operation', { cause: error });
}
```

---

## 7. Logging Standards

### Log Levels
```typescript
// Use appropriate log levels
logger.debug('Detailed debugging info', { monitorId, checkData });
logger.info('Normal operations', { action: 'monitor.created', monitorId });
logger.warn('Warning conditions', { slowQuery: true, duration: 5000 });
logger.error('Error conditions', { error: err.message, stack: err.stack });
logger.fatal('System failures', { service: 'database', error: 'connection lost' });
```

### Structured Logging
```typescript
// ✅ GOOD: Structured with context
logger.info('Monitor check completed', {
  monitorId: monitor.id,
  responseTime: 245,
  status: 'up',
  location: 'us-east'
});

// ❌ BAD: String concatenation
console.log('Monitor ' + monitor.id + ' is ' + status);
```

### Logging Rules
1. **No console.log** in production code
2. **Include context** in all logs (IDs, user, action)
3. **Use structured format** for machine parsing
4. **Avoid logging sensitive data** (passwords, tokens)
5. **Log at boundaries** (API entry/exit, job start/end)

---

## 8. Testing Standards

### Test Organization
```typescript
// ✅ GOOD: Clear test structure
describe('MonitorService', () => {
  describe('createMonitor', () => {
    it('should create a monitor with valid data', async () => {
      // Arrange
      const monitorData = { name: 'Test', url: 'https://example.com' };

      // Act
      const monitor = await service.createMonitor(monitorData);

      // Assert
      expect(monitor).toMatchObject(monitorData);
      expect(monitor.id).toBeDefined();
    });

    it('should throw ValidationError for invalid URL', async () => {
      // Test implementation
    });
  });
});
```

### Test Naming
- **Test files**: Same name as source with `.test.ts`
- **Test names**: Should describe what is being tested
- **Use "should"** in test descriptions

### Testing Rules
1. **AAA Pattern**: Arrange, Act, Assert
2. **One assertion concept** per test
3. **Test behavior**, not implementation
4. **Mock external dependencies**
5. **Use fixtures** for test data

---

## 9. Documentation

### JSDoc for Public APIs
```typescript
/**
 * Creates a new monitor for the specified team
 *
 * @param teamId - The team ID that owns the monitor
 * @param data - Monitor configuration data
 * @returns The created monitor with generated ID
 * @throws {ValidationError} If monitor data is invalid
 * @throws {QuotaExceededError} If team exceeds monitor limit
 *
 * @example
 * const monitor = await createMonitor(123, {
 *   name: 'API Health Check',
 *   url: 'https://api.example.com/health',
 *   interval: 60
 * });
 */
async function createMonitor(teamId: number, data: CreateMonitorDto): Promise<Monitor> {
  // Implementation
}
```

### Inline Comments
```typescript
// ✅ GOOD: Explain WHY, not WHAT
// Retry with exponential backoff to handle transient network failures
const retryDelay = Math.min(1000 * Math.pow(2, attempt), 30000);

// ❌ BAD: Obvious comments
// Increment counter by 1
counter++;
```

### README Files
Each package should have a README with:
- Purpose and responsibilities
- Setup instructions
- Available scripts
- Architecture decisions
- API documentation links

---

## 10. Git Conventions

### Branch Naming
```bash
feature/add-monitor-dashboard
fix/resolve-memory-leak
chore/update-dependencies
docs/api-documentation
refactor/simplify-auth-flow
```

### Commit Messages
```bash
# Format: <type>(<scope>): <subject>

feat(api): add monitor batch update endpoint
fix(worker): resolve memory leak in job processor
docs(frontend): update component documentation
refactor(shared): simplify validation schemas
chore: update dependencies to latest versions

# Commit body (optional)
- Detailed explanation of changes
- Breaking changes noted with BREAKING CHANGE:
- Reference issues with Fixes #123
```

### Commit Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, etc.)
- **refactor**: Code refactoring
- **perf**: Performance improvements
- **test**: Test changes
- **chore**: Maintenance tasks

---

## 11. Code Review Checklist

### Before Submitting PR
- [ ] Code follows naming conventions
- [ ] All tests pass (`pnpm test`)
- [ ] Linting passes (`pnpm lint`)
- [ ] Types are correct (`pnpm type-check`)
- [ ] Documentation updated if needed
- [ ] No commented-out code
- [ ] No console.log statements
- [ ] No hardcoded values (use constants/config)
- [ ] Error handling is appropriate
- [ ] Logging added for important operations

### Review Focus Areas
1. **Correctness**: Does it solve the problem?
2. **Performance**: Any obvious bottlenecks?
3. **Security**: Any vulnerabilities introduced?
4. **Maintainability**: Will others understand it?
5. **Testing**: Are edge cases covered?
6. **Documentation**: Is it properly documented?

### PR Size Guidelines
- **Small**: < 200 lines (preferred)
- **Medium**: 200-500 lines (acceptable)
- **Large**: > 500 lines (consider splitting)

---

## 🚀 Enforcement

These standards are enforced through:
1. **ESLint** - Automated linting rules
2. **Prettier** - Consistent formatting
3. **TypeScript** - Type checking
4. **Husky** - Pre-commit hooks
5. **CI/CD** - Automated checks on PRs

---

## 📚 References

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

---

**Remember**: These standards are living documents. Propose changes through PRs when you identify improvements.