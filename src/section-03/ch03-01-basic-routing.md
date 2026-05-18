# Basic Routing

Routing is how your Express app responds to different URLs and HTTP methods. The basic pattern is simple, but there are details worth understanding. Let me walk you through everything.

## The Basic Pattern

Every Express route follows this pattern:

```javascript
app.METHOD(PATH, HANDLER)
```

- **METHOD** is an HTTP method in lowercase: `get`, `post`, `put`, `delete`, `patch`, etc.
- **PATH** is the URL path to match, like `/` or `/users` or `/users/:id`.
- **HANDLER** is a function that runs when the route matches. It receives `req` (request) and `res` (response) objects.

Here are some basic examples:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Home page');
});

app.get('/about', (req, res) => {
  res.send('About page');
});

app.post('/users', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

app.put('/users/1', (req, res) => {
  res.json({ message: 'User updated' });
});

app.delete('/users/1', (req, res) => {
  res.json({ message: 'User deleted' });
});
```

## Multiple Handler Functions

A single route can have multiple handler functions. This is useful when you want to run some checks before the main handler. Each handler must call `next()` to pass control to the next one.

```javascript
// Check if user is authenticated, then get their profile
app.get('/profile',
  (req, res, next) => {
    console.log('Checking authentication...');
    // Pretend we check a session token
    const isAuthenticated = true;
    if (isAuthenticated) {
      next(); // Move to the next handler
    } else {
      res.status(401).json({ error: 'Not authenticated' });
    }
  },
  (req, res) => {
    res.json({ name: 'Alice', email: 'alice@example.com' });
  }
);
```

You can also pass an array of handlers:

```javascript
const authenticate = (req, res, next) => {
  console.log('Authenticating...');
  next();
};

const loadUser = (req, res, next) => {
  req.user = { id: 1, name: 'Alice' };
  next();
};

const sendProfile = (req, res) => {
  res.json(req.user);
};

app.get('/profile', [authenticate, loadUser, sendProfile]);
```

## How Route Matching Works

Express matches routes in the order they are defined. The first route that matches wins. This means order matters.

```javascript
// This route catches everything starting with /users
app.get('/users/*', (req, res) => {
  res.send('Catch-all');
});

// This route will NEVER be reached because the catch-all matches first
app.get('/users/admin', (req, res) => {
  res.send('Admin page');
});
```

The fix is to put more specific routes first:

```javascript
// Specific route first
app.get('/users/admin', (req, res) => {
  res.send('Admin page');
});

// Catch-all after
app.get('/users/*', (req, res) => {
  res.send('Catch-all');
});
```

## Route Matching Details

Here are some things about route matching that surprised me when I first learned them:

**Routes match exact paths by default:**

```javascript
app.get('/about', handler); // Matches /about but NOT /about/ or /about/us
```

**Trailing slashes matter unless configured otherwise:**

```javascript
app.get('/about', handler);  // Matches /about
app.get('/about/', handler);  // Matches /about/

// Disable trailing slash redirect
app.set('strict routing', true);
```

**The path matching is prefix-based for `app.use()`, but exact for `app.METHOD()`:**

```javascript
// app.use matches any path that STARTS with the given path
app.use('/api', router); // Matches /api, /api/users, /api/users/123

// app.get matches the EXACT path
app.get('/api', handler); // Matches /api but NOT /api/users
```

## Responding to Different Methods on the Same Path

You can define different handlers for different HTTP methods on the same path:

```javascript
app.get('/articles', (req, res) => {
  res.json({ action: 'List all articles' });
});

app.post('/articles', (req, res) => {
  res.status(201).json({ action: 'Create article' });
});

app.put('/articles', (req, res) => {
  res.json({ action: 'Update article' });
});

app.delete('/articles', (req, res) => {
  res.json({ action: 'Delete article' });
});
```

## A Special Method: app.all()

`app.all()` matches every HTTP method. It is useful for middleware that should run regardless of the method:

```javascript
// Require authentication for every request to /api/*
app.all('/api/*', (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'Token required' });
  }
  next();
});
```

## The req and res Objects

Every route handler receives the request and response objects. Here are the properties and methods I use most:

**Request (req):**

```javascript
req.params    // URL parameters like /users/:id
req.query     // Query string like ?page=2&sort=name
req.body      // Parsed request body (needs express.json() middleware)
req.headers   // Request headers
req.method    // HTTP method (GET, POST, etc.)
req.url       // Full URL path including query string
req.path      // URL path without query string
req.ip        // Client IP address
req.cookies   // Parsed cookies (needs cookie-parser middleware)
```

**Response (res):**

```javascript
res.send()       // Send a response (auto-detects type)
res.json()       // Send a JSON response
res.status()     // Set the status code
res.redirect()   // Redirect to another URL
res.render()     // Render a template
res.sendFile()   // Send a file
res.cookie()     // Set a cookie
res.clearCookie() // Clear a cookie
res.set()        // Set a response header
res.type()       // Set Content-Type header
```

These are the building blocks. Every route you write will use `req` to get information and `res` to send a response. The more comfortable you are with these objects, the faster you will build routes.
