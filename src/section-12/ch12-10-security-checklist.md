# Security Checklist

This is the checklist I run through before deploying any Express app. Some items are quick, some take more effort. All of them matter.

## Dependencies

- [ ] Run `npm audit` and fix all vulnerabilities
- [ ] Update all packages to latest stable versions
- [ ] Remove unused dependencies
- [ ] Check for known vulnerabilities in your lockfile

```bash
npm audit
npm outdated
npm update
```

Set up automated dependency scanning. GitHub Dependabot and Snyk both provide free plans that alert you when vulnerabilities are found.

## Headers

- [ ] Use Helmet.js for security headers
- [ ] Set Content-Security-Policy for your app
- [ ] Enable HSTS after confirming HTTPS works
- [ ] Set X-Content-Type-Options to nosniff
- [ ] Configure Referrer-Policy

```js
app.use(helmet());
```

## Authentication and Sessions

- [ ] Use HTTPS everywhere (redirect HTTP to HTTPS)
- [ ] Set cookies with `httpOnly`, `secure`, and `sameSite`
- [ ] Use strong, random session secrets
- [ ] Implement CSRF protection for state-changing requests
- [ ] Hash passwords with bcrypt (never store plain text)

```js
app.use(session({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 24 * 60 * 60 * 1000
  },
  secret: process.env.SESSION_SECRET
}));
```

## Input Handling

- [ ] Validate all user input on the server
- [ ] Sanitize input to prevent XSS
- [ ] Use parameterized queries for all database operations
- [ ] Limit file upload sizes and types
- [ ] Escape output in templates

## Rate Limiting

- [ ] Add rate limiting to authentication endpoints
- [ ] Add general rate limiting to all routes
- [ ] Configure trust proxy if behind a load balancer

```js
const strictLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 5 });
app.post('/login', strictLimiter, loginHandler);
```

## CORS

- [ ] Restrict allowed origins to your actual domains
- [ ] Only allow necessary HTTP methods
- [ ] Only allow necessary headers
- [ ] Be cautious with `credentials: true`

```js
app.use(cors({ origin: 'https://myapp.com', credentials: true }));
```

## Secrets Management

- [ ] Never hardcode secrets in source code
- [ ] Use `.env` files with `.gitignore`
- [ ] Use different secrets for each environment
- [ ] Rotate secrets regularly
- [ ] Validate required environment variables at startup

## Error Handling

- [ ] Never expose stack traces in production
- [ ] Return generic error messages to users
- [ ] Log detailed errors server-side only
- [ ] Handle uncaught exceptions and unhandled rejections

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message
  });
});
```

## Data Protection

- [ ] Encrypt sensitive data at rest
- [ ] Use TLS for data in transit
- [ ] Do not log sensitive data (passwords, tokens, PII)
- [ ] Implement data retention policies

## Server Configuration

- [ ] Set `NODE_ENV=production` for performance and security
- [ ] Disable directory listing
- [ ] Remove unnecessary default routes
- [ ] Set appropriate file permissions
- [ ] Keep your server OS updated

Security is never done. This checklist is a starting point. Review it before every deployment, and update it as you learn about new threats. Stay safe out there.
