# Logging and Monitoring

When something goes wrong in production, you need to know about it. Console.log is not enough. You need structured logs, log rotation, and monitoring that alerts you when things break. Let me set up a proper logging and monitoring system.

## Winston for Structured Logging

Winston is the standard logging library for Node.js. It supports multiple transports (console, files, external services) and structured log formats.

### Setup

```js
// src/config/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'ecommerce-api' },
  transports: [
    // Write all logs to console
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),
    // Write info and above to combined.log
    new winston.transports.File({ filename: 'logs/combined.log' }),
    // Write errors only to error.log
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
  ],
});

module.exports = logger;
```

### Using the Logger

Replace `console.log` and `console.error` with the Winston logger:

```js
const logger = require('../config/logger');

// Instead of console.log
logger.info('User registered', { userId: user._id, email: user.email });

// Instead of console.error
logger.error('Payment failed', { orderId: order._id, error: err.message });

// Warning level
logger.warn('Low stock', { productId: product._id, stock: product.stock });

// Debug level (only in development)
logger.debug('Query executed', { query: req.query, duration: ms });
```

Each log entry gets a timestamp, level, and structured data. This makes logs searchable and filterable.

## Log Rotation

Logs grow over time. Without rotation, they fill up your disk. I use `winston-daily-rotate-file`:

```js
const DailyRotateFile = require('winston-daily-rotate-file');

const transport = new DailyRotateFile({
  filename: 'logs/application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  maxSize: '20m',
  maxFiles: '14d',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
});

logger.add(transport);
```

This creates a new log file every day, keeps 14 days of logs, and rotates when a file reaches 20 MB.

## Request Logging

I replace Morgan with a Winston-based request logger so all logs go to the same place:

```js
// src/middleware/requestLogger.js
const logger = require('../config/logger');

module.exports = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info('HTTP Request', {
      method: req.method,
      path: req.originalUrl,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
      userAgent: req.get('user-agent'),
    });
  });

  next();
};
```

## Health Monitoring

A simple health endpoint tells you if the app is alive:

```js
app.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
  };

  try {
    await mongoose.connection.db.admin().ping();
    health.database = 'ok';
  } catch {
    health.database = 'error';
    health.status = 'degraded';
  }

  res.status(health.status === 'ok' ? 200 : 503).json(health);
});
```

## APM with Application Performance Monitoring

For serious production monitoring, I use an APM tool. Some options:

- **New Relic**: Full APM with transaction tracing
- **Datadog**: Infrastructure and application monitoring
- **Sentry**: Error tracking with stack traces and context

Sentry is great for catching errors:

```js
const Sentry = require('@sentry/node');

Sentry.init({ dsn: process.env.SENTRY_DSN });

app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.errorHandler());
```

Now every unhandled error gets reported to Sentry with full context. I get an email or Slack notification when something breaks. No more finding out about errors from angry users.

Logging is not exciting, but it is essential. When production breaks at 3 AM, good logs are the difference between a quick fix and a long nightmare.
