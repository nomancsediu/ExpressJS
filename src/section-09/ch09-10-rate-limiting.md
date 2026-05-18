# Rate Limiting

If I leave my API unprotected, someone could write a script to hit my login endpoint a thousand times per second. They might guess a password, overwhelm my server, or run up my database bill. Rate limiting prevents this by restricting how many requests a client can make in a given time period.

## What Rate Limiting Does

Rate limiting tracks requests from each client and blocks them after they exceed a threshold. The client gets a 429 (Too Many Requests) response instead of hitting my server.

```
Request 1: 200 OK
Request 2: 200 OK
...
Request 100: 200 OK
Request 101: 429 Too Many Requests
Request 102: 429 Too Many Requests
...wait 15 minutes...
Request 103: 200 OK
```

## express-rate-limit

```bash
npm install express-rate-limit
```

This is the most popular rate limiting library for Express. It is simple and effective.

## Basic Setup

```js
const rateLimit = require('express-rate-limit');

// General API rate limit: 100 requests per 15 minutes
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: {
    status: 'error',
    message: 'Too many requests, please try again later',
  },
});

app.use('/api', apiLimiter);
```

Every client (identified by IP address) gets 100 requests per 15-minute window. After that, they get a 429 response.

## Brute Force Prevention

Login and registration endpoints need stricter limits. I do not want someone trying thousands of password combinations:

```js
// Login rate limit: 5 attempts per 15 minutes
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: {
    status: 'error',
    message: 'Too many login attempts, please try again in 15 minutes',
  },
  standardHeaders: true,
  legacyHeaders: false,
});

router.post('/login', loginLimiter, loginHandler);
router.post('/register', loginLimiter, registerHandler);
```

Five attempts in 15 minutes is generous for a real user but devastating for a brute force attack.

## Different Strategies

### IP-Based (Default)

Limits by client IP address. Good for public APIs.

```js
const ipLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 30,
  keyGenerator: (req) => req.ip,
});
```

### User-Based

Limits by authenticated user ID. Better for logged-in users sharing an IP (like an office network):

```js
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 30,
  keyGenerator: (req) => req.user?.id || req.ip,
});
```

### API Key-Based

Limits by API key for third-party integrations:

```js
const apiKeyLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 1000,
  keyGenerator: (req) => req.headers['x-api-key'] || req.ip,
});
```

## Custom Response Headers

I enable standard headers so clients know their limits:

```js
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true,  // Send RateLimit-* headers
  legacyHeaders: false,   // Disable X-RateLimit-* headers
});
```

Response headers:
```
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1694837400
```

Clients can use these headers to slow down before hitting the limit.

## Graduated Rate Limiting

For really sensitive endpoints, I increase the restriction with each failed attempt:

```js
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true, // Only count failed logins
  handler: (req, res, next, options) => {
    // Could add additional logging or alerting here
    res.status(429).json({
      status: 'error',
      message: 'Too many failed login attempts. Account temporarily locked.',
    });
  },
});
```

The `skipSuccessfulRequests` option means only failed attempts count toward the limit. A user who types their password wrong three times and then gets it right is not penalized.

## A Complete Example

```js
const rateLimit = require('express-rate-limit');

// General: 100 per 15 min
const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });

// Auth: 5 per 15 min
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 5 });

// Write operations: 20 per 15 min
const writeLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 20 });

app.use('/api', apiLimiter);
app.use('/api/auth/login', authLimiter);
app.use('/api/auth/register', authLimiter);
app.use('/api/posts', writeLimiter); // POST, PUT, DELETE
```

Rate limiting is not optional for any public API. It takes five minutes to add and saves you from real problems. Add it from the start, not after your first incident.
