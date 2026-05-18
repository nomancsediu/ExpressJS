# Error-Handling Middleware

Error-handling middleware is special. It looks different from regular middleware because it has four parameters instead of three. Express uses that extra first parameter to know this is an error handler.

## The Four-Parameter Signature

```js
function errorHandler(err, req, res, next) {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
}

app.use(errorHandler);
```

That first `err` parameter is what makes Express treat this as error middleware. If you only have three parameters, Express thinks it is regular middleware even if you name the first parameter `err`. Yes, I made that mistake too.

## How Errors Reach the Handler

There are a few ways errors get to your error middleware:

```js
// 1. Pass an error to next()
app.get('/broken', (req, res, next) => {
  const err = new Error('Something failed');
  next(err);
});

// 2. Throw inside an async function (Express 5 handles this)
app.get('/async-broken', async (req, res) => {
  throw new Error('Async failure');
});

// 3. Synchronous throws work in Express 4 too
app.get('/sync-broken', (req, res) => {
  throw new Error('Sync failure');
});
```

In Express 5, rejected promises in route handlers are automatically caught and forwarded to error middleware. In Express 4, you had to catch them manually and call `next(err)`.

## Sending Proper Error Responses

A good error handler gives the client useful information without leaking internals:

```js
function errorHandler(err, req, res, next) {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational
    ? err.message
    : 'Internal server error';

  res.status(statusCode).json({
    status: 'error',
    message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
}
```

In development, I include the stack trace so I can debug. In production, I hide it.

## Operational vs Programming Errors

This distinction changed how I think about errors:

- **Operational errors** are expected problems: invalid input, missing resources, network failures. These are not bugs. They are situations you should handle gracefully.
- **Programming errors** are bugs: calling a method on `undefined`, typos in variable names, logic mistakes. These should crash the process in production so a restart can fix things.

I create a custom error class to mark operational errors:

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Usage
app.get('/users/:id', (req, res, next) => {
  const user = findUser(req.params.id);
  if (!user) {
    return next(new AppError('User not found', 404));
  }
  res.json(user);
});
```

This way, my error handler knows whether to show the message or hide it. Clean and safe.

Always put your error-handling middleware last. It only catches errors from middleware registered before it.
