# Health Checks

How do you know if your app is working? You could visit it in a browser, but that does not scale. Health check endpoints give automated systems a way to ask "are you okay?" and get a reliable answer.

## Basic Health Endpoint

The simplest health check just says the app is running:

```js
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});
```

This tells you the process is alive and accepting requests. But it does not tell you if the database is connected or if external services are reachable.

## Checking Dependencies

A proper health check verifies that everything the app needs is available:

```js
// src/routes/health.js
const mongoose = require('mongoose');
const redis = require('../config/redis');
const logger = require('../config/logger');

const healthCheck = async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: Math.floor(process.uptime()),
    version: process.env.npm_package_version || '1.0.0',
    checks: {},
  };

  let isHealthy = true;

  // Check MongoDB
  try {
    const start = Date.now();
    await mongoose.connection.db.admin().ping();
    health.checks.database = {
      status: 'ok',
      responseTime: `${Date.now() - start}ms`,
    };
  } catch (err) {
    health.checks.database = {
      status: 'error',
      message: err.message,
    };
    isHealthy = false;
  }

  // Check Redis (if used)
  if (redis) {
    try {
      const start = Date.now();
      await redis.ping();
      health.checks.redis = {
        status: 'ok',
        responseTime: `${Date.now() - start}ms`,
      };
    } catch (err) {
      health.checks.redis = {
        status: 'error',
        message: err.message,
      };
      isHealthy = false;
    }
  }

  // Check memory usage
  const memory = process.memoryUsage();
  health.checks.memory = {
    status: memory.heapUsed < 500 * 1024 * 1024 ? 'ok' : 'warning',
    heapUsed: `${Math.round(memory.heapUsed / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(memory.heapTotal / 1024 / 1024)}MB`,
  };

  health.status = isHealthy ? 'ok' : 'degraded';

  res.status(isHealthy ? 200 : 503).json(health);
};

module.exports = healthCheck;
```

Now the health endpoint tells me if the database is up, how much memory the app is using, and whether Redis is reachable. If anything is wrong, it returns a 503 status code.

## Readiness vs Liveness

In Kubernetes and similar orchestration systems, there are two types of health checks:

- **Liveness**: Is the process alive? If not, restart it.
- **Readiness**: Can the app handle traffic? If not, stop sending it requests.

I split these into two endpoints:

```js
// Liveness: Is the process running?
app.get('/health/live', (req, res) => {
  res.json({ status: 'ok' });
});

// Readiness: Can the app serve requests?
app.get('/health/ready', async (req, res) => {
  try {
    await mongoose.connection.db.admin().ping();
    res.json({ status: 'ok' });
  } catch {
    res.status(503).json({ status: 'not ready' });
  }
});
```

The liveness check is fast and lightweight. The readiness check verifies the app can actually do its job.

## Kubernetes Probes

In Kubernetes, I configure both probes:

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: 3000
  initialDelaySeconds: 15
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health/ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```

If the liveness probe fails, Kubernetes restarts the container. If the readiness probe fails, Kubernetes stops sending traffic to that pod but does not restart it.

## Monitoring Integration

External monitoring tools (UptimeRobot, Pingdom, Datadog) can ping `/health` on a schedule. If they get a non-200 response, they send you an alert. This catches problems before users notice them.

Health checks are simple to build but incredibly valuable. They are the first thing I add to any production app.
