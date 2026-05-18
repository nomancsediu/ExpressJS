# Production Best Practices

Running Express in production is not the same as running it on your laptop. There are settings you need to change, security measures you need to add, and patterns you need to follow. Let me walk through the ones that matter most.

## Set NODE_ENV to Production

This is the single most important thing. Express and many npm packages check `NODE_ENV` and change their behavior accordingly.

```js
// In production, Express enables response caching and disables verbose error messages
NODE_ENV=production
```

When `NODE_ENV` is `production`, Express caches view templates and CSS files, and generates shorter error messages. Some packages skip expensive checks. Performance improves significantly.

## Proper Error Handling

In development, seeing a full stack trace is helpful. In production, it is a security risk. Never expose internal errors to users.

```js
// src/middleware/errorHandler.js
module.exports = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';

  // Log the full error internally
  console.error(err);

  // Send a safe response to the client
  res.status(statusCode).json({
    success: false,
    message: process.env.NODE_ENV === 'production'
      ? (statusCode === 500 ? 'Something went wrong' : message)
      : message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
};
```

In production, 500 errors get a generic message. No stack traces leak to the outside world.

## Security Middleware

Use Helmet to set security headers automatically:

```js
const helmet = require('helmet');
app.use(helmet());
```

Helmet sets headers like `Content-Security-Policy`, `X-Frame-Options`, and `X-Content-Type-Options`. These protect against common attacks like clickjacking and MIME type sniffing.

Configure CORS properly:

```js
const cors = require('cors');
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(','),
  credentials: true,
}));
```

Never use `origin: '*'` in production. Whitelist your actual frontend domains.

## Rate Limiting

Prevent abuse by limiting how many requests a client can make:

```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later',
});

app.use('/api/', limiter);
```

Apply stricter limits to auth routes:

```js
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
});

app.use('/api/auth/login', authLimiter);
```

## Input Sanitization

Always validate and sanitize user input. Never trust what the client sends:

```js
// Use Joi or express-validator
const Joi = require('joi');

const registerSchema = Joi.object({
  name: Joi.string().trim().min(2).max(50).required(),
  email: Joi.string().email().lowercase().required(),
  password: Joi.string().min(6).required(),
});

// Middleware
const validate = (schema) => (req, res, next) => {
  const { error } = schema.validate(req.body);
  if (error) throw new ApiError(400, error.details[0].message);
  next();
};
```

## Use Compression

Reduce response size with compression middleware:

```js
const compression = require('compression');
app.use(compression());
```

This can cut response sizes by 70% or more. Less data over the wire means faster responses.

These practices are not optional. They are the difference between an app that survives in production and one that gets hacked or falls over under load.
