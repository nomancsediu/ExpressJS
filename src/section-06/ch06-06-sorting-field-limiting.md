# Sorting and Field Limiting

Pagination and filtering control *which* records you return. Sorting controls the *order*, and field limiting controls *which fields* each record includes. Together, they give clients full control over the response shape.

## Sorting

Let clients specify the sort order with a query parameter:

```js
app.get('/users', (req, res) => {
  let results = [...users];

  // Sort by query parameter
  // GET /users?sort=name
  // GET /users?sort=-name  (descending)
  // GET /users?sort=-createdAt,name  (multiple fields)
  if (req.query.sort) {
    const sortFields = req.query.sort.split(',');

    results.sort((a, b) => {
      for (const field of sortFields) {
        const descending = field.startsWith('-');
        const key = descending ? field.substring(1) : field;
        const direction = descending ? -1 : 1;

        if (a[key] < b[key]) return -1 * direction;
        if (a[key] > b[key]) return 1 * direction;
      }
      return 0;
    });
  } else {
    // Default sort
    results.sort((a, b) => a.id - b.id);
  }

  res.json({ data: results });
});
```

The convention I like: prefix with `-` for descending. So `sort=name` is ascending, `sort=-name` is descending. Multiple fields are comma-separated: `sort=-age,name`.

## Sorting with MongoDB

If you are using MongoDB, sorting is even easier:

```js
app.get('/users', async (req, res) => {
  const sortStr = req.query.sort || 'createdAt';
  const sortBy = sortStr.startsWith('-')
    ? { [sortStr.substring(1)]: -1 }
    : { [sortStr]: 1 };

  const users = await db.User.find().sort(sortBy);
  res.json({ data: users });
});
```

## Field Limiting (Sparse Fieldsets)

Sometimes clients only need a few fields. Returning everything wastes bandwidth, especially with large records:

```js
app.get('/users', (req, res) => {
  let results = [...users];

  // GET /users?fields=id,name
  // GET /users?fields=id,name,email
  if (req.query.fields) {
    const requestedFields = req.query.fields.split(',');
    const allowedFields = ['id', 'name', 'email', 'role', 'createdAt'];
    const safeFields = requestedFields.filter(f => allowedFields.includes(f));

    if (safeFields.length > 0) {
      results = results.map(user => {
        const filtered = {};
        safeFields.forEach(field => {
          filtered[field] = user[field];
        });
        return filtered;
      });
    }
  }

  res.json({ data: results });
});
```

Always validate the requested fields against an allowlist. Never let clients request arbitrary fields because some fields might be sensitive.

## Combining Everything

Pagination, filtering, sorting, and field limiting work together:

```js
app.get('/users', parseFilters, (req, res) => {
  let results = [...users];

  // 1. Filter
  if (req.filters.role) {
    results = results.filter(u => u.role === req.filters.role);
  }

  // 2. Sort
  if (req.query.sort) {
    results = applySorting(results, req.query.sort);
  }

  // 3. Paginate
  const page = Math.max(1, parseInt(req.query.page, 10) || 1);
  const limit = Math.min(100, Math.max(1, parseInt(req.query.limit, 10) || 10));
  const total = results.length;
  results = results.slice((page - 1) * limit, page * limit);

  // 4. Limit fields
  if (req.query.fields) {
    results = limitFields(results, req.query.fields);
  }

  res.json({
    data: results,
    pagination: { page, limit, total },
  });
});
```

The order matters: filter first to reduce the dataset, then sort, then paginate, then limit fields. This keeps your queries efficient.

## Request Example

```
GET /users?role=admin&sort=-createdAt&fields=id,name&page=2&limit=10
```

This reads as: "Give me admin users, sorted by newest first, only their ID and name, page 2 with 10 per page." Clear and powerful.
