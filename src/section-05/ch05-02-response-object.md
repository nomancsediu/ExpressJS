# The Response Object

The `res` object is how I send data back to the client. It wraps Node's `http.ServerResponse` and adds a ton of convenience methods. Let me go through them one by one.

## res.json()

Sends a JSON response. This is the one I use the most in API projects:

```js
app.get('/api/users', (req, res) => {
  res.json({ users: [{ name: 'Alice' }, { name: 'Bob' }] });
});
```

It sets the `Content-Type` header to `application/json` automatically and stringifies the object for you. It also handles null, arrays, and primitives.

## res.send()

A general-purpose method that works with strings, objects, buffers, and arrays:

```js
res.send('Hello world');        // text/html
res.send({ name: 'Alice' });    // application/json
res.send(Buffer.from('data'));  // application/octet-stream
```

Express guesses the content type based on what you pass. For HTML strings, it sets `text/html`. For objects, it behaves like `res.json()`.

## res.status()

Sets the HTTP status code. You can chain it with other methods:

```js
res.status(201).json({ created: true });
res.status(404).send('Not found');
res.status(500).json({ error: 'Server error' });
```

## res.end()

Ends the response without sending any data. Useful for empty responses:

```js
res.status(204).end(); // No Content
```

## res.redirect()

Tells the browser to go somewhere else:

```js
res.redirect('/login');             // 302 by default
res.redirect(301, '/new-home');     // permanent redirect
res.redirect('https://example.com'); // external redirect
```

Use 301 for permanent moves and 302 (or 307) for temporary ones. Search engines treat them differently.

## res.render()

Renders a view template. You need a template engine like EJS or Pug set up:

```js
app.set('view engine', 'ejs');

app.get('/profile', (req, res) => {
  res.render('profile', { name: 'Alice', age: 25 });
});
```

I do not use this much since I build APIs, but it is great for server-side rendering.

## res.cookie()

Sets a cookie on the response:

```js
res.cookie('token', 'abc123', {
  httpOnly: true,
  secure: true,
  maxAge: 86400000, // 1 day in milliseconds
  sameSite: 'strict',
});

res.cookie('theme', 'dark'); // simple cookie
```

## res.type()

Sets the `Content-Type` header:

```js
res.type('html');     // text/html
res.type('json');     // application/json
res.type('png');      // image/png
res.type('text/plain'); // also works with full MIME types
```

## Chaining

Most `res` methods return `res`, so you can chain them:

```js
res.status(201).type('json').send('{"created":true}');
```

Just remember that you can only send one response per request. Calling `res.json()` twice on the same request will throw an error.
