# App Structure

When I started learning Express, I put everything in a single file. It was fine for a Hello World app. But by the time I had ten routes, three middleware functions, and some database calls, that single file was over 500 lines and becoming a nightmare to navigate.

Good project structure is not about following rules. It is about making your code easy to find, easy to change, and easy to test. Let me share the structure that works for me.

## The MVC Pattern

MVC stands for Model-View-Controller. It is a way of organizing code by responsibility:

- **Model** handles data and business logic. Database queries, data validation, and transformation live here.
- **View** handles presentation. In an API, this is usually JSON responses. In a traditional web app, this is HTML templates.
- **Controller** handles the request/response cycle. It receives a request, calls the model, and returns a response.

Express does not enforce MVC. You could put everything in one file. But MVC gives you a mental framework for organizing your code, and most Express projects follow it loosely.

## Recommended Folder Structure

```
my-express-app/
  server.js              # Entry point, starts the server
  app.js                 # Express app configuration
  package.json
  .env
  .gitignore
  src/
    config/
      index.js           # App configuration from env vars
      database.js        # Database connection setup
    routes/
      index.js           # Route aggregator
      userRoutes.js      # User-related routes
      authRoutes.js      # Authentication routes
    controllers/
      userController.js  # User request handlers
      authController.js  # Auth request handlers
    models/
      User.js            # User data model
    middleware/
      auth.js            # Authentication middleware
      errorHandler.js    # Global error handler
      validate.js        # Input validation middleware
    utils/
      logger.js          # Logging utility
      helpers.js         # General helper functions
  public/
    css/
    js/
    images/
```

## How It Fits Together

Let me show you how a request flows through this structure.

**The route file defines the URL and connects it to a controller:**

```javascript
// src/routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { authenticate } = require('../middleware/auth');

router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', authenticate, userController.updateUser);
router.delete('/:id', authenticate, userController.deleteUser);

module.exports = router;
```

**The controller handles the request logic:**

```javascript
// src/controllers/userController.js
const User = require('../models/User');

exports.getAllUsers = async (req, res, next) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (err) {
    next(err);
  }
};

exports.getUserById = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
};

exports.createUser = async (req, res, next) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
};
```

**The model handles data access:**

```javascript
// src/models/User.js
const db = require('../config/database');

class User {
  static async findAll() {
    const result = await db.query('SELECT id, name, email FROM users');
    return result.rows;
  }

  static async findById(id) {
    const result = await db.query(
      'SELECT id, name, email FROM users WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  }

  static async create(data) {
    const result = await db.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email',
      [data.name, data.email]
    );
    return result.rows[0];
  }
}

module.exports = User;
```

**Routes are aggregated in a central file:**

```javascript
// src/routes/index.js
const express = require('express');
const router = express.Router();

const userRoutes = require('./userRoutes');
const authRoutes = require('./authRoutes');

router.use('/users', userRoutes);
router.use('/auth', authRoutes);

module.exports = router;
```

**And wired into the app:**

```javascript
// app.js
const express = require('express');
const routes = require('./src/routes');

const app = express();
app.use(express.json());
app.use('/api', routes);
app.use(require('./src/middleware/errorHandler'));

module.exports = app;
```

## Why Separate Things?

You might wonder why we do not just put the database query directly in the route handler. Here is why separation matters:

- **Reusability.** The `User.findById` method can be called from a route handler, a scheduled job, or a test without duplicating the query.
- **Testability.** You can test controllers with mock models, and models with a test database. Everything in one file makes testing nearly impossible.
- **Maintainability.** When the database schema changes, you update the model file. You do not have to find every route that queries that table.
- **Readability.** Each file has a single responsibility. When you open a file, you know what it does.

## A Common Mistake

The most common structural mistake I see is putting business logic in route handlers. If your route handler is doing data transformation, validation, or complex conditional logic, that code probably belongs in the model or a separate service file.

A good rule of thumb: route handlers should be thin. They extract data from the request, call a function, and return a response. The "smart" code lives elsewhere.

Start simple, add structure as you need it, and always ask yourself: "If I need to change this later, will I know where to find it?"
