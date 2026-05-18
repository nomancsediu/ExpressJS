# Middleware Order and next()

One of the most important things I learned about Express is that order matters. Middleware runs in the order you add it. If you put things in the wrong order, nothing works the way you expect.

## Order Matters

```js
// This works: parsing happens before the route
app.use(express.json());
app.post('/data', (req, res) => {
  console.log(req.body); // the parsed JSON
  res.send('OK');
});
```

```js
// This breaks: the route runs before parsing
app.post('/data', (req, res) => {
  console.log(req.body); // undefined!
  res.send('OK');
});
app.use(express.json()); // too late
```

Express processes middleware top to bottom. If a route matches before your body parser runs, the request body will not be parsed yet. I cannot count how many times I debugged `undefined` body issues only to find the parser was registered after the route.

## How next() Works

Calling `next()` hands control to the next middleware or route in the stack. Calling `next('route')` skips the rest of the current middleware stack for that route. Calling `next(err)` jumps straight to error-handling middleware.

```js
app.get('/admin',
  (req, res, next) => {
    if (!req.user) {
      next(new Error('Unauthorized'));
    } else {
      next(); // continue to the next handler
    }
  },
  (req, res) => {
    res.send('Admin panel');
  }
);
```

If `next(err)` is called, Express skips all regular middleware and jumps to the error handler.

## Early Returns

When you send a response inside middleware, you must make sure the rest of the code does not keep running:

```js
// Bug: both res.send and next() can run
const badMiddleware = (req, res, next) => {
  if (!req.headers.authorization) {
    res.status(401).send('Unauthorized');
    // forgot to return!
  }
  next(); // this still runs!
};
```

The fix is simple:

```js
const goodMiddleware = (req, res, next) => {
  if (!req.headers.authorization) {
    return res.status(401).send('Unauthorized');
  }
  next();
};
```

Adding `return` before `res.send()` or `res.json()` ensures the function exits. This is not a Express feature. It is just JavaScript. But forgetting it causes confusing bugs.

## Passing Errors

When something goes wrong in your middleware, pass the error to `next()`:

```js
const validateInput = (req, res, next) => {
  if (!req.body.email) {
    return next(new Error('Email is required'));
  }
  next();
};
```

Express will skip all remaining regular middleware and jump to your error handler. We will cover error-handling middleware in detail in the next chapter.

The takeaway: always be intentional about the order of your middleware and always use `return` when you send a response early.
