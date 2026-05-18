# Graceful Shutdown

When you deploy a new version, the old one needs to stop. When the server restarts, your app needs to shut down. The question is: do you yank the power cord or do you close things properly? Graceful shutdown means closing connections, finishing in-progress requests, and cleaning up before the process exits.

## Why It Matters

Without graceful shutdown, here is what can happen:

- In-progress HTTP requests get cut off mid-response
- Database connections are abandoned instead of closed
- File handles and timers are left dangling
- Users see errors and failed requests during every deploy

With graceful shutdown, existing requests finish, connections close properly, and the process exits cleanly.

## Handling Signals

Node.js receives signals from the operating system when it needs to stop. The important ones:

- **SIGTERM**: "Please shut down gracefully" (used by Docker, PM2, Kubernetes)
- **SIGINT**: "Stop now" (Ctrl+C in the terminal)

I listen for both:

```js
// src/server.js
const dotenv = require('dotenv');
dotenv.config();

const app = require('./app');
const { connectDB } = require('./config/db');
const logger = require('./config/logger');

const PORT = process.env.PORT || 3000;

let server;

connectDB().then(() => {
  server = app.listen(PORT, () => {
    logger.info(`Server running on port ${PORT}`);
  });
});

// Graceful shutdown handler
const shutdown = async (signal) => {
  logger.info(`${signal} received. Shutting down gracefully...`);

  // Stop accepting new connections
  server.close(() => {
    logger.info('HTTP server closed');
  });

  // Close database connection
  try {
    await mongoose.connection.close();
    logger.info('Database connection closed');
  } catch (err) {
    logger.error('Error closing database', { error: err.message });
  }

  // Force exit after timeout
  setTimeout(() => {
    logger.error('Forced shutdown after timeout');
    process.exit(1);
  }, 10000);

  // If all connections close before timeout, exit cleanly
  server.on('close', () => {
    process.exit(0);
  });
};

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

## The Step-by-Step Process

1. **Receive signal**: SIGTERM or SIGINT arrives
2. **Stop accepting new requests**: `server.close()` prevents new connections
3. **Let in-progress requests finish**: Existing connections stay open until they complete
4. **Close database connections**: `mongoose.connection.close()` releases database resources
5. **Exit**: Process exits with code 0 (success) or 1 (forced)

## Connection Draining

The `server.close()` method stops new connections but lets existing ones finish. I can add a timeout for long-running requests:

```js
const shutdown = async (signal) => {
  logger.info(`${signal} received`);

  // Give in-progress requests 10 seconds to finish
  setTimeout(() => {
    logger.error('Timeout reached, forcing shutdown');
    process.exit(1);
  }, 10000);

  server.close(() => {
    logger.info('All connections closed');
    mongoose.connection.close(false, () => {
      logger.info('Database disconnected');
      process.exit(0);
    });
  });
};
```

If all requests finish within 10 seconds, the process exits cleanly. If not, it is forced to exit. Either way, the app stops.

## Unhandled Rejection and Exception

I also catch unhandled promise rejections and uncaught exceptions. These should crash the process, but gracefully:

```js
process.on('unhandledRejection', (err) => {
  logger.error('Unhandled Rejection', { error: err });
  server.close(() => process.exit(1));
});

process.on('uncaughtException', (err) => {
  logger.error('Uncaught Exception', { error: err });
  server.close(() => process.exit(1));
});
```

Graceful shutdown is about respect. Respect for your users who have in-progress requests, and respect for your infrastructure that needs clean exits. Every production Express app should handle this.
