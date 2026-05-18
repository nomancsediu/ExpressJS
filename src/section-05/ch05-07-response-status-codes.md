# Response Status Codes

HTTP status codes tell the client what happened with their request. I used to just use 200 for everything, but proper status codes make your API much easier to use and debug.

## res.status()

Set the status code before sending the response:

```js
res.status(200).json({ data: users });
res.status(201).json({ created: true });
res.status(400).json({ error: 'Bad request' });
res.status(404).json({ error: 'Not found' });
res.status(500).json({ error: 'Server error' });
```

## 2xx Success Codes

These mean the request worked:

- **200 OK**: The standard success response. Used for GET and PUT.
- **201 Created**: A new resource was created. Used for POST.
- **204 No Content**: Success, but no body to return. Used for DELETE.

```js
app.get('/users', (req, res) => {
  res.status(200).json(users);
});

app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);
});

app.delete('/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).end();
});
```

## 3xx Redirection Codes

- **301 Moved Permanently**: The resource has a new permanent URL.
- **302 Found**: Temporary redirect.
- **304 Not Modified**: The cached version is still valid.

```js
app.get('/old-page', (req, res) => {
  res.redirect(301, '/new-page');
});
```

## 4xx Client Error Codes

The client made a mistake:

- **400 Bad Request**: Malformed request body or parameters.
- **401 Unauthorized**: Authentication is required.
- **403 Forbidden**: Authenticated but not allowed.
- **404 Not Found**: The resource does not exist.
- **409 Conflict**: The request conflicts with current state.
- **422 Unprocessable Entity**: Valid format but semantic errors.
- **429 Too Many Requests**: Rate limited.

```js
app.post('/users', (req, res) => {
  if (!req.body.email) {
    return res.status(400).json({ error: 'Email is required' });
  }
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Login required' });
  }
  const user = findUser(req.body.email);
  if (user) {
    return res.status(409).json({ error: 'Email already exists' });
  }
  res.status(201).json(createUser(req.body));
});
```

## 5xx Server Error Codes

The server messed up:

- **500 Internal Server Error**: Something broke on the server.
- **502 Bad Gateway**: The upstream server returned an invalid response.
- **503 Service Unavailable**: The server is overloaded or down for maintenance.

```js
app.get('/data', async (req, res) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

## A Simple Rule

I follow this rule: if the client did something wrong, use 4xx. If the server did something wrong, use 5xx. And for success, pick the most specific 2xx code. Your API users will thank you.
