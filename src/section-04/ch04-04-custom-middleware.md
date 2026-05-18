# Custom Middleware

Third-party middleware is great, but sometimes you need something specific to your app. Writing custom middleware is one of the most empowering things I learned in Express. It is simpler than I expected.

## Basic Structure

Every middleware is just a function with three parameters:

```js
function myMiddleware(req, res, next) {
  // do something
  next();
}
```

You can also write it as an arrow function:

```js
const myMiddleware = (req, res, next) => {
  // do something
  next();
};
```

## Logging Middleware

Let me write a simple logger that records the method, URL, and timestamp of every request:

```js
const requestLogger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.url}`);
  next();
};

app.use(requestLogger);
```

Now every request gets logged. This helped me understand what my server was actually receiving.

## Authentication Middleware

A common pattern is checking if a user is authenticated before letting them access certain routes:

```js
const requireAuth = (req, res, next) => {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  // verify token (simplified)
  try {
    req.user = verifyToken(token); // your verification logic
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

app.get('/profile', requireAuth, (req, res) => {
  res.json({ user: req.user });
});
```

Notice the `return` before `res.status(401)`. Without it, the code keeps running and might call `next()` too. I made that mistake and got double responses.

## Request Timing Middleware

Want to know how long each request takes? This middleware measures it:

```js
const requestTimer = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} took ${duration}ms`);
  });

  next();
};

app.use(requestTimer);
```

The `finish` event fires when the response has been sent. This is a neat trick I picked up from reading the Node.js docs.

## Adding Data to the Request

Middleware can attach custom properties to `req` that downstream handlers can use:

```js
const addRequestId = (req, res, next) => {
  req.id = Math.random().toString(36).substring(2, 10);
  next();
};

app.use(addRequestId);
```

This pattern is how authentication middleware works. It verifies the token, then attaches the user to `req.user` so route handlers can access it.

Custom middleware gives you total control over how your app processes requests. Once you start writing your own, you will see opportunities everywhere to clean up and organize your code.
