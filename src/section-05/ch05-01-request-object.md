# The Request Object

The `req` object is an enhanced version of Node's native `http.IncomingMessage`. Express adds a bunch of useful properties and methods on top of it. Let me walk through the ones I use most.

## req.params

Route parameters are the dynamic parts of your URL. You define them with a colon:

```js
app.get('/users/:id', (req, res) => {
  console.log(req.params.id); // the value from the URL
  res.send(`User ID: ${req.params.id}`);
});
```

You can have multiple parameters:

```js
app.get('/posts/:postId/comments/:commentId', (req, res) => {
  const { postId, commentId } = req.params;
  res.json({ postId, commentId });
});
```

## req.query

Query strings are everything after the `?` in a URL:

```js
// GET /search?q=express&page=2
app.get('/search', (req, res) => {
  console.log(req.query.q);    // "express"
  console.log(req.query.page); // "2"
  res.json(req.query);
});
```

Remember: query values are always strings. If you need numbers, parse them yourself:

```js
const page = parseInt(req.query.page, 10) || 1;
```

## req.body

The parsed request body. This only works if you have body-parsing middleware set up:

```js
app.use(express.json());

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  res.json({ name, email });
});
```

Without `express.json()`, `req.body` is `undefined`. I keep forgetting this.

## req.headers

All the HTTP headers as an object:

```js
app.get('/info', (req, res) => {
  console.log(req.headers['content-type']);
  console.log(req.headers['user-agent']);
  res.json(req.headers);
});
```

Headers are lowercased automatically, so `Content-Type` becomes `content-type`.

## req.ip and req.ips

The client IP address. If you are behind a proxy, enable trust proxy first:

```js
app.set('trust proxy', true);

app.get('/my-ip', (req, res) => {
  res.json({
    ip: req.ip,        // client IP
    ips: req.ips,      // array of IPs if behind proxies
  });
});
```

## req.path and req.method

```js
app.use((req, res, next) => {
  console.log(req.method); // "GET", "POST", etc.
  console.log(req.path);   // "/users/42" (without query string)
  console.log(req.url);    // "/users/42?page=1" (with query string)
  console.log(req.originalUrl); // full original URL
  next();
});
```

## req.cookies

Available when you use `cookie-parser`:

```js
app.use(cookieParser());

app.get('/check', (req, res) => {
  console.log(req.cookies);       // all cookies as an object
  console.log(req.cookies.token); // a specific cookie
  res.send('Checked cookies');
});
```

These are the properties I reach for most often. There are more like `req.hostname`, `req.protocol`, `req.secure`, and `req.xhr`, but the ones above cover the vast majority of what I need day to day.
