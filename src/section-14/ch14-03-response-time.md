# Response Time

Measuring response time is the first step to improving it. You cannot optimize what you do not measure. I learned this after spending hours "optimizing" code that was not even the bottleneck.

## Measuring Response Time

Express adds a `X-Response-Time` header automatically with the `response-time` middleware:

```bash
npm install response-time
```

```javascript
const responseTime = require('response-time');
app.use(responseTime());
```

Now every response includes a header like `X-Response-Time: 45.3ms`. This is useful for debugging and monitoring.

You can also log slow requests:

```javascript
app.use(responseTime((req, res, time) => {
  if (time > 500) {
    console.warn(`Slow request: ${req.method} ${req.url} took ${time.toFixed(2)}ms`);
  }
}));
```

## Custom Timing Middleware

I sometimes add my own timing middleware for more control:

```javascript
app.use((req, res, next) => {
  const start = process.hrtime.bigint();

  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - start) / 1e6;
    console.log(`${req.method} ${req.path} - ${duration.toFixed(2)}ms`);
  });

  next();
});
```

`process.hrtime.bigint()` gives nanosecond precision, which is more accurate than `Date.now()`.

## Optimizing Route Handlers

Once I can measure, I look for slow handlers. Common fixes:

**1. Avoid unnecessary await:**

```javascript
// Slow: waiting for something you don't need
app.post('/api/orders', async (req, res) => {
  const order = await Order.create(req.body);
  await sendEmail(order); // don't need to wait for this
  res.status(201).json(order);
});

// Better: fire and forget non-critical work
app.post('/api/orders', async (req, res) => {
  const order = await Order.create(req.body);
  sendEmail(order).catch(err => console.error('Email failed:', err));
  res.status(201).json(order);
});
```

**2. Run independent operations in parallel:**

```javascript
// Slow: sequential
app.get('/api/dashboard', async (req, res) => {
  const users = await User.countDocuments();
  const orders = await Order.countDocuments();
  const revenue = await Order.getTotalRevenue();
  res.json({ users, orders, revenue });
});

// Better: parallel
app.get('/api/dashboard', async (req, res) => {
  const [users, orders, revenue] = await Promise.all([
    User.countDocuments(),
    Order.countDocuments(),
    Order.getTotalRevenue(),
  ]);
  res.json({ users, orders, revenue });
});
```

**3. Reduce middleware overhead:**

Only apply middleware where needed. Move route-specific middleware to the route instead of globally:

```javascript
// Instead of applying auth to everything
app.use(authMiddleware);

// Apply only where needed
app.get('/api/public', publicHandler);
app.get('/api/private', authMiddleware, privateHandler);
```

Small changes like these often cut response times in half. The key is measuring first so you optimize the right things.
