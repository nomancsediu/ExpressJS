# App-Level vs Router-Level Middleware

Express gives you two places to hang middleware: on the app itself, or on a router instance. They work the same way, but their scope is different. Understanding when to use which has helped me organize my projects much better.

## App-Level Middleware

When you use `app.use()`, the middleware applies to every request that hits your server:

```js
const express = require('express');
const app = express();

app.use(express.json());
app.use(morgan('dev'));
app.use(helmet());
```

These run on all routes, no matter what. That is perfect for global concerns like body parsing, logging, and security headers.

You can also scope app-level middleware to a path:

```js
app.use('/api', requireAuth);
```

Now `requireAuth` only runs on routes that start with `/api`.

## Router-Level Middleware

A router is like a mini Express app. It has its own middleware stack, its own routes, and it can be mounted on the main app at a specific path:

```js
const router = express.Router();

// middleware only for this router
router.use(requireAuth);

router.get('/profile', (req, res) => {
  res.json(req.user);
});

router.get('/settings', (req, res) => {
  res.json({ settings: 'your settings' });
});

// mount the router on the app
app.use('/user', router);
```

Now every route in this router requires authentication. Requests to `/user/profile` and `/user/settings` both go through `requireAuth` first.

## When to Use Which

I use app-level middleware for things that every route needs:

```js
app.use(express.json());        // everyone needs body parsing
app.use(morgan('dev'));         // everyone needs logging
app.use(compression());         // everyone benefits from compression
app.use(helmet());              // everyone needs security headers
```

I use router-level middleware for things that only certain route groups need:

```js
const apiRouter = express.Router();
apiRouter.use(rateLimiter);     // only API routes get rate limited

const adminRouter = express.Router();
adminRouter.use(requireAdmin);  // only admin routes need admin check
```

## Organizing with Routers

As your app grows, putting everything in one file becomes a mess. Routers let you split routes into separate modules:

```js
// routes/users.js
const router = express.Router();

router.get('/', getAllUsers);
router.get('/:id', getUser);
router.post('/', createUser);

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/users', usersRouter);
```

Each router file is self-contained. It can have its own middleware and its own error handling. This pattern scales really well. I wish I had learned it earlier instead of cramming everything into `app.js`.

The rule of thumb: if the middleware applies to your entire application, put it on the app. If it applies to a specific group of routes, put it on a router.
