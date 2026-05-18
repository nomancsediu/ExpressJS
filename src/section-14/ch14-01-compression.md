# Compression

The first and easiest performance win I ever applied was adding compression. Most Express apps send plain text responses, but browsers can handle compressed responses just fine. Enabling compression typically reduces response sizes by 60-80%.

## Adding the Compression Middleware

```bash
npm install compression
```

```javascript
const compression = require('compression');
const express = require('express');
const app = express();

// Add compression early in the middleware stack
app.use(compression());
```

That is it. One line of code and your responses are compressed. For a JSON response that is 50KB uncompressed, the browser might only download 8KB.

## How It Works

When a browser makes a request, it sends an `Accept-Encoding` header telling the server which compression formats it supports:

```
Accept-Encoding: gzip, deflate, br
```

The compression middleware checks this header, picks the best format, compresses the response, and adds the `Content-Encoding` header. The browser decompresses it automatically. You do nothing on the client side.

## Configuration Options

You can customize the middleware:

```javascript
app.use(compression({
  // Only compress responses larger than 1KB
  threshold: 1024,

  // Compression level (1-9, higher = more compression but slower)
  level: 6,

  // Filter function to decide what to compress
  filter: (req, res) => {
    // Do not compress if the client does not want it
    if (req.headers['x-no-compression']) {
      return false;
    }
    // Use the default filter for everything else
    return compression.filter(req, res);
  },
}));
```

The `threshold` option is important. Compressing tiny responses wastes CPU time for almost no size reduction. I usually set it to 1KB.

## Gzip vs Brotli

Gzip is the default and works everywhere. Brotli is newer and compresses better (typically 15-20% smaller than gzip). Most modern browsers support Brotli.

To use Brotli, install `shrink-ray-current`:

```bash
npm install shrink-ray-current
```

```javascript
const shrinkRay = require('shrink-ray-current');
app.use(shrinkRay());
```

This automatically uses Brotli when the browser supports it and falls back to gzip otherwise.

## When Not to Compress

Not everything benefits from compression:

- **Already compressed files**: Images (JPEG, PNG, WebP) and videos are already compressed. Compressing them again wastes CPU.
- **Small responses**: Under 1KB, compression adds overhead.
- **Server-sent events**: SSE streams can have issues with buffered compression.

I serve static files through Nginx or a CDN with compression handled there, and let the Express compression middleware handle dynamic API responses. This one change made my apps feel noticeably snappier.
