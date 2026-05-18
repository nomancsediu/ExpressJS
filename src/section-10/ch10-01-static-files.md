# Serving Static Files

Every web app needs to serve static files. Images, stylesheets, client-side JavaScript, fonts. Express makes this easy with the built-in `express.static` middleware.

## Basic Usage

```js
const express = require('express');
const app = express();

// Serve files from the "public" directory
app.use(express.static('public'));
```

Now if you have `public/style.css`, it is available at `http://localhost:3000/style.css`. Express looks for the file and serves it automatically. No route needed.

## Multiple Directories

You can serve files from more than one directory. Just call `express.static` multiple times:

```js
app.use(express.static('public'));
app.use(express.static('files'));
```

Express checks each directory in order. If `public/logo.png` exists, it serves that one first. If not, it checks `files/logo.png`. The first match wins.

## Virtual Path Prefix

Sometimes you want a URL prefix before your static files. Like putting all assets under `/assets`:

```js
app.use('/assets', express.static('public'));
```

Now `public/style.css` is available at `http://localhost:3000/assets/style.css`. The file path stays the same on disk, but the URL changes. I find this helpful for organizing routes and avoiding conflicts.

## Caching

`express.static` sets cache headers by default. You can customize the caching behavior:

```js
app.use(express.static('public', {
  maxAge: '1d',           // Cache for 1 day
  etag: true,             // Enable ETag (default: true)
  lastModified: true,     // Send Last-Modified header
  immutable: true         // Tell browser the file won't change
}));
```

For production, longer cache times save bandwidth. For development, I set `maxAge: 0` so I always get the latest file.

## Other Options

```js
app.use(express.static('public', {
  dotfiles: 'ignore',     // Ignore hidden files like .git
  extensions: ['html'],   // Try adding .html if path not found
  fallthrough: true       // Pass to next middleware if not found
}));
```

The `fallthrough` option is important. When `true` (the default), missing files pass to the next middleware. When `false`, a 404 is sent immediately.

One thing I learned: always put `express.static` before your route handlers. This way, static files are served quickly without going through your route logic.
