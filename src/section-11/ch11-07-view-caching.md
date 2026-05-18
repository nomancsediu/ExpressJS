# View Caching

Every time Express renders a template, the engine reads the file from disk, parses it, and generates HTML. For a busy server, this is wasted work if the template has not changed. View caching solves this by keeping the compiled template in memory.

## How View Caching Works

When caching is enabled, Express compiles each template once and stores the result. On subsequent requests, it skips the file read and parsing step. The cached compiled template runs directly with new data.

## Enabling and Disabling

View caching is controlled by the `view cache` setting:

```js
// Enable caching (automatic in production)
app.set('view cache', true);

// Disable caching (automatic in development)
app.set('view cache', false);
```

In practice, you rarely set this manually. Express enables view caching automatically when `NODE_ENV` is set to `production`:

```bash
# Caching is ON
NODE_ENV=production node server.js

# Caching is OFF
node server.js
```

When caching is off, you can edit templates and see changes immediately by refreshing the browser. When caching is on, you need to restart the server to see template changes.

## Performance Impact

The difference can be significant. Here is a simple benchmark I ran:

```js
app.get('/bench', (req, res) => {
  res.render('heavy-template', {
    items: Array.from({ length: 1000 }, (_, i) => ({
      name: `Item ${i}`,
      price: (Math.random() * 100).toFixed(2)
    }))
  });
});
```

Without caching, rendering took about 15ms per request on my machine. With caching, it dropped to about 3ms. For a template with complex logic and many partials, the savings are even bigger.

## When Caching Matters

Caching helps most when:

- Your app serves many requests per second
- Your templates are complex with many includes
- Your server has slow disk I/O (common on cloud instances)

It matters less when:

- Your app has low traffic
- Templates are simple
- You are in development mode

## Custom Cache Control

You can manually clear the cache if needed, like after a content update:

```js
// Clear the entire view cache
app.cache = {};

// In development, some engines let you watch for file changes
if (app.get('env') === 'development') {
  app.set('view cache', false);

  // Some setups use nodemon to restart the server on file changes
  // This also clears the cache since the process restarts
}
```

## A Common Mistake

I once spent an hour debugging a template that would not update. I kept editing the file and refreshing, but the old version kept showing. Turns out I had accidentally set `NODE_ENV=production` in my `.env` file. View caching was on and serving the old compiled template. Always check your environment when templates seem stuck.

## Best Practice

Let Express handle caching automatically. Do not set `view cache` manually. Develop with `NODE_ENV` unset or set to `development`. Deploy with `NODE_ENV=production`. Express does the right thing in both cases.
