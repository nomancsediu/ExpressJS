# Production vs Development Errors

The error response I send during development is very different from what I send in production. In development, I want every detail I can get. In production, I want to hide everything from the user. Getting this wrong can leak sensitive information to attackers.

## The Problem with Verbose Errors

During development, a stack trace is helpful. In production, it is dangerous:

```
Error: Cannot read property 'password' of undefined
    at UserController.login (/app/controllers/user.js:45:12)
    at Layer.handle (/app/node_modules/express/lib/router/layer.js:95:5)
```

This tells an attacker my file structure, the framework I use, and that I have a password field. That is a lot of information I do not want to share.

## Different Responses Per Environment

I check `NODE_ENV` in my global error handler and adjust the response:

```js
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    // Send full error details
    return res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack,
    });
  }

  // Production: only send safe details
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  }

  // Unknown errors: send generic message
  console.error('UNEXPECTED ERROR:', err);
  return res.status(500).json({
    status: 'error',
    message: 'Something went wrong',
  });
});
```

## Hiding Stack Traces

Notice how the development response includes `stack` and the full `error` object, while production only includes the `message` for operational errors and a generic message for everything else. Stack traces never reach the client in production.

## Common Mistakes I Made

**Forgetting to set NODE_ENV**: If you do not set `NODE_ENV=production`, Express runs in development mode by default. That means verbose error messages in production. Always set it.

```bash
# In your production environment
export NODE_ENV=production
node app.js
```

**Leaking database errors**: When a database query fails, the error object might contain your query, table names, or connection details. Never send raw database errors to the client.

```js
// Bad: leaking database error details
res.status(500).json({ error: err.message });

// Good: generic message
res.status(500).json({ message: 'Something went wrong' });
```

**Showing error codes for everything**: Even `isOperational` errors should be reviewed. A "duplicate key error" from MongoDB might reveal that an email already exists in your system, which is useful for an attacker doing enumeration.

## A Cleaner Approach

I refactor the error handler into a dedicated function:

```js
const sendErrorDev = (err, res) => {
  res.status(err.statusCode).json({
    status: err.status,
    error: err,
    message: err.message,
    stack: err.stack,
  });
};

const sendErrorProd = (err, res) => {
  if (err.isOperational) {
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  } else {
    console.error('ERROR:', err);
    res.status(500).json({
      status: 'error',
      message: 'Something went wrong',
    });
  }
};

app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, res);
  } else {
    sendErrorProd(err, res);
  }
});
```

This keeps the logic clear and makes it easy to add more environments later, like staging. The rule is simple: be generous with error details during development, be stingy in production.
