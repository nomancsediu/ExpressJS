# Named Routes and URL Building

Hardcoding URLs in your code is a habit that will cause you pain. When you change a route path, you have to find and update every place that references it. Miss one, and something breaks. Named routes solve this by giving your routes identifiers and building URLs from those identifiers instead of writing paths by hand.

## The Problem With Hardcoded URLs

Imagine this scenario:

```javascript
// Route definition
app.get('/api/v1/users/:id', getUser);

// Hardcoded URL somewhere else
res.redirect('/api/v1/users/42');

// And in a template or frontend
fetch('/api/v1/users/' + userId);

// And in an email template
click here: https://example.com/api/v1/users/42
```

Now you decide to change the route to `/api/v2/members/:id`. You have to find every single reference and update it. In a large project, that could be dozens of places across backend code, frontend code, email templates, and documentation.

## The Named Routes Concept

The idea is simple: give each route a name, and use that name to generate URLs. If the path changes, you update it in one place, and all generated URLs update automatically.

Express does not have built-in named routes, but we can implement them. Here are a few approaches.

## Approach 1: Simple Route Registry

Create a central registry of your routes:

```javascript
// src/routes/registry.js
const routes = {
  'users.list':    { method: 'GET',  path: '/api/users' },
  'users.detail':  { method: 'GET',  path: '/api/users/:id' },
  'users.create':  { method: 'POST', path: '/api/users' },
  'users.update':  { method: 'PUT',  path: '/api/users/:id' },
  'users.delete':  { method: 'DELETE', path: '/api/users/:id' },
  'auth.login':    { method: 'POST', path: '/api/auth/login' },
  'auth.register': { method: 'POST', path: '/api/auth/register' },
  'posts.list':    { method: 'GET',  path: '/api/posts' },
  'posts.detail':  { method: 'GET',  path: '/api/posts/:id' },
};

function buildUrl(name, params = {}, query = {}) {
  const route = routes[name];
  if (!route) {
    throw new Error(`Route "${name}" not found`);
  }

  let url = route.path;

  // Replace parameters
  Object.entries(params).forEach(([key, value]) => {
    url = url.replace(`:${key}`, encodeURIComponent(value));
  });

  // Check for unreplaced parameters
  const missingParams = url.match(/:\w+/g);
  if (missingParams) {
    throw new Error(`Missing parameters for route "${name}": ${missingParams.join(', ')}`);
  }

  // Add query string
  const queryString = Object.entries(query)
    .filter(([, v]) => v !== undefined && v !== null)
    .map(([k, v]) => `${encodeURIComponent(k)}=${encodeURIComponent(v)}`)
    .join('&');

  if (queryString) {
    url += `?${queryString}`;
  }

  return url;
}

module.exports = { routes, buildUrl };
```

Using the registry:

```javascript
const { buildUrl } = require('./registry');

// Build a URL with parameters
buildUrl('users.detail', { id: 42 });
// -> '/api/users/42'

// Build a URL with query parameters
buildUrl('users.list', {}, { page: 2, sort: 'name' });
// -> '/api/users?page=2&sort=name'

// Build a URL with both
buildUrl('posts.detail', { id: 5 }, { preview: true });
// -> '/api/posts/5?preview=true'

// Throw an error for missing parameters
buildUrl('users.detail');
// Error: Missing parameters for route "users.detail": :id
```

## Approach 2: Route Registration Helper

Instead of maintaining a separate registry, create a helper that registers routes and records their names at the same time:

```javascript
// src/utils/namedRouter.js
const routeRegistry = {};

function registerRoute(app, method, path, name, ...handlers) {
  // Register with Express
  app[method](path, ...handlers);

  // Store in registry
  routeRegistry[name] = { method, path };
}

function urlFor(name, params = {}, query = {}) {
  const route = routeRegistry[name];
  if (!route) {
    throw new Error(`Route "${name}" not found in registry`);
  }

  let url = route.path;
  Object.entries(params).forEach(([key, value]) => {
    url = url.replace(`:${key}`, encodeURIComponent(value));
  });

  const qs = Object.entries(query)
    .filter(([, v]) => v !== undefined && v !== null)
    .map(([k, v]) => `${encodeURIComponent(k)}=${encodeURIComponent(v)}`)
    .join('&');

  return qs ? `${url}?${qs}` : url;
}

module.exports = { registerRoute, urlFor, routeRegistry };
```

Using the helper:

```javascript
const { registerRoute, urlFor } = require('./utils/namedRouter');

// Register routes with names
registerRoute(app, 'get', '/api/users', 'users.list', (req, res) => {
  const users = getUsers();
  res.json({
    users,
    _links: {
      self: urlFor('users.list', {}, { page: req.query.page }),
      next: urlFor('users.list', {}, { page: (parseInt(req.query.page) || 1) + 1 })
    }
  });
});

registerRoute(app, 'get', '/api/users/:id', 'users.detail', (req, res) => {
  const user = getUser(req.params.id);
  res.json({
    user,
    _links: {
      self: urlFor('users.detail', { id: user.id }),
      posts: urlFor('posts.list', {}, { userId: user.id })
    }
  });
});

// Generate URLs anywhere in your code
const loginUrl = urlFor('auth.login');
const profileUrl = urlFor('users.detail', { id: 42 });
const userListUrl = urlFor('users.list', {}, { page: 3, limit: 25 });
```

## Approach 3: Using a Package

If you prefer an established package, `express-reverse` adds named route support to Express:

```bash
npm install express-reverse
```

```javascript
const express = require('express');
const expressReverse = require('express-reverse');

const app = express();
expressReverse(app);

// Define named routes
app.get('users.list', '/api/users', (req, res) => {
  res.json({ users: [] });
});

app.get('users.detail', '/api/users/:id', (req, res) => {
  res.json({ id: req.params.id });
});

// Generate URLs with app.urlFor()
app.get('/test', (req, res) => {
  const listUrl = app.urlFor('users.list');
  const detailUrl = app.urlFor('users.detail', { id: 42 });

  res.json({ listUrl, detailUrl });
});
```

## When to Use Named Routes

I do not use named routes for every single route. That would be overkill. I use them when:

- **Multiple places reference the same URL.** If a URL appears in more than one file, it should be generated, not hardcoded.
- **URLs are included in external communications.** Email links, webhook callbacks, and shared URLs should all use named routes.
- **The API path might change.** During development, routes change often. Named routes make refactoring painless.
- **Building HATEOAS-style APIs.** If your API includes links in responses, named routes keep those links consistent.

For simple projects with few routes, hardcoded URLs are fine. But once you start seeing the same path in three or more places, it is time to use named routes.

The upfront effort of setting up named routes pays off every time you change a URL and only have to update one file instead of twenty.
