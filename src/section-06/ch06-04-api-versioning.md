# API Versioning

APIs change. You add fields, rename endpoints, and change behavior. But existing clients depend on the current API. Versioning lets you evolve your API without breaking old clients. I learned this the hard way when I renamed a field and broke three mobile apps.

## URL Versioning

The most common approach. Put the version number right in the URL:

```js
const v1Router = express.Router();
const v2Router = express.Router();

// V1: returns simple user object
v1Router.get('/users', (req, res) => {
  res.json([
    { id: 1, name: 'Alice' },
  ]);
});

// V2: returns user with additional fields
v2Router.get('/users', (req, res) => {
  res.json([
    { id: 1, name: 'Alice', email: 'alice@test.com', createdAt: '2024-01-01' },
  ]);
});

app.use('/v1', v1Router);
app.use('/v2', v2Router);
```

Now `/v1/users` and `/v2/users` return different formats. Old clients keep working on v1, new clients use v2.

Pros: Simple, visible, easy to understand. Cons: URLs get longer, you duplicate code.

## Header Versioning

Use a custom header instead of putting the version in the URL:

```js
app.get('/users', (req, res) => {
  const version = req.get('API-Version') || '1';
  const accept = req.get('Accept');

  if (version === '2' || accept === 'application/vnd.myapi.v2+json') {
    res.json([
      { id: 1, name: 'Alice', email: 'alice@test.com' },
    ]);
  } else {
    res.json([
      { id: 1, name: 'Alice' },
    ]);
  }
});
```

Pros: Clean URLs. Cons: Hidden from view, harder to test in a browser.

## Query Parameter Versioning

Add the version as a query parameter:

```js
app.get('/users', (req, res) => {
  const version = req.query.version || '1';

  if (version === '2') {
    res.json([
      { id: 1, name: 'Alice', email: 'alice@test.com' },
    ]);
  } else {
    res.json([
      { id: 1, name: 'Alice' },
    ]);
  }
});
```

Request: `GET /users?version=2`

Pros: Easy to add, easy to test. Cons: Not very RESTful, can be confused with filtering params.

## Middleware Approach

For cleaner code, use middleware to set the version:

```js
app.use((req, res, next) => {
  req.version = req.get('API-Version') || req.query.v || '1';
  next();
});

app.get('/users', (req, res) => {
  if (req.version === '2') {
    return v2GetUsers(req, res);
  }
  v1GetUsers(req, res);
});
```

## Which Should You Use?

I use URL versioning for most projects. It is the most visible and the easiest for clients to understand. Header versioning is great for APIs that want clean URLs. Query versioning is the quickest to add but the least conventional.

The most important thing is to pick one and be consistent. Mixing versioning styles within the same API is confusing for everyone.
