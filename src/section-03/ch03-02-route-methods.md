# Route Methods

HTTP has several methods, and Express supports all of them. Each method has a purpose, and using the right one makes your API predictable and RESTful. Let me go through each one.

## The Standard Methods

### GET - Retrieve Data

GET is for reading data. It should never modify anything on the server.

```javascript
// Get all users
app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' }
  ]);
});

// Get a single user
app.get('/api/users/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'Alice' });
});
```

### POST - Create Data

POST is for creating new resources.

```javascript
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  const newUser = {
    id: Date.now(),
    name,
    email
  };
  res.status(201).json(newUser);
});
```

Notice the `201` status code. It means "Created." This is more specific than `200` for POST requests.

### PUT - Replace Data

PUT replaces an entire resource with the provided data.

```javascript
app.put('/api/users/:id', (req, res) => {
  const { name, email } = req.body;
  // In a real app, you would update the database
  const updatedUser = {
    id: req.params.id,
    name,
    email
  };
  res.json(updatedUser);
});
```

### PATCH - Partially Update Data

PATCH updates only the fields that are provided, leaving the rest unchanged.

```javascript
app.patch('/api/users/:id', (req, res) => {
  // Only update the fields that were sent
  const updates = req.body;
  const updatedUser = {
    id: req.params.id,
    ...updates
  };
  res.json(updatedUser);
});
```

The difference between PUT and PATCH matters. PUT expects the complete object. PATCH expects only the fields being changed. If you PUT with just `{ name: 'Alice' }`, the email should be removed. If you PATCH with `{ name: 'Alice' }`, only the name changes and the email stays the same.

### DELETE - Remove Data

DELETE removes a resource.

```javascript
app.delete('/api/users/:id', (req, res) => {
  // In a real app, you would delete from the database
  res.json({ message: `User ${req.params.id} deleted` });
});
```

Some APIs return `204 No Content` with no body for successful deletions. Others return the deleted resource or a confirmation message. Both approaches are valid.

```javascript
// Option 1: No content
app.delete('/api/users/:id', (req, res) => {
  res.status(204).end();
});

// Option 2: Confirmation message
app.delete('/api/users/:id', (req, res) => {
  res.json({ message: 'User deleted successfully' });
});
```

## Less Common Methods

### HEAD - Get Headers Only

HEAD is like GET but returns only the headers, no body. It is useful for checking if a resource exists or getting metadata without downloading the full content.

```javascript
app.head('/api/users/:id', (req, res) => {
  // Set headers as if it were a GET request
  res.set('Content-Type', 'application/json');
  res.set('Content-Length', '42');
  res.end(); // No body
});
```

In practice, Express automatically handles HEAD requests for GET routes. If you define a GET route, Express will respond to HEAD requests for the same path by sending the headers without the body.

### OPTIONS - Describe Available Methods

OPTIONS tells the client which HTTP methods are supported for a URL. Browsers send OPTIONS requests automatically during CORS preflight checks.

```javascript
app.options('/api/users', (req, res) => {
  res.set('Allow', 'GET, POST, HEAD, OPTIONS');
  res.end();
});
```

Again, Express and the CORS middleware usually handle OPTIONS for you. But knowing about it helps when debugging CORS issues.

## app.route() - Chain Handlers for One Path

When you have multiple methods on the same path, `app.route()` lets you chain them together. This avoids repeating the path:

```javascript
// Without app.route() - repetitive
app.get('/articles', getArticles);
app.post('/articles', createArticle);
app.put('/articles', updateArticle);
app.delete('/articles', deleteArticle);

// With app.route() - cleaner
app.route('/articles')
  .get((req, res) => {
    res.json({ action: 'List articles' });
  })
  .post((req, res) => {
    res.status(201).json({ action: 'Create article' });
  })
  .put((req, res) => {
    res.json({ action: 'Update article' });
  })
  .delete((req, res) => {
    res.json({ action: 'Delete article' });
  });
```

This becomes even more useful with `express.Router()`, which we will cover later:

```javascript
const router = express.Router();

router.route('/users/:id')
  .get((req, res) => {
    res.json({ action: 'Get user', id: req.params.id });
  })
  .put((req, res) => {
    res.json({ action: 'Update user', id: req.params.id });
  })
  .patch((req, res) => {
    res.json({ action: 'Patch user', id: req.params.id });
  })
  .delete((req, res) => {
    res.json({ action: 'Delete user', id: req.params.id });
  });

app.use('/api', router);
```

## Choosing the Right Method

Here is a quick reference I keep in mind:

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read data | Yes | Yes |
| POST | Create data | No | No |
| PUT | Replace data | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove data | Yes | No |
| HEAD | Read headers only | Yes | Yes |
| OPTIONS | Check supported methods | Yes | Yes |

**Idempotent** means making the same request multiple times produces the same result. GET, PUT, and DELETE are idempotent. POST is not: making the same POST request twice creates two resources.

**Safe** means the method does not modify data on the server. GET and HEAD are safe. Everything else is not.

Using the correct HTTP method is not just about following rules. It makes your API intuitive. When another developer sees a GET request, they know it will not change anything. When they see DELETE, they know what to expect. Consistency builds trust.
