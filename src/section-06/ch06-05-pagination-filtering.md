# Pagination and Filtering

When your database has thousands of records, returning all of them in one response is a bad idea. It is slow, wastes bandwidth, and overwhelms the client. Pagination solves this by returning a subset of results at a time. Filtering narrows down what you return.

## Page-Based Pagination

The simplest approach. The client provides a page number and a limit:

```js
app.get('/users', (req, res) => {
  const page = parseInt(req.query.page, 10) || 1;
  const limit = parseInt(req.query.limit, 10) || 10;
  const offset = (page - 1) * limit;

  const results = users.slice(offset, offset + limit);
  const total = users.length;
  const totalPages = Math.ceil(total / limit);

  res.json({
    data: results,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1,
    },
  });
});
```

Request: `GET /users?page=2&limit=10`

Response:

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 10,
    "total": 45,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": true
  }
}
```

## Cursor-Based Pagination

Page-based pagination has a problem: if data changes between requests, you might skip or duplicate items. Cursor-based pagination avoids this by using a pointer:

```js
app.get('/posts', async (req, res) => {
  const cursor = req.query.cursor; // the ID of the last item seen
  const limit = parseInt(req.query.limit, 10) || 10;

  let query = db.posts.find().sort({ id: 1 }).limit(limit + 1);

  if (cursor) {
    query = query.where('id').gt(cursor);
  }

  const results = await query;
  const hasMore = results.length > limit;
  const data = hasMore ? results.slice(0, limit) : results;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  res.json({
    data,
    pagination: {
      nextCursor,
      hasMore,
    },
  });
});
```

Request: `GET /posts?cursor=42&limit=10`

Cursor-based pagination is better for large, frequently changing datasets. Social media feeds use it because new posts arrive constantly.

## Filtering

Let clients filter results by specific fields:

```js
app.get('/users', (req, res) => {
  let results = [...users];

  // Filter by name
  if (req.query.name) {
    results = results.filter(u =>
      u.name.toLowerCase().includes(req.query.name.toLowerCase())
    );
  }

  // Filter by email domain
  if (req.query.domain) {
    results = results.filter(u =>
      u.email.endsWith(`@${req.query.domain}`)
    );
  }

  // Filter by role
  if (req.query.role) {
    results = results.filter(u => u.role === req.query.role);
  }

  res.json({ data: results, count: results.length });
});
```

Request: `GET /users?role=admin&domain=test.com`

## Filter Middleware

For repeated filtering logic, I extract it into middleware:

```js
const parseFilters = (req, res, next) => {
  req.filters = {};

  const allowedFilters = ['role', 'status', 'domain'];
  allowedFilters.forEach(key => {
    if (req.query[key]) {
      req.filters[key] = req.query[key];
    }
  });

  next();
};

app.get('/users', parseFilters, (req, res) => {
  const results = applyFilters(users, req.filters);
  res.json({ data: results });
});
```

This keeps route handlers clean and makes filters reusable across endpoints.

## Validation

Always validate pagination parameters:

```js
const page = Math.max(1, parseInt(req.query.page, 10) || 1);
const limit = Math.min(100, Math.max(1, parseInt(req.query.limit, 10) || 10));
```

This prevents negative pages, zero limits, and someone requesting a million records at once.
