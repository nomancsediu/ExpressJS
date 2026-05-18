# Global Error Handler

The global error handler is the safety net for your entire app. Every error that passes through `next(err)` ends up here. This is where I centralize my error response logic so I never have to repeat it in individual routes.

## The 4-Parameter Error Middleware

Express recognizes error-handling middleware by its four parameters: `err`, `req`, `res`, `next`. If a middleware has exactly four parameters, Express knows it is an error handler.

```js
app.use((err, req, res, next) => {
  // This is an error-handling middleware
  // It only runs when next(err) is called
});
```

The order matters. This middleware must be defined after all your routes and regular middleware. Otherwise, it will never catch anything.

## Structuring the Response

Here is the global error handler I use:

```js
const AppError = require('../utils/AppError');

app.use((err, req, res, next) => {
  // Default values
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  // Operational errors: trusted, send details to client
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  }

  // Programming or unknown errors: don't leak details
  console.error('UNEXPECTED ERROR:', err);

  return res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  });
});
```

This is the power of the `isOperational` flag. Expected errors get a helpful message. Unexpected errors get a generic response so I do not leak internal details.

## Catch-All for 404s

Before the error handler, I add a catch-all middleware for routes that do not exist:

```js
// After all routes, before error handler
app.all('*', (req, res, next) => {
  next(new AppError(`Cannot find ${req.originalUrl}`, 404));
});
```

When someone hits a URL that does not match any route, this creates a nice 404 error that flows into my global handler.

## The Complete Setup

```js
const express = require('express');
const AppError = require('./utils/AppError');

const app = express();

// ... routes ...

// Catch unmatched routes
app.all('*', (req, res, next) => {
  next(new AppError(`Cannot find ${req.originalUrl}`, 404));
});

// Global error handler (must be last)
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  }

  console.error('UNEXPECTED ERROR:', err);
  return res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  });
});

module.exports = app;
```

With this setup, every error in my app gets handled consistently. No matter where the error comes from, the response always follows the same format.
