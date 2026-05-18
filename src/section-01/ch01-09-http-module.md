# The HTTP Module

Before Express, there was the raw `http` module. Understanding how it works helps you appreciate what Express does for you. It also helps you debug problems when they arise. Let me build a server step by step using just the `http` module.

## Creating a Basic Server

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello, World!');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

That works. Every time someone visits `http://localhost:3000`, they get "Hello, World!" back. But this server treats every URL the same. Whether you visit `/`, `/about`, or `/api/users`, you get the same response. That is not very useful.

## Adding Manual Routing

Let me add routing by checking `req.url` and `req.method`:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  const { method, url } = req;

  // Home page
  if (method === 'GET' && url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Welcome to the home page');
    return;
  }

  // About page
  if (method === 'GET' && url === '/about') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('About us');
    return;
  }

  // API users list
  if (method === 'GET' && url === '/api/users') {
    const users = [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ];
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(users));
    return;
  }

  // Create a user
  if (method === 'POST' && url === '/api/users') {
    let body = '';
    req.on('data', chunk => {
      body += chunk.toString();
    });
    req.on('end', () => {
      const newUser = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ message: 'User created', user: newUser }));
    });
    return;
  }

  // 404 for everything else
  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('Not Found');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

Already you can see the code getting long and messy. And I have only four routes. Imagine a real app with twenty or thirty routes.

## The Problems With Raw HTTP

Here is what frustrates me about building with just the `http` module:

**1. Parsing request bodies is manual and tedious.**

```javascript
// You have to do this for EVERY POST/PUT/PATCH route
let body = '';
req.on('data', chunk => { body += chunk.toString(); });
req.on('end', () => {
  // Now you can use body
});
```

**2. Route matching is primitive.**

You can match exact strings easily, but what about patterns like `/users/:id`? You have to parse the URL yourself:

```javascript
// Matching /users/:id manually
if (url.startsWith('/users/')) {
  const id = url.split('/')[2]; // Fragile and ugly
  // What if the URL is /users/ or /users/123/extra?
}
```

**3. No middleware system.**

Want to log every request? Add logging code to every route handler. Want to check authentication? Add auth checks to every protected route. There is no way to share logic across routes.

```javascript
// Without middleware, you repeat yourself
if (method === 'GET' && url === '/api/profile') {
  if (!isAuthenticated(req)) {
    res.writeHead(401);
    res.end('Unauthorized');
    return;
  }
  // Handle the route
}

if (method === 'GET' && url === '/api/settings') {
  if (!isAuthenticated(req)) {
    res.writeHead(401);
    res.end('Unauthorized');
    return;
  }
  // Handle the route
}
```

**4. Error handling is messy.**

```javascript
req.on('error', (err) => {
  // Handle request errors
});

res.on('error', (err) => {
  // Handle response errors
});

// Uncaught errors crash the process
process.on('uncaughtException', (err) => {
  console.error('Unhandled error:', err);
  process.exit(1);
});
```

**5. No helpers for common tasks.**

Sending JSON, setting cookies, handling redirects, parsing query strings, serving static files. All of these require you to write boilerplate code from scratch.

## Why Express Exists

Express was created to solve exactly these problems. It provides:

- Clean route definitions with pattern matching
- Automatic request body parsing
- A middleware system for shared logic
- Helper methods for responses
- Error handling that works
- A huge ecosystem of plugins

Here is the same four-route server in Express:

```javascript
const express = require('express');
const app = express();

app.use(express.json()); // Body parsing in one line

app.get('/', (req, res) => res.send('Welcome to the home page'));
app.get('/about', (req, res) => res.send('About us'));

app.get('/api/users', (req, res) => {
  const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];
  res.json(users);
});

app.post('/api/users', (req, res) => {
  res.status(201).json({ message: 'User created', user: req.body });
});

app.use((req, res) => res.status(404).send('Not Found'));

app.listen(3000, () => console.log('Server running on port 3000'));
```

Same functionality, half the code, much more readable. That is why Express exists. It takes the raw power of Node.js `http` and wraps it in a clean, developer-friendly API.

Now that you have seen the hard way, you will appreciate the easy way when we get to Express.
