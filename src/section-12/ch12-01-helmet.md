# Helmet.js

Helmet.js sets various HTTP headers to protect your Express app from common web vulnerabilities. It is the first security package I install in every project.

## What Helmet Protects Against

By default, Helmet sets these headers:

- **Content-Security-Policy** prevents XSS by controlling what resources can load
- **X-Content-Type-Options** stops MIME type sniffing
- **X-Frame-Options** prevents clickjacking
- **X-XSS-Protection** enables browser XSS filters
- **Strict-Transport-Security** enforces HTTPS
- **Referrer-Policy** controls referrer information
- **Cross-Origin headers** various cross-origin protections

These headers tell browsers to be more careful when handling your pages. Without them, browsers make assumptions that attackers can exploit.

## Installation

```bash
npm install helmet
```

## Basic Usage

```js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet());
```

That one line adds 15+ security headers. It is the easiest security win you will ever get.

## Customizing Headers

Not every header works for every app. You can enable or disable individual middlewares:

```js
app.use(helmet({
  contentSecurityPolicy: false,  // Disable CSP (we will set it manually)
  crossOriginEmbedderPolicy: false, // May break embedded resources
}));

// Or configure individual headers
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", 'https://fonts.googleapis.com'],
    scriptSrc: ["'self'"],
    imgSrc: ["'self'", 'data:', 'https://res.cloudinary.com'],
    fontSrc: ["'self'", 'https://fonts.gstatic.com'],
    connectSrc: ["'self'"],
  }
}));
```

## Common Issues

Helmet can break things. I have spent hours debugging why my CSS or scripts stopped loading. It is almost always the Content Security Policy blocking resources from external domains.

```js
// If your app uses external scripts like analytics
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", 'https://www.googletagmanager.com'],
  }
}));
```

## Disabling Specific Protections

```js
app.use(helmet({
  // Disable if you need to embed your page in iframes
  xFrameOptions: false,

  // Disable during development if it causes issues
  contentSecurityPolicy: false,
}));
```

I recommend starting with the full default Helmet and then relaxing specific headers as needed. It is better to start secure and open holes deliberately than to start open and try to close holes later.

## Checking Your Headers

Use curl or browser dev tools to verify headers are set:

```bash
curl -I http://localhost:3000
```

You should see headers like `X-Content-Type-Options: nosniff` and `X-Frame-Options: SAMEORIGIN` in the response.

Helmet is not a silver bullet. It does not fix your code vulnerabilities. But it adds a strong layer of browser-level protection with almost zero effort. There is no reason not to use it.
