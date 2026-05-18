# Built-in Middleware

Express used to ship with a lot of middleware bundled in. Body parsing, cookies, sessions, they were all included. Starting with Express 4, most of that was moved out. Now Express 5 keeps the same philosophy: only the essentials are built in.

As of Express 5, the built-in middleware functions are:

## express.json()

This parses incoming requests with JSON payloads. It is probably the one you will use the most.

```js
const express = require('express');
const app = express();

app.use(express.json());

app.post('/api/users', (req, res) => {
  console.log(req.body); // the parsed JSON object
  res.json({ received: true });
});
```

Without `express.json()`, `req.body` is `undefined`. I forgot this so many times when I started.

## express.urlencoded()

This parses requests with URL-encoded payloads, which is what HTML forms send by default.

```js
app.use(express.urlencoded({ extended: true }));

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  res.send(`Hello ${username}`);
});
```

Set `extended: true` if you need nested objects. Use `extended: false` for simple key-value pairs with the `querystring` library.

## express.static()

This serves static files like images, CSS, and JavaScript. You point it at a directory and it handles the rest.

```js
app.use(express.static('public'));
app.use('/assets', express.static('files'));
```

Now a request to `/logo.png` serves `public/logo.png`, and `/assets/report.pdf` serves `files/report.pdf`. Super handy for front-end assets.

## express.raw()

This parses the request body as a raw `Buffer`. Useful when you are dealing with binary data.

```js
app.use(express.raw({ type: 'application/octet-stream', limit: '10mb' }));

app.post('/upload', (req, res) => {
  console.log(req.body); // a Buffer
  res.send('Received raw data');
});
```

## express.text()

This parses the request body as a plain string.

```js
app.use(express.text({ type: 'text/plain' }));

app.post('/message', (req, res) => {
  console.log(req.body); // a string
  res.send('Got your message');
});
```

## Quick Tip

You can configure the `limit` option on any body parser to control the maximum payload size:

```js
app.use(express.json({ limit: '1mb' }));
```

This protects your server from someone sending a ridiculously large JSON body. I always set a limit in production now after learning why it matters the hard way.

These five functions cover most of what you need for parsing requests and serving files. Everything else comes from third-party packages.
