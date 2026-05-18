# Synchronous Error Handling

Synchronous errors are the easiest to handle in Express. If an error is thrown in a synchronous route handler, Express catches it automatically and passes it to your error middleware. Let me show you how this works.

## Try/Catch in Routes

The most straightforward way to handle sync errors is with try/catch:

```js
app.get('/users/:id', (req, res, next) => {
  try {
    const user = findUserById(req.params.id);

    if (!user) {
      throw new Error('User not found');
    }

    res.json(user);
  } catch (err) {
    next(err); // Pass to Express error handler
  }
});
```

When I call `next(err)`, Express skips all regular middleware and goes straight to the error-handling middleware (the one with four parameters). This is the standard pattern I use everywhere.

## Express Catches Sync Errors Automatically

Here is something I did not realize at first. If a synchronous error is thrown inside a route handler and I do not wrap it in try/catch, Express still catches it.

```js
app.get('/crash', (req, res) => {
  throw new Error('Something broke!');
  // Express catches this and calls next(err) for you
});
```

Express wraps synchronous handlers and catches thrown errors. So this route will not crash your entire app. The error gets passed to your error middleware if you have one, or Express sends its default HTML error page.

## The Problem: Forgetting Try/Catch

Even though Express catches sync errors automatically, I still use try/catch in my routes. Why? Because it makes the error handling explicit. When I read the code later, I can see exactly where errors are expected and how they flow.

```js
// I prefer this explicit style
app.post('/users', (req, res, next) => {
  try {
    const { name, email } = req.body;

    if (!name || !email) {
      const err = new Error('Name and email are required');
      err.statusCode = 400;
      throw err;
    }

    const user = createUser({ name, email });
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
});
```

## Unhandled Sync Errors

If you do not have an error middleware and a sync error slips through, Express sends a default response. In development, that includes the stack trace. In production, it is a generic 500 page. Neither is great. That is why I always set up a proper global error handler, which I will cover later in this section.

For now, remember: sync errors in Express are manageable. The real trouble starts with async errors, which I will cover next.
