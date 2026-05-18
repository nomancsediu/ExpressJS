# CRUD Operations

CRUD stands for Create, Read, Update, and Delete. These four operations cover most of what an API does. Let me implement all of them in Express for a simple users resource.

## The Full Setup

```js
const express = require('express');
const app = express();

app.use(express.json());

// In-memory data store (use a database in real projects)
let users = [
  { id: 1, name: 'Alice', email: 'alice@test.com' },
  { id: 2, name: 'Bob', email: 'bob@test.com' },
];
let nextId = 3;
```

## Create (POST)

```js
app.post('/users', (req, res) => {
  const { name, email } = req.body;

  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }

  const user = { id: nextId++, name, email };
  users.push(user);

  res.status(201).json(user);
});
```

Always return 201 for created resources. Include the created object in the response so the client gets the generated ID.

## Read (GET)

Get all users:

```js
app.get('/users', (req, res) => {
  res.json(users);
});
```

Get a single user:

```js
app.get('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const user = users.find(u => u.id === id);

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json(user);
});
```

Always handle the not-found case. Returning 200 with `null` is confusing.

## Update (PUT and PATCH)

PUT replaces the entire resource. PATCH updates only the provided fields:

```js
// PUT: full replacement
app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const index = users.findIndex(u => u.id === id);

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const { name, email } = req.body;
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }

  users[index] = { id, name, email };
  res.json(users[index]);
});

// PATCH: partial update
app.patch('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const user = users.find(u => u.id === id);

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  if (req.body.name) user.name = req.body.name;
  if (req.body.email) user.email = req.body.email;

  res.json(user);
});
```

## Delete (DELETE)

```js
app.delete('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const index = users.findIndex(u => u.id === id);

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const deleted = users.splice(index, 1);
  res.json(deleted[0]);
});
```

Some APIs return 204 with no body for deletes. I prefer returning the deleted resource so the client knows what was removed.

## Summary Table

| Operation | Method | Path | Success Code |
|---|---|---|---|
| Create | POST | /users | 201 |
| Read all | GET | /users | 200 |
| Read one | GET | /users/:id | 200 |
| Replace | PUT | /users/:id | 200 |
| Partial update | PATCH | /users/:id | 200 |
| Delete | DELETE | /users/:id | 200 or 204 |

This pattern repeats for every resource. Users, posts, comments, orders, they all follow the same structure. Once you know it, you can build CRUD for anything.
