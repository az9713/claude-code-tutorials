---
name: monitoring
description: Application monitoring, logging, and observability
---

# Monitoring & Observability

## The Three Pillars

1. **Logs** - Discrete events with context
2. **Metrics** - Aggregated numerical data over time
3. **Traces** - Request flow across services

## Structured Logging

### Setup

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ['password', 'token', 'authorization', '*.password'],
});

// Child logger with request context
function requestLogger(req: Request) {
  return logger.child({
    requestId: req.id,
    userId: req.user?.id,
    path: req.path,
    method: req.method,
  });
}
```

### Best Practices

```typescript
// ✅ Structured logging with context
logger.info({ userId, action: 'login', ip: req.ip }, 'User logged in');

// ✅ Include relevant data
logger.error({ err, orderId, userId }, 'Failed to process order');

// ❌ Avoid string concatenation
logger.info(`User ${userId} logged in`);

// ✅ Use appropriate levels
logger.trace('Detailed debugging info');
logger.debug('Debugging info');
logger.info('Normal operation');
logger.warn('Something unexpected');
logger.error('Error occurred');
logger.fatal('System cannot continue');
```

### Request Logging Middleware

```typescript
import { randomUUID } from 'crypto';

function requestLogging(req: Request, res: Response, next: NextFunction) {
  req.id = req.headers['x-request-id'] as string || randomUUID();
  const log = requestLogger(req);

  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    log.info({
      statusCode: res.statusCode,
      duration,
      contentLength: res.get('content-length'),
    }, 'Request completed');
  });

  req.log = log;
  next();
}
```

## Metrics

### Common Metrics

```typescript
import { Counter, Histogram, Gauge, Registry } from 'prom-client';

const registry = new Registry();

// Request counter
const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [registry],
});

// Request duration histogram
const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [registry],
});

// Active connections gauge
const activeConnections = new Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
  registers: [registry],
});

// Middleware
function metricsMiddleware(req: Request, res: Response, next: NextFunction) {
  const end = httpRequestDuration.startTimer({ method: req.method, path: req.route?.path });

  res.on('finish', () => {
    httpRequestsTotal.inc({ method: req.method, path: req.route?.path, status: res.statusCode });
    end({ status: res.statusCode });
  });

  next();
}

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', registry.contentType);
  res.send(await registry.metrics());
});
```

### Key Metrics to Track

| Category | Metrics |
|----------|---------|
| **RED Method** | Rate, Errors, Duration |
| **USE Method** | Utilization, Saturation, Errors |
| **Request** | Count, latency (p50, p95, p99), error rate |
| **Resources** | CPU, memory, disk, network |
| **Business** | Signups, orders, revenue |
| **Dependencies** | Database query time, external API latency |

## Distributed Tracing

### OpenTelemetry Setup

```typescript
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  serviceName: 'my-service',
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Manual span creation
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('my-service');

async function processOrder(orderId: string) {
  return tracer.startActiveSpan('processOrder', async (span) => {
    span.setAttribute('orderId', orderId);

    try {
      await validateOrder(orderId);
      await chargePayment(orderId);
      await sendConfirmation(orderId);
      span.setStatus({ code: SpanStatusCode.OK });
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

## Alerting

### Alert Rules (Prometheus)

```yaml
groups:
  - name: app-alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: High error rate detected
          description: Error rate is {{ $value | humanizePercentage }}

      # Slow responses
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: High latency detected
          description: p95 latency is {{ $value }}s

      # Service down
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: Service is down
```

## Health Checks

```typescript
interface HealthCheck {
  name: string;
  check: () => Promise<{ status: 'healthy' | 'unhealthy'; message?: string }>;
}

const healthChecks: HealthCheck[] = [
  {
    name: 'database',
    check: async () => {
      try {
        await db.query('SELECT 1');
        return { status: 'healthy' };
      } catch (error) {
        return { status: 'unhealthy', message: error.message };
      }
    },
  },
  {
    name: 'redis',
    check: async () => {
      try {
        await redis.ping();
        return { status: 'healthy' };
      } catch (error) {
        return { status: 'unhealthy', message: error.message };
      }
    },
  },
];

app.get('/health', async (req, res) => {
  const results = await Promise.all(
    healthChecks.map(async (hc) => ({
      name: hc.name,
      ...(await hc.check()),
    }))
  );

  const isHealthy = results.every((r) => r.status === 'healthy');
  res.status(isHealthy ? 200 : 503).json({
    status: isHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks: results,
  });
});
```

## Observability Checklist

- [ ] Structured logging with request correlation
- [ ] Sensitive data redacted from logs
- [ ] Key metrics exposed (RED/USE)
- [ ] Distributed tracing enabled
- [ ] Health check endpoints
- [ ] Alerting rules defined
- [ ] Dashboards for key metrics
- [ ] Log aggregation configured
- [ ] Error tracking (Sentry, etc.)
