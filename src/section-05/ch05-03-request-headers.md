# Request Headers

HTTP headers carry metadata about the request. The client sends them with every request, and they tell your server things like what format the client wants, who is making the request, and what kind of data is in the body.

## Reading Headers

The `req.headers` object contains all headers, automatically lowercased:

```js
app.get('/debug', (req, res) => {
  console.log(req.headers['content-type']);
  console.log(req.headers['authorization']);
  console.log(req.headers['user-agent']);
  res.json(req.headers);
});
```

## req.get()

Express provides a cleaner way to read headers with `req.get()`. It is case-insensitive:

```js
app.get('/info', (req, res) => {
  const contentType = req.get('Content-Type');
  const auth = req.get('authorization');
  const agent = req.get('USER-AGENT');

  res.json({ contentType, auth, agent });
});
```

I prefer `req.get()` because I do not have to worry about casing. Either way works.

## req.accepts()

This checks what content types the client can handle. It reads the `Accept` header:

```js
app.get('/data', (req, res) => {
  if (req.accepts('json')) {
    res.json({ message: 'Hello' });
  } else if (req.accepts('html')) {
    res.send('<h1>Hello</h1>');
  } else {
    res.status(406).send('Not Acceptable');
  }
});
```

You can also pass an array:

```js
const type = req.accepts(['json', 'xml', 'text']);
// returns the best match, or false
```

## Custom Headers

You can send and read custom headers. A common pattern is using `X-` prefixed headers for app-specific metadata, though that convention is outdated now:

```js
// Client sends:
// X-Request-ID: abc-123

// Server reads:
app.use((req, res, next) => {
  req.id = req.get('X-Request-ID') || generateId();
  next();
});
```

I use custom headers for things like request tracing, API keys, and rate limit info.

## Common Headers You Will See

| Header | What It Means |
|---|---|
| `Content-Type` | Format of the request body |
| `Authorization` | Credentials for authentication |
| `Accept` | What response formats the client wants |
| `User-Agent` | Info about the client software |
| `Host` | The domain the request was sent to |
| `Cookie` | Cookies stored for this domain |
| `X-Forwarded-For` | Original IP when behind a proxy |
| `If-None-Match` | ETag for conditional requests |

Understanding headers has helped me debug so many issues. When something goes wrong with CORS, authentication, or content types, the answer is usually in the headers. I always check them first now.
