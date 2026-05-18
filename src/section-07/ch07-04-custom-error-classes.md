# Custom Error Classes

Throwing plain `Error` objects works, but it limits what I can do. When every error is just a message string, I lose important context like status codes, error types, and whether the error is something I expected. That is why I create custom error classes.

## The AppError Class

This is the base custom error class I use in all my projects:

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);

    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

Let me break down what each property does:

- **statusCode**: The HTTP status code to send back (400, 404, 500, etc.)
- **status**: A readable label. Client errors are "fail", server errors are "error"
- **isOperational**: Marks this as an error I expected and handled, not a programming bug

The `isOperational` flag is the key insight. It lets my global error handler distinguish between errors I anticipated (bad user input, missing resource) and unexpected bugs (undefined variable, type error).

## Using AppError in Routes

```js
const AppError = require('../utils/AppError');

app.get('/users/:id', async (req, res, next) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    return next(new AppError('No user found with that ID', 404));
  }

  res.json(user);
});

app.post('/users', (req, res, next) => {
  const { name, email } = req.body;

  if (!name || !email) {
    return next(new AppError('Please provide name and email', 400));
  }

  // Create user...
});
```

Clean and readable. I can see at a glance what went wrong and what status code goes with it.

## Specific Error Types

For common scenarios, I create subclasses:

```js
class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403);
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed') {
    super(message, 400);
  }
}
```

Now my route code reads like plain English:

```js
app.get('/posts/:id', async (req, res, next) => {
  const post = await Post.findById(req.params.id);
  if (!post) return next(new NotFoundError('Post'));
  res.json(post);
});
```

Custom error classes make error handling consistent, readable, and easy to maintain. I never throw plain `Error` objects anymore.
