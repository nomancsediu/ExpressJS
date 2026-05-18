# CORS

Cross-Origin Resource Sharing, or CORS, is a browser security feature. It controls whether a web page from one origin can make requests to a different origin. If your API runs on `api.example.com` and your frontend runs on `example.com`, you need CORS configured properly.

## The Problem

Browsers block cross-origin requests by default. If your frontend at `http://localhost:5173` tries to fetch `http://localhost:3000/api/data`, the browser says no. This is the Same-Origin Policy, and it protects users from malicious websites stealing data from other sites.

But legitimate apps need cross-origin requests. CORS is the mechanism that lets you allow them safely.

## Installation

```bash
npm install cors
```

## Basic Usage

```js
const express = require('express');
const cors = require('cors');
const app = express();

// Allow all origins (not recommended for production)
app.use(cors());
```

This adds the `Access-Control-Allow-Origin: *` header to every response. Any website can now call your API. Convenient for development, risky for production.

## Allowed Origins

Restrict CORS to specific domains:

```js
const corsOptions = {
  origin: 'https://myapp.com'
};

app.use(cors(corsOptions));
```

For multiple origins:

```js
const allowedOrigins = [
  'https://myapp.com',
  'https://admin.myapp.com',
  'http://localhost:5173'  // For development
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));
```

The `!origin` check handles server-to-server requests and tools like Postman that do not send an Origin header.

## Credentials

By default, CORS does not send cookies or authorization headers. If your API uses cookies for authentication, you need:

```js
app.use(cors({
  origin: 'https://myapp.com',
  credentials: true  // Allow cookies and auth headers
}));
```

On the frontend, you also need to set credentials:

```js
fetch('http://localhost:3000/api/data', {
  credentials: 'include'
});
```

Important: when `credentials: true`, you cannot use `origin: '*'`. You must specify exact origins.

## Preflight Requests

For complex requests (like PUT, DELETE, or requests with custom headers), the browser sends a preflight OPTIONS request first. The `cors` middleware handles this automatically.

You can configure which methods and headers are allowed:

```js
app.use(cors({
  origin: 'https://myapp.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  maxAge: 86400  // Cache preflight for 24 hours
}));
```

The `maxAge` option reduces preflight requests by telling the browser to cache the result. This improves performance for apps that make many cross-origin requests.

## Per-Route CORS

You can apply CORS to specific routes only:

```js
app.get('/public', cors(), (req, res) => {
  res.json({ data: 'anyone can see this' });
});

app.post('/private', cors({ origin: 'https://myapp.com' }), (req, res) => {
  res.json({ data: 'only myapp.com can see this' });
});
```

I configure CORS early in my projects, even before I need it. Debugging CORS errors later is frustrating because the errors are confusing and the browser blocks things silently.
