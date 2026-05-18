# Query Parameters

Query parameters are the key-value pairs that appear after the question mark in a URL. They are how you send optional data to the server. Unlike route parameters, which are part of the URL path, query parameters are flexible and optional. Let me show you how to work with them.

## The Basics

In a URL like `/api/users?page=2&sort=name`, the query string is `page=2&sort=name`. Express parses this automatically and makes it available as `req.query`:

```javascript
app.get('/api/users', (req, res) => {
  console.log(req.query);
  // { page: '2', sort: 'name' }

  const page = req.query.page;
  const sort = req.query.sort;

  res.json({ page, sort });
});

// GET /api/users?page=2&sort=name
// Response: { "page": "2", "sort": "name" }
```

Just like route parameters, query values are always strings. You need to convert them yourself:

```javascript
app.get('/api/users', (req, res) => {
  const page = parseInt(req.query.page, 10) || 1;
  const limit = parseInt(req.query.limit, 10) || 10;

  res.json({ page, limit });
});
```

## Filtering with Query Parameters

Query parameters are perfect for filtering data:

```javascript
app.get('/api/products', (req, res) => {
  // Start with all products
  let products = [
    { id: 1, name: 'Laptop', category: 'electronics', price: 999 },
    { id: 2, name: 'Shirt', category: 'clothing', price: 29 },
    { id: 3, name: 'Phone', category: 'electronics', price: 699 },
    { id: 4, name: 'Pants', category: 'clothing', price: 49 },
    { id: 5, name: 'Tablet', category: 'electronics', price: 499 }
  ];

  // Filter by category
  if (req.query.category) {
    products = products.filter(p => p.category === req.query.category);
  }

  // Filter by max price
  if (req.query.maxPrice) {
    const max = parseInt(req.query.maxPrice, 10);
    products = products.filter(p => p.price <= max);
  }

  res.json(products);
});

// GET /api/products?category=electronics
// GET /api/products?maxPrice=500
// GET /api/products?category=clothing&maxPrice=40
```

## Pagination

Pagination is one of the most common uses of query parameters. Here is the pattern I use:

```javascript
app.get('/api/users', (req, res) => {
  // Parse pagination parameters with defaults
  const page = Math.max(1, parseInt(req.query.page, 10) || 1);
  const limit = Math.min(100, Math.max(1, parseInt(req.query.limit, 10) || 10));
  const offset = (page - 1) * limit;

  // Simulate a database query
  const allUsers = Array.from({ length: 95 }, (_, i) => ({
    id: i + 1,
    name: `User ${i + 1}`
  }));

  const users = allUsers.slice(offset, offset + limit);
  const total = allUsers.length;
  const totalPages = Math.ceil(total / limit);

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    }
  });
});

// GET /api/users?page=3&limit=10
```

Notice I cap the limit at 100. Without this, someone could request `?limit=1000000` and overload your server.

## Search

Search endpoints combine query parameters with database lookups:

```javascript
app.get('/api/search', (req, res) => {
  const { q, category, sort } = req.query;

  if (!q) {
    return res.status(400).json({ error: 'Search query (q) is required' });
  }

  // Build your search logic
  const results = searchDatabase({
    query: q,
    category: category || null,
    sort: sort || 'relevance'
  });

  res.json({
    query: q,
    count: results.length,
    results
  });
});

// GET /api/search?q=express&category=tutorials&sort=newest
```

## Sorting

```javascript
app.get('/api/articles', (req, res) => {
  const { sortBy, order } = req.query;

  let articles = [
    { id: 1, title: 'Express Basics', createdAt: '2024-01-15' },
    { id: 2, title: 'Advanced Routing', createdAt: '2024-02-20' },
    { id: 3, title: 'Middleware Guide', createdAt: '2024-01-10' }
  ];

  const validSortFields = ['title', 'createdAt'];
  const field = validSortFields.includes(sortBy) ? sortBy : 'createdAt';
  const direction = order === 'asc' ? 1 : -1;

  articles.sort((a, b) => {
    if (a[field] < b[field]) return -1 * direction;
    if (a[field] > b[field]) return 1 * direction;
    return 0;
  });

  res.json(articles);
});

// GET /api/articles?sortBy=title&order=asc
// GET /api/articles?sortBy=createdAt&order=desc
```

## Query Parameters vs Route Parameters

One question I had when starting was: when should I use route parameters and when should I use query parameters? Here is the guideline I follow:

**Use route parameters for required values that identify a resource:**

```javascript
// Good: user ID identifies a specific user
app.get('/users/:id', handler);

// Good: both IDs identify specific resources
app.get('/posts/:postId/comments/:commentId', handler);
```

**Use query parameters for optional values that filter, sort, or modify the response:**

```javascript
// Good: filtering and pagination are optional
app.get('/users?role=admin&page=2', handler);

// Good: search terms are optional modifiers
app.get('/search?q=express', handler);
```

**The rule of thumb:** If the value is required to identify what the user wants, make it a route parameter. If the value is optional and modifies how the results are presented, make it a query parameter.

| Feature | Route Params | Query Params |
|---------|-------------|-------------|
| Position | Part of the URL path | After the `?` |
| Required | Usually yes | Usually no |
| Purpose | Identify resources | Filter, sort, paginate |
| Example | `/users/42` | `/users?role=admin` |
| Access | `req.params` | `req.query` |
| Type | Always string | Always string |

## Handling Arrays in Query Strings

Sometimes you need to pass multiple values for the same key. Express handles this:

```javascript
// GET /api/users?role=admin&role=editor
// req.query.role -> ['admin', 'editor']

// GET /api/users?tags=javascript&tags=node
// req.query.tags -> ['javascript', 'node']

app.get('/api/users', (req, res) => {
  const roles = Array.isArray(req.query.role)
    ? req.query.role
    : req.query.role
      ? [req.query.role]
      : [];

  res.json({ roles });
});
```

I always wrap single values in an array to handle both cases consistently. A single `?role=admin` gives you a string, but `?role=admin&role=editor` gives you an array. This inconsistency catches a lot of beginners off guard.

Query parameters are the workhorse of any API. Master them and your APIs will be flexible, user-friendly, and easy to use.
