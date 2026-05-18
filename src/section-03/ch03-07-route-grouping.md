# Route Grouping

As your application grows, you will have dozens of routes. Without organization, finding the right route file becomes a scavenger hunt. Route grouping is about organizing routes by feature so that related code stays together and your project stays manageable.

## Why Grouping Matters

I once worked on a project where every route was in a single file. It had 47 routes. Finding a specific route meant scrolling through hundreds of lines. Adding a new route meant figuring out where to put it. Debugging meant searching through a wall of code.

Grouping routes by feature solved all of those problems. When I need to work on authentication, I open the auth routes file. When I need to add a new blog feature, I go to the blog routes. Each file is small, focused, and easy to navigate.

## Feature-Based Organization

The key principle is: group by feature, not by technical role. A "feature" is a coherent piece of functionality that a user interacts with. Authentication is a feature. User management is a feature. Blog posts are a feature.

Here is the folder structure I use:

```
src/
  routes/
    index.js              # Route aggregator
    auth.routes.js        # Login, register, logout, password reset
    user.routes.js        # User CRUD, profile
    post.routes.js        # Blog posts CRUD
    comment.routes.js     # Comments on posts
    admin.routes.js       # Admin-only routes
    public.routes.js      # Public pages, no auth required
```

Each file handles all routes for its feature:

```javascript
// src/routes/auth.routes.js
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');

router.post('/login', authController.login);
router.post('/register', authController.register);
router.post('/logout', authController.logout);
router.post('/forgot-password', authController.forgotPassword);
router.post('/reset-password', authController.resetPassword);

module.exports = router;
```

```javascript
// src/routes/user.routes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/', userController.listUsers);
router.get('/:id', userController.getUser);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);
router.get('/:id/posts', userController.getUserPosts);

module.exports = router;
```

## The Route Aggregator

The `index.js` file brings all routes together:

```javascript
// src/routes/index.js
const express = require('express');
const router = express.Router();

const authRoutes = require('./auth.routes');
const userRoutes = require('./user.routes');
const postRoutes = require('./post.routes');
const commentRoutes = require('./comment.routes');
const adminRoutes = require('./admin.routes');
const publicRoutes = require('./public.routes');

// Public routes (no authentication)
router.use('/', publicRoutes);

// Auth routes
router.use('/auth', authRoutes);

// Protected routes (authentication required)
router.use('/users', userRoutes);
router.use('/posts', postRoutes);
router.use('/comments', commentRoutes);

// Admin routes (authentication + admin role required)
router.use('/admin', adminRoutes);

module.exports = router;
```

And the main app mounts the aggregator:

```javascript
// app.js
const express = require('express');
const routes = require('./src/routes');

const app = express();
app.use(express.json());
app.use('/api', routes);

app.listen(3000);
```

## Middleware Per Group

Different route groups often need different middleware. Public routes need no authentication. Protected routes need authentication. Admin routes need authentication and authorization. Here is how I handle that:

```javascript
// src/middleware/auth.js
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  // Verify token and attach user to request
  req.user = { id: 1, role: 'user' }; // Simplified
  next();
};

const requireAdmin = (req, res, next) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access required' });
  }
  next();
};

module.exports = { authenticate, requireAdmin };
```

```javascript
// src/routes/index.js
const { authenticate, requireAdmin } = require('../middleware/auth');

// Public routes
router.use('/', publicRoutes);

// Auth routes (no auth needed to log in)
router.use('/auth', authRoutes);

// Apply authentication to all routes below this line
router.use(authenticate);

// Protected routes (auth required)
router.use('/users', userRoutes);
router.use('/posts', postRoutes);
router.use('/comments', commentRoutes);

// Admin routes (auth + admin required)
router.use('/admin', requireAdmin, adminRoutes);
```

Because middleware runs in order, placing `router.use(authenticate)` before the protected routes means every request to those routes goes through authentication first. No need to add `authenticate` to every single route.

## Naming Conventions

Consistent naming makes your project easier to navigate. Here are the conventions I follow:

**File names:**
- Use kebab-case or dot notation: `user-routes.js` or `user.routes.js`
- Be consistent. Pick one style and stick with it
- Include the type in the name: `user.routes.js`, `user.controller.js`, `user.model.js`

**Route paths:**
- Use plural nouns for collections: `/users`, `/posts`, `/comments`
- Use kebab-case for multi-word paths: `/blog-posts`, `/password-reset`
- Be consistent with singular vs plural. I prefer plural for collections

**URL patterns for a resource:**
```
GET    /users           List users
POST   /users           Create user
GET    /users/:id       Get one user
PUT    /users/:id       Replace user
PATCH  /users/:id       Update user
DELETE /users/:id       Delete user
```

## A Complete Example

Here is a full project with properly grouped routes:

```
src/
  routes/
    index.js
    auth.routes.js
    user.routes.js
    product.routes.js
    order.routes.js
```

```javascript
// src/routes/product.routes.js
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');
const { validateProduct } = require('../middleware/validation');

router.get('/', productController.listProducts);
router.get('/:id', productController.getProduct);
router.post('/', validateProduct, productController.createProduct);
router.put('/:id', validateProduct, productController.updateProduct);
router.delete('/:id', productController.deleteProduct);

module.exports = router;
```

```javascript
// src/routes/order.routes.js
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');

router.get('/', orderController.listOrders);
router.get('/:id', orderController.getOrder);
router.post('/', orderController.createOrder);
router.patch('/:id/status', orderController.updateOrderStatus);

module.exports = router;
```

Route grouping is not just about organization. It is about making your project scalable. When you add a new feature, you create a new route file and add one line to the aggregator. When you need to find a bug, you go straight to the relevant file. When a new developer joins the team, they can understand the project structure in minutes, not hours.
