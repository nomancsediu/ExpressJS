# Redis Caching

Some data is expensive to compute and changes infrequently. Every time a user requests it, I hit the database, run the query, and send the response. When hundreds of users request the same data, that is hundreds of identical database queries. Redis solves this by storing results in memory for fast retrieval.

## What Redis Is

Redis is an in-memory key-value store. It is insanely fast because it keeps data in RAM instead of on disk. I use it primarily for caching and session storage.

```bash
# Install Redis client
npm install redis
```

## Basic Setup

```js
const redis = require('redis');

const client = redis.createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
});

client.on('error', (err) => console.error('Redis error:', err));

await client.connect();

module.exports = client;
```

## Caching Responses

I cache the result of expensive database queries:

```js
const getCachedUsers = async (req, res, next) => {
  const cacheKey = 'users:all';

  // Check cache first
  const cached = await client.get(cacheKey);

  if (cached) {
    return res.json(JSON.parse(cached));
  }

  // Not in cache, query database
  const users = await User.find().lean();

  // Store in cache with expiration (1 hour)
  await client.setEx(cacheKey, 3600, JSON.stringify(users));

  res.json(users);
};

app.get('/users', getCachedUsers);
```

The first request hits the database. Every subsequent request within the hour gets the cached response. This can reduce response times from hundreds of milliseconds to single digits.

## Cache Keys

I use a naming convention for my cache keys to keep things organized:

```js
// Specific resource
`user:${userId}`
`post:${postId}`

// Collections with filters
`users:page:${page}:limit:${limit}`
`posts:category:${category}`

// Computed results
`stats:daily-signups:${date}`
`search:${queryHash}`
```

## Cache Invalidation

The hardest part of caching is knowing when to clear it. If I cache user data and the user updates their profile, the cache becomes stale. I need to invalidate it:

```js
// After updating a user, clear the cache
router.patch('/users/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });

  // Invalidate the specific user cache
  await client.del(`user:${req.params.id}`);

  // Invalidate the users list cache
  const keys = await client.keys('users:*');
  if (keys.length) await client.del(keys);

  res.json(user);
});
```

A common pattern is to set a short expiration time so stale data clears itself:

```js
// Cache for 5 minutes, then refresh
await client.setEx(cacheKey, 300, JSON.stringify(data));
```

## Session Storage

Redis is a popular choice for storing session data:

```js
const session = require('express-session');
const RedisStore = require('connect-redis').default;

app.use(session({
  store: new RedisStore({ client }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 86400000 }, // 1 day
}));
```

Sessions in Redis survive server restarts, unlike the default in-memory store. This is essential for production.

## When to Cache

I cache data that is:
- Read frequently but updated rarely
- Expensive to compute or query
- Not sensitive to being slightly stale

I do NOT cache data that:
- Must always be real-time (account balances)
- Contains sensitive information without encryption
- Changes on every request

Redis is a tool, not a default. I only add caching when I have a real performance problem to solve.
