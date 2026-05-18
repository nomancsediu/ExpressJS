# app.js and Server Setup

The `app.js` file is the heart of your Express application. It is where you configure middleware, mount routes, and define how your app behaves. Let me build a complete, production-ready version step by step.

## The Basic Version

Here is where we left off in the Hello World chapter:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, World!');
});

app.listen(3000);
```

This works, but it is not how I would set up a real project. Let me improve it piece by piece.

## Adding Essential Middleware

```javascript
const express = require('express');
const morgan = require('morgan');
const helmet = require('helmet');
const cors = require('cors');
const cookieParser = require('cookie-parser');

const app = express();

// Security headers
app.use(helmet());

// CORS support
app.use(cors());

// Request logging
app.use(morgan('dev'));

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies (form submissions)
app.use(express.urlencoded({ extended: true }));

// Parse cookies
app.use(cookieParser());

// Serve static files
app.use(express.static('public'));

app.get('/', (req, res) => {
  res.json({ message: 'Hello, World!' });
});

module.exports = app;
```

Let me explain each middleware:

- **helmet** sets security-related HTTP headers. It protects against common attacks like clickjacking and XSS. Always use it.
- **cors** enables Cross-Origin Resource Sharing. Without it, browsers block requests from different domains.
- **morgan** logs every request to the console. In development, I use the `dev` format which is colorful and concise. In production, I use `combined`.
- **express.json** parses incoming JSON request bodies and makes them available as `req.body`.
- **express.urlencoded** parses form submissions. The `extended: true` option allows rich objects and arrays.
- **cookieParser** parses cookies from the Cookie header and makes them available as `req.cookies`.
- **express.static** serves files from the `public` directory. A request for `/images/logo.png` serves `public/images/logo.png`.

## App Settings

Express has some built-in settings you can configure:

```javascript
// Set the view engine for template rendering
app.set('view engine', 'ejs');

// Set the views directory
app.set('views', './src/views');

// Enable case-sensitive routing
app.set('case sensitive routing', true);

// Disable the X-Powered-By header (extra security)
app.disable('x-powered-by');
```

Actually, `helmet()` already removes the `X-Powered-By` header, but it is good to know you can do it manually too.

## Mounting Routes

```javascript
const routes = require('./src/routes');

// Mount all API routes under /api
app.use('/api', routes);

// Health check at the root level
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});
```

## Error Handling

Every production app needs error handling. Express expects error-handling middleware to have four parameters:

```javascript
// 404 handler - no route matched
app.use((req, res) => {
  res.status(404).json({
    error: 'Not Found',
    message: `Cannot ${req.method} ${req.path}`
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);

  const statusCode = err.statusCode || 500;
  const message = process.env.NODE_ENV === 'production'
    ? 'Internal Server Error'
    : err.message;

  res.status(statusCode).json({
    error: err.name || 'InternalServerError',
    message: message
  });
});
```

In production, I never send the actual error message to the client because it might reveal internal details. In development, I want to see the real error.

## The Complete app.js

Here is the full version:

```javascript
const express = require('express');
const morgan = require('morgan');
const helmet = require('helmet');
const cors = require('cors');
const cookieParser = require('cookie-parser');

const routes = require('./src/routes');

const app = express();

// Security
app.use(helmet());
app.disable('x-powered-by');

// CORS
app.use(cors({
  origin: process.env.CORS_ORIGIN || '*',
  credentials: true
}));

// Logging
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
} else {
  app.use(morgan('combined'));
}

// Body parsing
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// Static files
app.use(express.static('public'));

// Routes
app.use('/api', routes);

app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    error: 'Not Found',
    message: `Cannot ${req.method} ${req.path}`
  });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  const statusCode = err.statusCode || 500;
  const message = process.env.NODE_ENV === 'production'
    ? 'Internal Server Error'
    : err.message;

  res.status(statusCode).json({
    error: err.name || 'InternalServerError',
    message
  });
});

module.exports = app;
```

## server.js With Graceful Shutdown

The `server.js` file starts the app and handles shutdown properly:

```javascript
const app = require('./app');
const dotenv = require('dotenv');

dotenv.config();

const PORT = process.env.PORT || 3000;

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT} in ${process.env.NODE_ENV || 'development'} mode`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});

process.on('SIGINT', () => {
  console.log('SIGINT received. Shutting down gracefully...');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.error('Unhandled Rejection:', err);
  server.close(() => process.exit(1));
});

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  server.close(() => process.exit(1));
});
```

Graceful shutdown is important in production. When you deploy a new version, the hosting platform sends a SIGTERM signal. Your server should stop accepting new requests, finish handling the ones in progress, and then exit cleanly. Without this, active requests get cut off mid-response.

The `unhandledRejection` and `uncaughtException` handlers are safety nets. They log the error and shut down the process. In production, a process manager like PM2 or systemd would restart it automatically.
