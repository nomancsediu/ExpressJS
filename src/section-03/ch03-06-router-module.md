# The Router Module

As your Express app grows, putting all routes in `app.js` becomes unmanageable. The `express.Router()` class solves this by letting you create modular, mountable route handlers. I consider it essential for any project with more than a handful of routes.

## What is express.Router()

A Router is like a mini Express application. It has its own middleware stack and routes, but it cannot listen on a port by itself. You mount it onto your main app at a specific path.

Think of it this way: `app` is the main application, and each `Router` is a sub-application focused on one feature.

## Creating a Basic Router

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();

// Define routes on the router instead of app
router.get('/', (req, res) => {
  res.json({ message: 'List all users' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get user ${req.params.id}` });
});

router.post('/', (req, res) => {
  res.status(201).json({ message: 'Create user' });
});

router.put('/:id', (req, res) => {
  res.json({ message: `Update user ${req.params.id}` });
});

router.delete('/:id', (req, res) => {
  res.json({ message: `Delete user ${req.params.id}` });
});

module.exports = router;
```

Then mount it in your main app:

```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/userRoutes');

const app = express();
app.use(express.json());

// Mount the user routes at /api/users
app.use('/api/users', userRoutes);

app.listen(3000);
```

Now these routes are available:

```
GET    /api/users       -> List all users
GET    /api/users/42    -> Get user 42
POST   /api/users       -> Create user
PUT    /api/users/42    -> Update user 42
DELETE /api/users/42    -> Delete user 42
```

The router does not know about the `/api/users` prefix. It only sees `/` and `/:id`. The main app adds the prefix when mounting. This separation means you can change the prefix without touching the router code.

## Router-Level Middleware

Routers can have their own middleware that only applies to routes within that router:

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();

// This middleware only runs for user routes
router.use((req, res, next) => {
  console.log(`User route accessed: ${req.method} ${req.url}`);
  next();
});

// Require authentication for all user routes
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  next();
};

router.use(authenticate);

router.get('/', (req, res) => {
  res.json({ users: [] });
});

module.exports = router;
```

This is powerful because it means different routers can have different middleware. Your auth routes might not need authentication, but your user routes do. Your admin routes might need extra authorization checks. Each router handles its own concerns.

## Mounting Multiple Routers

Here is how I organize a medium-sized application:

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

const authRoutes = require('./authRoutes');
const userRoutes = require('./userRoutes');
const postRoutes = require('./postRoutes');
const commentRoutes = require('./commentRoutes');
const adminRoutes = require('./adminRoutes');

router.use('/auth', authRoutes);
router.use('/users', userRoutes);
router.use('/posts', postRoutes);
router.use('/comments', commentRoutes);
router.use('/admin', adminRoutes);

module.exports = router;
```

```javascript
// app.js
const express = require('express');
const routes = require('./routes');

const app = express();
app.use(express.json());
app.use('/api', routes);

app.listen(3000);
```

Resulting URL structure:

```
/api/auth/login
/api/auth/register
/api/users/
/api/users/:id
/api/posts/
/api/posts/:id
/api/posts/:id/comments
/api/admin/dashboard
```

## Nested Routers

Routers can be nested inside other routers. This is useful for resources that belong to other resources:

```javascript
// routes/postRoutes.js
const express = require('express');
const router = express.Router();
const commentRoutes = require('./commentRoutes');

router.get('/', (req, res) => {
  res.json({ message: 'List all posts' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get post ${req.params.id}` });
});

// Nest comment routes under posts
router.use('/:postId/comments', commentRoutes);

module.exports = router;
```

```javascript
// routes/commentRoutes.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  const { postId } = req.params;
  res.json({ message: `List comments for post ${postId}` });
});

router.post('/', (req, res) => {
  const { postId } = req.params;
  res.status(201).json({ message: `Create comment on post ${postId}` });
});

router.get('/:commentId', (req, res) => {
  const { postId, commentId } = req.params;
  res.json({ message: `Comment ${commentId} on post ${postId}` });
});

module.exports = router;
```

Now these URLs work:

```
GET  /api/posts/5/comments        -> List comments on post 5
POST /api/posts/5/comments        -> Add comment to post 5
GET  /api/posts/5/comments/12     -> Get comment 12 on post 5
```

The `postId` parameter from the parent router is available in the child router. Express passes parameters through the chain automatically.

## Router Options

The `express.Router()` function accepts an options object:

```javascript
const router = express.Router({
  caseSensitive: true,   // Routes are case-sensitive
  mergeParams: false,     // Do not share req.params from parent router
  strict: false           // /foo and /foo/ are treated the same
});
```

The `mergeParams` option is important for nested routers. By default, it is `false`, which means a nested router does not see the parent's `req.params`. If you need them, set `mergeParams: true`:

```javascript
const router = express.Router({ mergeParams: true });
```

Wait, actually Express does pass params from parent routers by default in recent versions. But if you encounter a case where `req.params` from a parent router is missing, check the `mergeParams` option.

## A Real-World Example

Here is how I structure routes for a blog application:

```javascript
// app.js
const express = require('express');
const app = express();
const apiRoutes = require('./routes/api');

app.use(express.json());
app.use('/api', apiRoutes);

app.listen(3000);

// routes/api.js
const router = express.Router();
const authRoutes = require('./auth');
const blogRoutes = require('./blog');

router.use('/auth', authRoutes);
router.use('/blog', blogRoutes);

module.exports = router;

// routes/blog.js
const router = express.Router();
const postRoutes = require('./posts');
const categoryRoutes = require('./categories');

router.use('/posts', postRoutes);
router.use('/categories', categoryRoutes);

module.exports = router;
```

This gives me a clean, two-level URL structure: `/api/blog/posts`, `/api/blog/categories`, `/api/auth/login`. Each router file is small and focused on one feature. The `express.Router()` is what makes this possible.
