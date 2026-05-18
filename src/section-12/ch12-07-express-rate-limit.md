# Rate Limiting

Rate limiting controls how many requests a user can make in a given time period. It protects your server from abuse, whether accidental or intentional. Without it, a single user can overwhelm your server with requests.

## Why Rate Limit?

- **Brute force attacks:** Someone tries thousands of passwords on your login endpoint
- **API abuse:** A user scrapes your data by making too many requests
- **DDoS attacks:** Floods of requests try to take your server down
- **Accidental loops:** A buggy client sends requests in an infinite loop

Rate limiting is not just about security. It is about fairness and resource management too.

## Installation

```bash
npm install express-rate-limit
```

## Basic Usage

```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                    // Limit each IP to 100 requests per window
  message: 'Too many requests, please try again later'
});

app.use(limiter);
```

This limits every IP to 100 requests per 15 minutes. When the limit is exceeded, Express returns a 429 status code.

## Per-Route Limits

Different endpoints need different limits. A login endpoint should be stricter than a public API:

```js
const strictLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                      // Only 5 attempts
  message: 'Too many login attempts'
});

const apiLimiter = rateLimit({
  windowMs: 60 * 1000,        // 1 minute
  max: 60,                     // 60 requests per minute
  message: 'API rate limit exceeded'
});

app.post('/login', strictLimiter, authController.login);
app.use('/api/', apiLimiter);
```

## Custom Key Generation

By default, rate limiting uses the client IP. You can customize this:

```js
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 30,
  keyGenerator: (req) => {
    // Rate limit by user ID if authenticated, otherwise by IP
    return req.user ? req.user.id : req.ip;
  }
});
```

This is useful for APIs with authenticated users. You want to limit per user, not per IP, because multiple users might share an IP (like an office network).

## Trust Proxy

If your app runs behind a proxy like Nginx or a load balancer, `req.ip` might always be the proxy IP. You need to tell Express to trust the proxy:

```js
app.set('trust proxy', 1);  // Trust the first proxy

const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  trustProxy: true  // express-rate-limit specific option
});
```

Without this, all users behind the same proxy share one rate limit. That means one person's abuse blocks everyone.

## Custom Response

```js
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  },
  standardHeaders: true,  // Return rate limit info in headers
  legacyHeaders: false    // Disable X-RateLimit-* headers
});
```

With `standardHeaders: true`, responses include `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` headers. Clients can use these to show a countdown or slow down before hitting the limit.

## Sliding Window vs Fixed Window

The default is a fixed window. This means limits reset all at once when the window expires. A sliding window is smoother:

```js
const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60,
  skipFailedRequests: true,  // Don't count failed requests (4xx/5xx)
  skipSuccessfulRequests: false
});
```

I add rate limiting early in every project. It is simple to set up and prevents headaches later. Start generous and tighten as you learn your traffic patterns.
