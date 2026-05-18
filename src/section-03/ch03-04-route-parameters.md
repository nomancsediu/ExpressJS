# Route Parameters

Route parameters let you capture values from the URL. They are how you tell Express, "This part of the URL is a variable." I use them in almost every route I write, so understanding them well is important.

## The Basics

Route parameters are defined with a colon in the path and accessed through `req.params`:

```javascript
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ userId });
});

// GET /users/42       -> { "userId": "42" }
// GET /users/alice    -> { "userId": "alice" }
```

Important detail: route parameters are always strings. Even if the URL is `/users/42`, `req.params.id` is the string `"42"`, not the number `42`. If you need a number, you have to parse it yourself:

```javascript
app.get('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id, 10);
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'ID must be a number' });
  }
  res.json({ userId });
});
```

## Multiple Parameters

You can have multiple parameters in a single route:

```javascript
app.get('/posts/:postId/comments/:commentId', (req, res) => {
  const { postId, commentId } = req.params;
  res.json({ postId, commentId });
});

// GET /posts/5/comments/12 -> { "postId": "5", "commentId": "12" }
```

Another common pattern:

```javascript
app.get('/organizations/:orgId/projects/:projectId', (req, res) => {
  const { orgId, projectId } = req.params;
  res.json({ orgId, projectId });
});
```

## Hyphenated and Dotted Parameters

Express can split parameters at hyphens and dots:

```javascript
// Hyphen-separated
app.get('/flights/:from-:to', (req, res) => {
  res.json({ from: req.params.from, to: req.params.to });
});
// GET /flights-SFO-LAX -> { "from": "SFO", "to": "LAX" }

// Dot-separated
app.get('/files/:filename.:extension', (req, res) => {
  res.json({ filename: req.params.filename, extension: req.params.extension });
});
// GET /files/report.pdf -> { "filename": "report", "extension": "pdf" }
```

## Wildcard Parameters

When you use an asterisk in a route, the matched portion goes into `req.params` as a numbered key:

```javascript
app.get('/files/*', (req, res) => {
  const filePath = req.params[0]; // The wildcard match
  res.json({ path: filePath });
});

// GET /files/docs/report.pdf -> { "path": "docs/report.pdf" }
// GET /files/images/2024/vacation/beach.jpg -> { "path": "images/2024/vacation/beach.jpg" }
```

## Parameter Middleware

Express has a special method called `app.param()` that runs middleware whenever a specific parameter appears in any route. This is useful for loading resources automatically:

```javascript
// Load user whenever :userId appears in a route
app.param('userId', (req, res, next, id) => {
  console.log(`Loading user with ID: ${id}`);

  // Simulate database lookup
  const user = { id: id, name: 'User ' + id };

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  req.user = user; // Attach to the request object
  next();
});

// Now :userId routes automatically have req.user
app.get('/users/:userId', (req, res) => {
  res.json(req.user); // Already loaded by param middleware
});

app.put('/users/:userId', (req, res) => {
  // req.user is available here too
  res.json({ ...req.user, ...req.body });
});

app.delete('/users/:userId', (req, res) => {
  res.json({ message: `Deleted user ${req.user.id}` });
});
```

This pattern keeps your route handlers clean. Instead of doing a database lookup in every handler, you do it once in the param middleware.

## Parameter Validation

You should always validate route parameters. Never trust user input, even from URLs. Here are some validation patterns I use:

```javascript
// Validate that ID is numeric
app.get('/users/:id', (req, res, next) => {
  const id = parseInt(req.params.id, 10);
  if (isNaN(id) || id <= 0) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  req.userId = id;
  next();
}, (req, res) => {
  res.json({ userId: req.userId });
});

// Validate UUID format
app.get('/items/:id', (req, res, next) => {
  const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
  if (!uuidRegex.test(req.params.id)) {
    return res.status(400).json({ error: 'Invalid item ID format' });
  }
  next();
}, (req, res) => {
  res.json({ itemId: req.params.id });
});
```

For more complex validation, I use a library like `express-validator`:

```bash
npm install express-validator
```

```javascript
const { param } = require('express-validator');

app.get('/users/:id',
  param('id').isInt({ min: 1 }).withMessage('ID must be a positive integer'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ userId: req.params.id });
  }
);
```

## Sanitizing Parameters

Validation checks that input is valid. Sanitization cleans input to make it safe. Common sanitization tasks include trimming whitespace and escaping HTML:

```javascript
app.get('/search/:term', (req, res) => {
  // Sanitize the search term
  const term = req.params.term
    .trim()
    .toLowerCase()
    .replace(/[<>]/g, ''); // Remove angle brackets

  res.json({ search: term });
});
```

Route parameters seem simple on the surface, but handling them correctly is what separates a fragile API from a robust one. Always validate, always sanitize, and always remember they are strings until you convert them.
