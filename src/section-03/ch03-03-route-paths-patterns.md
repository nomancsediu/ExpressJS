# Route Paths and Patterns

Route paths are more flexible than they first appear. You can match exact strings, use patterns, wildcards, and even regular expressions. Let me show you every way to define a route path in Express.

## Exact String Matching

The simplest route path is an exact string:

```javascript
app.get('/about', handler);       // Matches only /about
app.get('/contact', handler);     // Matches only /contact
app.get('/api/users', handler);   // Matches only /api/users
```

This is straightforward. The path must match exactly.

## String Patterns

Express supports special characters in string patterns:

### The Question Mark (?)

The question mark makes the preceding character optional:

```javascript
// 'b' is optional, matches both /ab and /a
app.get('/ab?cd', handler);

// Practical example: singular or plural
app.get('/user?s', handler); // Matches /users or /user
```

### The Plus Sign (+)

The plus sign matches one or more of the preceding character:

```javascript
// Matches /acd, /abcd, /abbcd, /abbbcd, etc.
app.get('/ab+cd', handler);
```

I rarely use this one in practice, but it is there if you need it.

### The Asterisk (*)

The asterisk matches any string in that position:

```javascript
// Matches /abcd, /abXcd, /abANYTHINGcd
app.get('/ab*cd', handler);

// Practical example: catch any path under /files
app.get('/files/*', (req, res) => {
  // req.params[0] contains the wildcard part
  res.send(`File path: ${req.params[0]}`);
});
// GET /files/docs/report.pdf -> "File path: docs/report.pdf"
// GET /files/images/logo.png -> "File path: images/logo.png"
```

### Parentheses for Groups

Parentheses group characters together, and you can combine them with ?, +, and *:

```javascript
// /abe and /abcde both match (the 'cd' group is optional)
app.get('/ab(cd)?e', handler);
```

## Regular Expressions

For complex matching, you can use regular expressions. This is powerful but can be hard to read, so use it sparingly.

```javascript
// Match any path that contains "a" anywhere
app.get(/a/, (req, res) => {
  res.send('Path contains "a"');
});

// Match paths ending with .txt
app.get(/.*\.txt$/, (req, res) => {
  res.send('This is a text file request');
});

// Match only numeric IDs
app.get(/^\/users\/(\d+)$/, (req, res) => {
  // req.params[0] is the captured group
  res.send(`User ID: ${req.params[0]}`);
});
// GET /users/123 -> "User ID: 123"
// GET /users/abc -> 404 Not Found

// Match UUIDs
app.get(/^\/items\/[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/, (req, res) => {
  res.send('Valid UUID format');
});
```

## Named Route Parameters

The most common pattern is named parameters with a colon:

```javascript
app.get('/users/:id', (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});

app.get('/posts/:postId/comments/:commentId', (req, res) => {
  res.send(`Post ${req.params.postId}, Comment ${req.params.commentId}`);
});
```

Named parameters match any value except a forward slash. So `/users/123` matches, but `/users/123/edit` does not. The parameter stops at the next slash.

## Hyphens and Dots in Parameters

You can include hyphens and dots in parameter patterns:

```javascript
// Hyphenated parameter
app.get('/flights/:from-:to', (req, res) => {
  res.send(`Flight from ${req.params.from} to ${req.params.to}`);
});
// GET /flights-LAX-JFK -> from: LAX, to: JFK

// Parameter with a dot
app.get('/files/:name.:ext', (req, res) => {
  res.send(`File: ${req.params.name}.${req.params.ext}`);
});
// GET /files/report.pdf -> name: report, ext: pdf
```

## Case Sensitivity

By default, Express route paths are case-insensitive. `/About`, `/about`, and `/ABOUT` all match the same route. You can change this:

```javascript
// Enable case-sensitive routing globally
app.set('case sensitive routing', true);

// Now /About and /about are different routes
app.get('/About', (req, res) => {
  res.send('Uppercase About');
});

app.get('/about', (req, res) => {
  res.send('Lowercase about');
});
```

I usually leave case sensitivity disabled for web apps because users type URLs in unpredictable ways. But for APIs where consistency matters, enabling it can prevent confusion.

## Strict Routing

By default, Express treats `/about` and `/about/` as the same route. With strict routing, they are different:

```javascript
app.set('strict routing', true);

app.get('/about', (req, res) => {
  res.send('No trailing slash');
});

app.get('/about/', (req, res) => {
  res.send('With trailing slash');
});
```

I usually leave strict routing disabled because it is more forgiving. But some API designs care about trailing slashes, so it is good to know this option exists.

## Practical Examples

Here are some real-world route patterns I use:

```javascript
// Blog post with slug
app.get('/blog/:slug', (req, res) => {
  res.send(`Blog post: ${req.params.slug}`);
});

// API versioning
app.get('/api/v:version/users', (req, res) => {
  res.send(`API version ${req.params.version}`);
});

// File download with format
app.get('/reports/:year/:month.:format', (req, res) => {
  const { year, month, format } = req.params;
  res.send(`Report for ${month}/${year} in ${format} format`);
});
// GET /reports/2024/03.pdf -> year: 2024, month: 03, format: pdf

// Catch-all for single-page app routing
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```

Route patterns are one of those things where simple is usually better. Start with exact strings and named parameters. Reach for wildcards and regular expressions only when you genuinely need them. Your future self will thank you for keeping things readable.
