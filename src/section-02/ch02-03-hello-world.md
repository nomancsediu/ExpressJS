# Hello World, Line by Line

Every programming journey starts with Hello World. But instead of just showing you the code and moving on, I want to break down every single line. Understanding what each piece does will make everything else in Express make more sense.

## The Complete Hello World

```javascript
const express = require('express');

const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello, World!');
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

Save this as `app.js` and run it with `node app.js`. Visit `http://localhost:3000` in your browser and you will see "Hello, World!".

Now let me take it apart piece by piece.

## Line 1: Importing Express

```javascript
const express = require('express');
```

This line loads the Express module. The `require` function is how CommonJS imports modules. It reads the `express` package from `node_modules` and returns a function.

That function is the main entry point for creating an Express application. It is not the app itself yet. It is a factory function that creates the app.

## Line 3: Creating the App

```javascript
const app = express();
```

Calling `express()` creates a new Express application. This `app` object is the star of the show. It has methods for:

- Routing (`app.get`, `app.post`, `app.put`, `app.delete`)
- Middleware (`app.use`)
- Configuration (`app.set`, `app.get` for settings)
- Starting the server (`app.listen`)

The `app` object is also a function itself, which is why we can pass it to Node's `http.createServer()` if we want. But for most cases, we use `app.listen()` instead.

## Line 4: Setting the Port

```javascript
const PORT = 3000;
```

This is the port number your server listens on. In a real project, I would read this from an environment variable:

```javascript
const PORT = process.env.PORT || 3000;
```

This way, the port can be overridden in production. Hosting platforms like Heroku set the `PORT` environment variable automatically.

## Lines 6-8: Defining a Route

```javascript
app.get('/', (req, res) => {
  res.send('Hello, World!');
});
```

This is where the magic happens. Let me break it down further:

- **`app.get`** - Registers a handler for HTTP GET requests. There is also `app.post`, `app.put`, `app.delete`, and more.
- **`'/'`** - The URL path to match. The root path in this case.
- **`(req, res) => {}`** - The handler function that runs when a matching request comes in.
- **`req`** - The request object. It contains everything about the incoming request: URL, headers, query parameters, body, and more.
- **`res`** - The response object. It has methods for sending data back to the client.
- **`res.send()`** - Sends a response. It automatically sets the Content-Type header based on what you pass in. A string becomes text/html, an object becomes application/json.

## Lines 10-12: Starting the Server

```javascript
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

`app.listen` does two things:

1. It creates an HTTP server using Node's `http.createServer()`, passing the Express `app` as the request handler.
2. It starts listening on the specified port.

The callback function runs once the server is ready to accept connections. This is where I log a message so I know the server started successfully.

Under the hood, `app.listen` is basically doing this:

```javascript
const http = require('http');
const server = http.createServer(app);
server.listen(PORT, callback);
```

## A More Complete Hello World

Let me add a few more features to make this more realistic:

```javascript
const express = require('express');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware to parse JSON bodies
app.use(express.json());

// Home route
app.get('/', (req, res) => {
  res.json({ message: 'Hello, World!', timestamp: new Date().toISOString() });
});

// Health check route
app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});

// Echo route - sends back whatever you POST
app.post('/echo', (req, res) => {
  res.json({ youSent: req.body });
});

// 404 handler for unmatched routes
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

Now you can test different routes:

```bash
# GET the home page
curl http://localhost:3000/

# GET the health check
curl http://localhost:3000/health

# POST to the echo endpoint
curl -X POST http://localhost:3000/echo \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice"}'

# GET a non-existent route
curl http://localhost:3000/nonexistent
```

## What I Want You to Notice

The most important pattern in Express is this: **request comes in, flows through middleware, hits a route handler, and a response goes out.** Every Express app follows this pattern. The Hello World is just the simplest version of it.

As your app grows, you add more middleware, more routes, and more complexity. But the fundamental flow never changes. Keep that in mind as we move forward.
