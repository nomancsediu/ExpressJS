# Content Negotiation

Content negotiation lets your server send different response formats based on what the client asks for. The client tells you what it wants through the `Accept` header, and you respond accordingly. I did not know this was a thing until I needed to support both JSON and XML from the same endpoint.

## res.format()

Express provides `res.format()` to handle content negotiation cleanly:

```js
app.get('/users', (req, res) => {
  const users = [{ name: 'Alice' }, { name: 'Bob' }];

  res.format({
    'application/json': () => {
      res.json(users);
    },
    'text/xml': () => {
      const xml = users.map(u => `<user>${u.name}</user>`).join('');
      res.type('xml').send(`<users>${xml}</users>`);
    },
    'text/html': () => {
      const html = users.map(u => `<li>${u.name}</li>`).join('');
      res.send(`<ul>${html}</ul>`);
    },
    'default': () => {
      res.status(406).send('Not Acceptable');
    },
  });
});
```

If the client sends `Accept: application/json`, they get JSON. If they send `Accept: text/xml`, they get XML. If they ask for something you do not support, the `default` handler runs.

## The Accept Header

The `Accept` header can get complex. It supports quality values and wildcards:

```
Accept: application/json, text/html;q=0.9, */*;q=0.1
```

This means: "I prefer JSON, but HTML is fine too, and I will accept anything as a last resort." The `q` value ranges from 0 to 1, with 1 being the most preferred.

## req.accepts()

Check what the client accepts before sending a response:

```js
app.get('/data', (req, res) => {
  const type = req.accepts(['json', 'html', 'xml']);

  if (type === 'json') {
    res.json({ data: 'hello' });
  } else if (type === 'html') {
    res.send('<p>hello</p>');
  } else if (type === 'xml') {
    res.type('xml').send('<data>hello</data>');
  } else {
    res.status(406).send('Not Acceptable');
  }
});
```

`req.accepts()` returns the best match or `false` if nothing matches.

## API Versioning via Headers

Some APIs use the `Accept` header for versioning:

```
Accept: application/vnd.myapi.v2+json
```

You can handle this in middleware:

```js
app.use((req, res, next) => {
  const accept = req.get('Accept');
  const match = accept && accept.match(/vnd\.myapi\.v(\d+)\+json/);

  req.apiVersion = match ? parseInt(match[1], 10) : 1;
  next();
});

app.get('/users', (req, res) => {
  if (req.apiVersion >= 2) {
    res.json({ data: users, meta: { version: 2 } });
  } else {
    res.json(users);
  }
});
```

This is called content negotiation-based versioning. It keeps your URLs clean. I like it, but it can be harder for clients to discover.

Content negotiation is not something you need every day, but when you do need it, `res.format()` makes it straightforward. It is one of those Express features I am glad I learned about.
