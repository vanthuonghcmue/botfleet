# ⚙️ Worker Package Coding Standards

> **Package**: `@botfleet/worker`
> **Framework**: Node.js with BullMQ
> **Last Updated**: November 2025

## 📋 Table of Contents

1. [Package Structure](#package-structure)
2. [Job Processor Patterns](#job-processor-patterns)
3. [Queue Management](#queue-management)
4. [Multi-Location Support](#multi-location-support)
5. [Service Patterns](#service-patterns)
6. [Error Handling & Retries](#error-handling--retries)
7. [Performance Monitoring](#performance-monitoring)
8. [Database Access](#database-access)
9. [Logging & Observability](#logging--observability)
10. [Testing](#testing)

---

## 1. Package Structure

```
packages/worker/
├── src/
│   ├── processors/         # Job processors
│   │   ├── monitor-heartbeat.processor.ts
│   │   ├── incident-detection.processor.ts
│   │   └── notification-delivery.processor.ts
│   ├── services/          # Business logic (copied from API)
│   │   ├── monitor.service.ts
│   │   ├── incident.service.ts
│   │   └── notification.service.ts
│   ├── managers/          # Job orchestration
│   │   ├── flow-manager.ts
│   │   └── anomaly-manager.ts
│   ├── adapters/          # External service adapters
│   │   ├── influxdb.adapter.ts
│   │   └── notification.adapters.ts
│   ├── diagnostic/        # Health checks
│   │   ├── ping.ts
│   │   ├── dns.ts
│   │   └── ssl.ts
│   ├── config/           # Configuration
│   │   ├── index.ts
│   │   └── redis.ts
│   ├── lib/             # Utilities
│   │   └── utils.ts
│   └── index.ts        # Worker entry point
└── tests/             # Test files
```

### File Naming Convention
```typescript
// Processors: [job-name].processor.ts
monitor-heartbeat.processor.ts
incident-detection.processor.ts

// Services: [entity].service.ts (match API naming)
monitor.service.ts
incident.service.ts

// Managers: [function]-manager.ts
flow-manager.ts
queue-manager.ts

// Adapters: [service].adapter.ts
influxdb.adapter.ts
smtp.adapter.ts
```

---

## 2. Job Processor Patterns

### Processor Template
```typescript
/**
 * MonitorHeartbeatProcessor - Processes monitor check jobs
 *
 * PATTERN: Processors are stateless job handlers
 * - Single responsibility per processor
 * - Return structured results
 * - Let BullMQ handle retries
 */
import { Job, WorkerOptions } from 'bullmq';
import { Logger } from '@/lib/logger';
import { MonitorCheckData, MonitorCheckResult } from '@botfleet/shared';

export class MonitorHeartbeatProcessor {
  private logger: Logger;

  constructor({ logger }: { logger: Logger }) {
    this.logger = logger;
  }

  /**
   * Process a single monitor check job
   *
   * @param job - BullMQ job with monitor data
   * @returns Check result for storage
   */
  async process(job: Job<MonitorCheckData>): Promise<MonitorCheckResult> {
    const { monitorId, url, type, timeout } = job.data;

    this.logger.info('Processing monitor check', {
      jobId: job.id,
      monitorId,
      attempt: job.attemptsMade,
    });

    try {
      // Perform the check based on type
      const result = await this.performCheck(type, url, timeout);

      // Log success
      this.logger.info('Monitor check completed', {
        jobId: job.id,
        monitorId,
        status: result.status,
        responseTime: result.responseTime,
      });

      return {
        monitorId,
        status: result.status,
        responseTime: result.responseTime,
        statusCode: result.statusCode,
        location: process.env.WORKER_LOCATION || 'default',
        timestamp: new Date().toISOString(),
      };
    } catch (error) {
      // Log error but let it bubble up for BullMQ to handle
      this.logger.error('Monitor check failed', {
        jobId: job.id,
        monitorId,
        error: error.message,
        stack: error.stack,
      });
      throw error; // BullMQ will retry based on config
    }
  }

  private async performCheck(
    type: string,
    url: string,
    timeout: number
  ): Promise<CheckResult> {
    switch (type) {
      case 'http':
        return await this.checkHttp(url, timeout);
      case 'tcp':
        return await this.checkTcp(url, timeout);
      case 'dns':
        return await this.checkDns(url, timeout);
      default:
        throw new Error(`Unsupported monitor type: ${type}`);
    }
  }

  /**
   * Worker configuration for this processor
   */
  static getWorkerOptions(): WorkerOptions {
    return {
      concurrency: 10, // Process 10 jobs in parallel
      limiter: {
        max: 100,
        duration: 1000, // Max 100 jobs per second
      },
    };
  }
}
```

### Processor Rules

#### ✅ DO:
```typescript
// Return structured results
async process(job: Job): Promise<ProcessResult> {
  const result = await doWork(job.data);
  return {
    success: true,
    data: result,
    processedAt: new Date().toISOString(),
  };
}

// Use job progress for long-running tasks
async process(job: Job): Promise<void> {
  await job.updateProgress(10);
  await step1();

  await job.updateProgress(50);
  await step2();

  await job.updateProgress(100);
  return result;
}

// Log with job context
this.logger.info('Processing job', {
  jobId: job.id,
  queue: job.queueName,
  attempt: job.attemptsMade,
});
```

#### ❌ DON'T:
```typescript
// Don't catch and suppress errors
async process(job: Job) {
  try {
    await doWork();
  } catch (error) {
    // ❌ This prevents retries
    console.error(error);
    return { error: error.message };
  }
}

// Don't use global state
let processCount = 0; // ❌ Workers can be restarted

async process(job: Job) {
  processCount++; // ❌ Not reliable
}
```

---

## 3. Queue Management

### Queue Configuration
```typescript
// config/queues.ts
import { Queue, QueueOptions } from 'bullmq';
import { redisConnection } from './redis';

export const QUEUE_NAMES = {
  MONITOR_CHECKS: 'monitor-checks',
  INCIDENT_DETECTION: 'incident-detection',
  NOTIFICATIONS: 'notifications',
  ANOMALY_DETECTION: 'anomaly-detection',
} as const;

export const DEFAULT_JOB_OPTIONS = {
  removeOnComplete: {
    age: 3600, // Keep completed jobs for 1 hour
    count: 100, // Keep last 100 completed jobs
  },
  removeOnFail: {
    age: 86400, // Keep failed jobs for 24 hours
    count: 500, // Keep last 500 failed jobs
  },
  attempts: 3,
  backoff: {
    type: 'exponential' as const,
    delay: 2000, // Start with 2 second delay
  },
};

export function createQueue(name: string): Queue {
  return new Queue(name, {
    connection: redisConnection,
    defaultJobOptions: DEFAULT_JOB_OPTIONS,
  });
}
```

### Worker Registration
```typescript
// index.ts - Worker entry point
import { Worker } from 'bullmq';
import { MonitorHeartbeatProcessor } from './processors';
import { redisConnection } from './config/redis';

export async function startWorker() {
  const location = process.env.WORKER_LOCATION || 'default';
  const queueName = `monitor-checks-${location}`;

  const processor = new MonitorHeartbeatProcessor({
    logger: createLogger(`worker:${location}`),
  });

  const worker = new Worker(
    queueName,
    async (job) => processor.process(job),
    {
      connection: redisConnection,
      ...MonitorHeartbeatProcessor.getWorkerOptions(),
    }
  );

  // Handle worker events
  worker.on('completed', (job) => {
    logger.info('Job completed', {
      jobId: job.id,
      returnValue: job.returnvalue,
    });
  });

  worker.on('failed', (job, error) => {
    logger.error('Job failed', {
      jobId: job?.id,
      error: error.message,
      stack: error.stack,
    });
  });

  // Graceful shutdown
  process.on('SIGTERM', async () => {
    logger.info('Shutting down worker...');
    await worker.close();
    process.exit(0);
  });

  logger.info(`Worker started for location: ${location}`);
}

// Start the worker
startWorker().catch((error) => {
  logger.fatal('Failed to start worker', error);
  process.exit(1);
});
```

---

## 4. Multi-Location Support

### Location-Based Processing
```typescript
// managers/location-manager.ts
export class LocationManager {
  private location: string;
  private region: string;

  constructor() {
    this.location = process.env.WORKER_LOCATION || 'us-east-1';
    this.region = this.location.split('-')[0]; // Extract region
  }

  /**
   * Get queue name for current location
   */
  getQueueName(baseQueue: string): string {
    return `${baseQueue}-${this.location}`;
  }

  /**
   * Check if this worker should process a job
   */
  shouldProcessMonitor(monitor: Monitor): boolean {
    const locations = monitor.worker_locations || ['default'];
    return locations.includes(this.location) ||
           locations.includes(this.region);
  }

  /**
   * Add location metadata to results
   */
  enrichResult<T extends object>(result: T): T & LocationMetadata {
    return {
      ...result,
      location: this.location,
      region: this.region,
      processedBy: `worker-${this.location}-${process.pid}`,
    };
  }
}
```

### Geographic Distribution
```typescript
// Deploy workers in different regions
// us-east-1/worker.ts
process.env.WORKER_LOCATION = 'us-east-1';
process.env.WORKER_REGION = 'us';

// eu-west-1/worker.ts
process.env.WORKER_LOCATION = 'eu-west-1';
process.env.WORKER_REGION = 'eu';

// ap-south-1/worker.ts
process.env.WORKER_LOCATION = 'ap-south-1';
process.env.WORKER_REGION = 'ap';
```

---

## 5. Service Patterns

Services in Worker package are **copied** from API package for independence.

### Service Template
```typescript
/**
 * MonitorService - Business logic for monitor operations
 *
 * NOTE: This is a copy of API's MonitorService
 * Keep in sync manually or through tooling
 * Modifications specific to Worker context are allowed
 */
export class MonitorService {
  private monitorRepository: MonitorRepository;
  private influxAdapter: InfluxDBAdapter;

  constructor({
    monitorRepository,
    influxAdapter,
  }: ServiceDependencies) {
    this.monitorRepository = monitorRepository;
    this.influxAdapter = influxAdapter;
  }

  /**
   * Store heartbeat result
   * Worker-specific: Writes directly to InfluxDB
   */
  async storeHeartbeat(result: MonitorCheckResult): Promise<void> {
    // Write to InfluxDB for real-time metrics
    await this.influxAdapter.writeHeartbeat({
      measurement: 'monitor_heartbeat',
      tags: {
        monitor_id: result.monitorId.toString(),
        location: result.location,
        status: result.status,
      },
      fields: {
        response_time: result.responseTime,
        status_code: result.statusCode || 0,
      },
      timestamp: new Date(result.timestamp),
    });

    // Also update database for persistence
    await this.monitorRepository.updateLastCheck(
      result.monitorId,
      {
        last_check_at: result.timestamp,
        last_status: result.status,
        last_response_time: result.responseTime,
      }
    );
  }
}
```

---

## 6. Error Handling & Retries

### Retry Strategies
```typescript
// processors/base.processor.ts
export abstract class BaseProcessor {
  /**
   * Configure retry strategy based on error type
   */
  protected getRetryOptions(error: Error): RetryOptions {
    // Network errors - retry quickly
    if (error.code === 'ECONNREFUSED' || error.code === 'ETIMEDOUT') {
      return {
        attempts: 5,
        backoff: {
          type: 'fixed',
          delay: 5000, // 5 seconds
        },
      };
    }

    // Rate limiting - exponential backoff
    if (error.message.includes('rate limit')) {
      return {
        attempts: 3,
        backoff: {
          type: 'exponential',
          delay: 10000, // Start with 10 seconds
        },
      };
    }

    // Validation errors - don't retry
    if (error.name === 'ValidationError') {
      return {
        attempts: 1, // No retries
      };
    }

    // Default retry strategy
    return {
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 2000,
      },
    };
  }
}
```

### Error Classification
```typescript
// lib/errors.ts
export class RetryableError extends Error {
  constructor(message: string, public retryAfter?: number) {
    super(message);
    this.name = 'RetryableError';
  }
}

export class FatalError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'FatalError';
  }
}

// Usage in processor
async process(job: Job) {
  try {
    const result = await this.performCheck(job.data);
    return result;
  } catch (error) {
    // Don't retry for fatal errors
    if (error instanceof FatalError) {
      await job.moveToFailed(error, 'fatal');
      throw error;
    }

    // Custom retry delay
    if (error instanceof RetryableError && error.retryAfter) {
      await job.moveToDelayed(Date.now() + error.retryAfter);
      throw error;
    }

    // Let BullMQ handle other errors
    throw error;
  }
}
```

---

## 7. Performance Monitoring

### Metrics Collection
```typescript
// lib/metrics.ts
import { Registry, Histogram, Counter, Gauge } from 'prom-client';

export class WorkerMetrics {
  private registry: Registry;
  private jobDuration: Histogram;
  private jobsProcessed: Counter;
  private jobsFailed: Counter;
  private activeJobs: Gauge;

  constructor() {
    this.registry = new Registry();

    this.jobDuration = new Histogram({
      name: 'worker_job_duration_seconds',
      help: 'Job processing duration in seconds',
      labelNames: ['queue', 'job_type', 'status'],
      buckets: [0.1, 0.5, 1, 2, 5, 10, 30],
      registers: [this.registry],
    });

    this.jobsProcessed = new Counter({
      name: 'worker_jobs_processed_total',
      help: 'Total number of jobs processed',
      labelNames: ['queue', 'job_type', 'status'],
      registers: [this.registry],
    });

    this.activeJobs = new Gauge({
      name: 'worker_active_jobs',
      help: 'Number of currently processing jobs',
      labelNames: ['queue'],
      registers: [this.registry],
    });
  }

  recordJobStart(queue: string): void {
    this.activeJobs.labels(queue).inc();
  }

  recordJobComplete(
    queue: string,
    jobType: string,
    duration: number,
    success: boolean
  ): void {
    const status = success ? 'success' : 'failure';

    this.jobDuration
      .labels(queue, jobType, status)
      .observe(duration / 1000);

    this.jobsProcessed
      .labels(queue, jobType, status)
      .inc();

    this.activeJobs.labels(queue).dec();
  }

  getMetrics(): string {
    return this.registry.metrics();
  }
}
```

### Performance Tracking
```typescript
// processors/monitored.processor.ts
export class MonitoredProcessor extends BaseProcessor {
  private metrics: WorkerMetrics;

  async process(job: Job): Promise<any> {
    const startTime = Date.now();
    this.metrics.recordJobStart(job.queueName);

    try {
      const result = await this.performWork(job);

      this.metrics.recordJobComplete(
        job.queueName,
        job.name,
        Date.now() - startTime,
        true
      );

      return result;
    } catch (error) {
      this.metrics.recordJobComplete(
        job.queueName,
        job.name,
        Date.now() - startTime,
        false
      );

      throw error;
    }
  }
}
```

---

## 8. Database Access

Worker package **copies** models from API package but uses read-heavy patterns.

### Model Usage
```typescript
// Copy models from API package
// models/monitor.model.ts
import { Model, DataTypes } from 'sequelize';

// Same model definition as API package
export class Monitor extends Model {
  // Model definition...
}

// Worker-specific: Add read-optimized scopes
Monitor.addScope('active', {
  where: { status: 'active', active: true },
});

Monitor.addScope('withTeam', {
  include: [
    {
      model: Team,
      attributes: ['id', 'name', 'organization_id'],
    },
  ],
});
```

### Repository Pattern
```typescript
// repositories/monitor.repository.ts
export class MonitorRepository {
  /**
   * Worker-optimized: Batch fetch for processing
   */
  async getMonitorsForProcessing(
    limit: number = 100
  ): Promise<Monitor[]> {
    return await Monitor.scope('active').findAll({
      where: {
        next_check_at: {
          [Op.lte]: new Date(),
        },
      },
      limit,
      order: [['next_check_at', 'ASC']],
      // Lock rows to prevent duplicate processing
      lock: true,
    });
  }

  /**
   * Worker-specific: Update after check
   */
  async updateAfterCheck(
    monitorId: number,
    result: CheckResult
  ): Promise<void> {
    await Monitor.update(
      {
        last_check_at: new Date(),
        last_status: result.status,
        last_response_time: result.responseTime,
        next_check_at: new Date(
          Date.now() + (result.interval * 1000)
        ),
      },
      {
        where: { id: monitorId },
      }
    );
  }
}
```

---

## 9. Logging & Observability

### Structured Logging
```typescript
// lib/logger.ts
import pino from 'pino';

export function createLogger(name: string) {
  return pino({
    name,
    level: process.env.LOG_LEVEL || 'info',
    formatters: {
      level: (label) => ({ level: label }),
    },
    base: {
      worker: process.env.WORKER_LOCATION,
      pid: process.pid,
      hostname: process.env.HOSTNAME,
    },
    timestamp: pino.stdTimeFunctions.isoTime,
  });
}
```

### Logging Patterns
```typescript
// Log at job boundaries
logger.info('Job started', {
  jobId: job.id,
  jobName: job.name,
  queue: job.queueName,
  data: job.data, // Be careful with sensitive data
});

// Log important checkpoints
logger.debug('Monitor check initiated', {
  monitorId,
  checkType,
  url: sanitizeUrl(url), // Remove credentials
});

// Log errors with context
logger.error('Job processing failed', {
  jobId: job.id,
  attempt: job.attemptsMade,
  maxAttempts: job.opts.attempts,
  error: {
    message: error.message,
    code: error.code,
    stack: error.stack,
  },
});

// Log performance metrics
logger.info('Job completed', {
  jobId: job.id,
  duration: Date.now() - startTime,
  memoryUsage: process.memoryUsage(),
});
```

---

## 10. Testing

### Unit Tests
```typescript
// tests/processors/monitor-heartbeat.test.ts
import { Job } from 'bullmq';
import { MonitorHeartbeatProcessor } from '@/processors';

describe('MonitorHeartbeatProcessor', () => {
  let processor: MonitorHeartbeatProcessor;
  let mockLogger: jest.Mocked<Logger>;

  beforeEach(() => {
    mockLogger = createMockLogger();
    processor = new MonitorHeartbeatProcessor({
      logger: mockLogger,
    });
  });

  describe('process', () => {
    it('should process HTTP monitor check', async () => {
      const job = createMockJob({
        data: {
          monitorId: 1,
          type: 'http',
          url: 'https://example.com',
          timeout: 5000,
        },
      });

      const result = await processor.process(job);

      expect(result).toMatchObject({
        monitorId: 1,
        status: 'up',
        responseTime: expect.any(Number),
      });
    });

    it('should retry on network errors', async () => {
      const job = createMockJob({
        data: {
          monitorId: 1,
          type: 'http',
          url: 'https://unreachable.example',
        },
      });

      await expect(processor.process(job))
        .rejects.toThrow('ECONNREFUSED');
    });
  });
});
```

### Integration Tests
```typescript
// tests/integration/worker.test.ts
import { Queue, Worker } from 'bullmq';
import { startWorker } from '@/index';

describe('Worker Integration', () => {
  let queue: Queue;
  let worker: Worker;

  beforeAll(async () => {
    queue = new Queue('test-queue');
    worker = await startWorker({ queue: 'test-queue' });
  });

  afterAll(async () => {
    await worker.close();
    await queue.close();
  });

  it('should process jobs from queue', async () => {
    const job = await queue.add('test-job', {
      monitorId: 1,
      url: 'https://example.com',
    });

    // Wait for job completion
    const result = await job.waitUntilFinished(
      queue.events,
      5000
    );

    expect(result).toMatchObject({
      success: true,
      status: 'up',
    });
  });
});
```

### Load Testing
```typescript
// tests/load/processor.load.test.ts
describe('Processor Load Tests', () => {
  it('should handle 1000 concurrent jobs', async () => {
    const jobs = Array.from({ length: 1000 }, (_, i) => ({
      name: 'monitor-check',
      data: { monitorId: i, url: `https://example.com/${i}` },
    }));

    const results = await queue.addBulk(jobs);

    // Monitor completion
    const completed = await Promise.all(
      results.map(job => job.waitUntilFinished(queue.events))
    );

    expect(completed).toHaveLength(1000);
  });
});
```

---

## 📚 Additional Resources

- [BullMQ Documentation](https://docs.bullmq.io/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Sequelize Documentation](https://sequelize.org/)

---

**Remember**: Workers should be stateless and idempotent. Any worker instance should be able to process any job!