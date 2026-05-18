# Making Express Apps Fast

Performance was not something I thought about when I started building Express apps. Everything felt fast in development. Then I deployed to production with real users, and suddenly responses were slow, the server was maxing out, and the app felt sluggish. I had to learn how to make Express fast the hard way.

## Why Performance Matters

Slow APIs lose users. Studies show that a 100ms delay in response time can reduce conversion rates significantly. When your Express app takes too long to respond, people leave.

Performance is also about cost. A slow app needs more server resources to handle the same traffic. Optimizing means you can serve more users with less infrastructure.

## Common Performance Bottlenecks

In my experience, these are the usual suspects in Express apps:

- **No compression**: Sending uncompressed responses wastes bandwidth
- **Missing caching**: Repeating the same database queries or computations
- **Slow database queries**: Missing indexes, fetching too much data, N+1 problems
- **Single-threaded bottleneck**: Node runs on one core by default
- **Memory leaks**: Gradually slowing down until the server crashes

## A Simple Example

Here is a basic Express route that has performance problems:

```javascript
app.get('/api/posts', async (req, res) => {
  // No caching, no pagination, fetching everything
  const posts = await Post.find().populate('author').populate('comments');
  res.json(posts);
});
```

With 10,000 posts, this route might take seconds. After applying the techniques in this section:

```javascript
app.get('/api/posts', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = 20;
  const posts = await Post.find()
    .select('title excerpt author')
    .populate('author', 'name')
    .skip((page - 1) * limit)
    .limit(limit)
    .lean();

  res.set('Cache-Control', 'public, max-age=60');
  res.json(posts);
});
```

Same functionality, much faster. In this section, I will cover compression, caching, database optimization, clustering, and more. Each chapter tackles a specific area where I found real improvements in my Express projects.
