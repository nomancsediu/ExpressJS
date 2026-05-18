# Asynchronous Error Handling

This is where things got confusing for me. Synchronous errors? Express handles them fine. Asynchronous errors? Express does not catch them at all. Let me explain why and what to do about it.

## Why Async Errors Slip Past Express

When a route handler is async (using async/await or callbacks), Express loses its ability to catch errors automatically. The error happens outside of Express's call stack.

```js
// This will NOT be caught by Express
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    throw new Error('User not found'); // This crashes the process!
  }

  res.json(user);
});
```

When that error is thrown inside an async function, it becomes a rejected promise. Express does not handle rejected promises. The request just hangs until it times out, and if you have no unhandled rejection handler, the process crashes.

## The Manual Way: Try/Catch Everywhere

I started by wrapping every async handler in try/catch:

```js
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      const err = new Error('User not found');
      err.statusCode = 404;
      throw err;
    }

    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

This works, but it is tedious. Every single route needs the same try/catch boilerplate. I kept forgetting it, and errors would slip through.

## The Better Way: express-async-errors

Then I found the `express-async-errors` package. It patches Express so that async errors are automatically forwarded to your error middleware.

```js
// Install: npm install express-async-errors
// Put this BEFORE your routes
import 'express-async-errors';

app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    const err = new Error('User not found');
    err.statusCode = 404;
    throw err;
  }

  res.json(user);
});
// No try/catch needed! Errors go to error middleware automatically.
```

One import at the top of your app, and every async error gets caught. This is my preferred approach now.

## The Wrapper Approach

If you do not want to use a patching library, you can write a wrapper function:

```js
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
}));
```

This is clean, explicit, and does not modify Express. Pick whichever approach fits your style. Just make sure you handle async errors somehow. Ignoring them is the fastest way to an unstable app.
