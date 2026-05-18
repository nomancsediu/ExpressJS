# Debugging Express Applications

Bugs happen. Even after writing Express apps for a while, I still spend a significant chunk of my time debugging. The difference between a frustrated developer and a productive one is knowing how to track down problems efficiently. Let me share the tools and techniques I use.

## The Express Debug Module

Express uses the `debug` package internally, and you can tap into its logging. This was a game changer for me because it shows you exactly what Express is doing under the hood.

Run your app with the DEBUG environment variable:

```bash
# See all Express debug output
DEBUG=express:* node app.js

# See only route matching
DEBUG=express:router node app.js

# See only the view system
DEBUG=express:view node app.js

# See your own debug messages
DEBUG=my-app:* node app.js

# Combine multiple namespaces
DEBUG=express:router,my-app:* node app.js

# See everything
DEBUG=* node app.js
```

You can use `debug` in your own code too:

```javascript
const debug = require('debug')('my-app:routes:users');

exports.getAllUsers = async (req, res, next) => {
  debug('Fetching all users');
  try {
    const users = await User.findAll();
    debug('Found %d users', users.length);
    res.json(users);
  } catch (err) {
    debug('Error fetching users: %s', err.message);
    next(err);
  }
};
```

The `%d` and `%s` are format placeholders (number and string). The debug output only shows when the DEBUG environment variable matches, so it does not slow down production.

## Using Morgan for Request Logging

Morgan logs every HTTP request. It tells you the method, URL, status code, and response time. I consider it essential for development.

```bash
npm install morgan
```

```javascript
const morgan = require('morgan');

// Development: colored, concise output
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Production: standard Apache log format
if (process.env.NODE_ENV === 'production') {
  app.use(morgan('combined'));
}
```

The `dev` format output looks like this:

```
GET /api/users 200 12.345 ms - 156
POST /api/users 201 45.678 ms - 89
GET /api/unknown 404 1.234 ms - 24
```

When something goes wrong, I check Morgan's output first. It tells me which endpoints are slow, which are returning errors, and in what order requests are coming in.

## Debugging with VS Code

VS Code has excellent Node.js debugging support. Here is how I set it up.

Create a `.vscode/launch.json` file:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Express App",
      "runtimeExecutable": "node",
      "runtimeArgs": ["--inspect", "app.js"],
      "env": {
        "NODE_ENV": "development",
        "DEBUG": "my-app:*"
      },
      "console": "integratedTerminal",
      "restart": true,
      "outputCapture": "std"
    }
  ]
}
```

Now you can set breakpoints in your code, press F5, and VS Code will pause execution when the breakpoint is hit. You can inspect variables, step through code line by line, and evaluate expressions in the debug console.

For debugging with nodemon, use this configuration instead:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug with Nodemon",
  "runtimeExecutable": "nodemon",
  "runtimeArgs": ["--inspect", "app.js"],
  "env": {
    "NODE_ENV": "development"
  },
  "console": "integratedTerminal",
  "restart": true
}
```

## Using console.log Effectively

Sometimes a quick `console.log` is the fastest way to debug. But I see people make it harder than it needs to be. Here are my tips:

```javascript
// Bad: hard to tell which log is which
console.log(user);

// Good: label your logs
console.log('user:', user);

// Better: use object shorthand for named variables
const userId = 123;
const status = 'active';
console.log({ userId, status });
// Output: { userId: 123, status: 'active' }

// Use console.table for arrays of objects
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];
console.table(users);

// Use console.trace to see where a function was called from
console.trace('Who called this?');

// Measure execution time
console.time('database query');
const result = await db.query('SELECT * FROM users');
console.timeEnd('database query');
```

## Common Express Bugs and How to Find Them

**1. "Cannot read property of undefined" in req.body**

This usually means you forgot to add the JSON body parser:

```javascript
// Add this before your routes
app.use(express.json());
```

**2. Middleware not running**

Middleware runs in the order it is added. If you add your routes before your middleware, the middleware will not run for those routes:

```javascript
// Wrong order
app.get('/api/users', userController.getAll); // Runs without auth
app.use(authMiddleware); // Too late

// Right order
app.use(authMiddleware); // Runs first
app.get('/api/users', userController.getAll); // Auth has already run
```

**3. Response sent twice**

This happens when you send a response and then try to send another one. Look for multiple `res.send()`, `res.json()`, or `res.redirect()` calls in the same handler. Remember that `return res.json()` is not the same as `res.json(); return;`. The `return` stops your function from continuing.

**4. next() not called in middleware**

If your middleware does not call `next()`, the request hangs forever because Express never moves to the next handler:

```javascript
// This middleware blocks everything after it
app.use((req, res, next) => {
  console.log(req.path);
  // Forgot to call next()!
});

// Always call next when you want processing to continue
app.use((req, res, next) => {
  console.log(req.path);
  next();
});
```

**5. Async errors not caught**

If an async route handler throws an error, Express 4 does not catch it automatically. You need to wrap async handlers:

```javascript
// This unhandled rejection crashes the app
app.get('/users', async (req, res) => {
  const users = await User.findAll(); // If this throws, app crashes
  res.json(users);
});

// Wrap with try/catch
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (err) {
    next(err);
  }
});
```

Or use a wrapper function:

```javascript
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));
```

Debugging is a skill that gets better with practice. The more bugs you track down, the faster you get at finding the next one. Keep these tools in your toolkit and use them regularly.
