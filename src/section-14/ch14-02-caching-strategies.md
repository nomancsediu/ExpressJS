# Caching Strategies

Caching is one of the most effective ways to improve performance. Instead of doing the same work over and over, you store the result and reuse it. I went from 200ms responses to 5ms just by adding the right caching.

## HTTP Caching

The browser can cache responses based on HTTP headers. This means the browser does not even need to ask your server again:

```javascript
// Cache for 5 minutes
app.get('/api/articles', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300');
  res.json(articles);
});

// Prevent caching for sensitive data
app.get('/api/profile', (req, res) => {
  res.set('Cache-Control', 'no-store');
  res.json(userData);
});
```

Common `Cache-Control` values:

- `public, max-age=300`: Cache for 5 minutes, share across users
- `private, max-age=60`: Cache for 1 minute, only for this user
- `no-cache`: Always revalidate before using cache
- `no-store`: Never cache

## ETag

Express enables ETag by default. An ETag is a fingerprint of the response. When the browser asks for the same URL again, it sends the ETag. If nothing changed, the server responds with 304 Not Modified and no body:

```javascript
// Express enables ETag by default
// You can customize it:
app.set('etag', 'strong'); // default, uses strong comparison
// or
app.set('etag', 'weak');   // semantically equivalent
```

```javascript
// Disable if not needed
app.set('etag', false);
```

ETags save bandwidth when content has not changed.

## Server-Side Caching

HTTP caching helps the browser, but what about repeated requests from different users? I use in-memory caching for that:

```javascript
const cache = new Map();

app.get('/api/popular-posts', async (req, res) => {
  const cacheKey = 'popular-posts';
  const cached = cache.get(cacheKey);

  if (cached && Date.now() - cached.timestamp < 60000) {
    return res.json(cached.data);
  }

  const posts = await Post.find({ popular: true }).sort({ views: -1 }).limit(10);
  cache.set(cacheKey, { data: posts, timestamp: Date.now() });

  res.json(posts);
});
```

## Redis for Distributed Caching

In-memory cache does not share across multiple server instances. Redis solves that:

```bash
npm install redis
```

```javascript
const redis = require('redis');
const client = redis.createClient({ url: 'redis://localhost:6379' });
await client.connect();

app.get('/api/popular-posts', async (req, res) => {
  const cached = await client.get('popular-posts');

  if (cached) {
    return res.json(JSON.parse(cached));
  }

  const posts = await Post.find({ popular: true }).sort({ views: -1 }).limit(10);
  await client.setEx('popular-posts', 60, JSON.stringify(posts));

  res.json(posts);
});
```

Redis works across processes and servers. It is my default choice for any production app running more than one instance.
