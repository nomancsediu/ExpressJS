# Error Logging

When my app breaks at 2 AM, logs are all I have. Console.log is fine during development, but in production I need structured, persistent logs. That is where Winston comes in.

## Why Winston?

Winston is the most popular logging library for Node.js. It supports multiple transports (console, files, external services), log levels, and custom formats. It gives me everything I need to track and debug errors.

## Basic Setup

```js
const winston = require('winston');

const logger = winston.createLogger({
  level: 'error',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
  ],
});

// Also log to console in development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}
```

## Log Format

I use JSON format for my logs. It makes them searchable and structured:

```json
{
  "level": "error",
  "message": "Database connection failed",
  "timestamp": "2024-03-15T10:30:00.000Z",
  "stack": "Error: Database connection failed\n    at connectDB...",
  "path": "/api/users",
  "method": "GET"
}
```

## Integrating with the Error Handler

I plug Winston into my global error middleware:

```js
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;

  // Log the error with request context
  logger.error({
    message: err.message,
    stack: err.stack,
    statusCode: err.statusCode,
    path: req.originalUrl,
    method: req.method,
    body: req.body,
    isOperational: err.isOperational || false,
  });

  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  }

  return res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  });
});
```

## Storing Logs

In production, I send logs to a file and also consider an external service:

```js
// File rotation to prevent huge log files
const DailyRotateFile = require('winston-daily-rotate-file');

logger.add(new DailyRotateFile({
  filename: 'logs/application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  maxSize: '20m',
  maxFiles: '14d', // Keep 14 days of logs
}));
```

## Alerting

For critical errors, I want to know right away. I can add a transport that sends alerts:

```js
// Simple email alert for critical errors
const mailTransport = new winston.transports.Stream({
  stream: {
    write: (message) => {
      const log = JSON.parse(message);
      if (log.level === 'error' && !log.isOperational) {
        sendAlertEmail(log); // Your alert function
      }
    },
  },
});

logger.add(mailTransport);
```

Good logging takes time to set up but saves hours of debugging. I learned this the hard way. Start logging from day one, not after your first production incident.
