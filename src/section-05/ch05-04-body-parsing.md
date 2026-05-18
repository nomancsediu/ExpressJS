# Body Parsing

When a client sends data to your server, it arrives as a raw stream of bytes. Body parsing middleware converts that stream into something usable on `req.body`. I used to take this for granted, but understanding how each parser works has saved me from many bugs.

## JSON Parsing

The most common parser. It reads the request body and parses it as a JSON object:

```js
app.use(express.json());

app.post('/api/users', (req, res) => {
  console.log(req.body); // { name: 'Alice', email: 'alice@test.com' }
  res.json({ received: true });
});
```

The client must send `Content-Type: application/json`. If they send something else, the body is not parsed.

## URL-Encoded Parsing

Used for HTML form submissions. The data comes as `key=value&key2=value2`:

```js
app.use(express.urlencoded({ extended: true }));

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  res.send(`Welcome ${username}`);
});
```

The `extended` option controls the parsing library:
- `extended: false` uses the `querystring` library. Simple key-value pairs only.
- `extended: true` uses the `qs` library. Supports nested objects and arrays.

```js
// With extended: true, this form data:
// user[name]=Alice&user[age]=25
// becomes: { user: { name: 'Alice', age: '25' } }
```

## Raw Parsing

Gets the body as a `Buffer`. Useful for binary data or webhooks with custom formats:

```js
app.use(express.raw({ type: 'application/octet-stream' }));

app.post('/upload', (req, res) => {
  console.log(req.body); // Buffer
  console.log(req.body.length); // size in bytes
  res.send('Received');
});
```

## Text Parsing

Gets the body as a plain string:

```js
app.use(express.text({ type: 'text/plain' }));

app.post('/message', (req, res) => {
  console.log(req.body); // string
  res.send('Got it');
});
```

## Multipart Data

Express does not have built-in multipart parsing. You need a library like `multer`:

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('avatar'), (req, res) => {
  console.log(req.file);  // file info
  console.log(req.body);  // text fields
  res.json({ uploaded: true });
});
```

## Size Limits

Always set limits to protect your server from oversized payloads:

```js
app.use(express.json({ limit: '1mb' }));
app.use(express.urlencoded({ limit: '10mb', extended: true }));
app.use(express.raw({ limit: '50mb' }));
```

I once forgot to set a limit and someone sent a 500MB JSON body. The server crashed. Now I always set limits.

## Combining Parsers

You can use multiple parsers together:

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

Express will try each parser based on the `Content-Type` header. JSON bodies get parsed as JSON, form bodies get parsed as URL-encoded. They do not conflict with each other.
