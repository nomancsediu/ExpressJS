# REST Principles

REST is not a framework or a library. It is an architectural style defined by Roy Fielding in 2000. You do not have to follow every rule, but understanding the principles helps you build better APIs. Let me break down the key ideas.

## Resources

Everything in REST is a resource. A resource is anything you can name: a user, a blog post, a comment, an order. Resources have URLs that identify them:

```
/users           → collection of users
/users/42        → a specific user
/users/42/posts  → posts by a specific user
```

Resources are nouns, not verbs. The URL tells you *what* you are acting on. The HTTP method tells you *what action* to take.

## Statelessness

Each request must contain all the information the server needs to process it. The server does not store session state between requests. Every request stands on its own.

This means no server-side sessions for API state. If you need to know who the user is, include a token in each request:

```js
// Stateless: each request carries auth info
app.get('/profile', requireAuth, (req, res) => {
  res.json(req.user);
});
```

Statelessness makes your API easier to scale because any server can handle any request.

## Uniform Interface

All resources are accessed the same way. You use the same URL patterns, the same HTTP methods, and the same response formats everywhere:

```js
// Same pattern for every resource
app.get('/users', getUsers);          // list
app.get('/users/:id', getUser);       // get one
app.post('/users', createUser);       // create
app.put('/users/:id', updateUser);    // replace
app.delete('/users/:id', deleteUser); // delete
```

The client does not need special knowledge for each endpoint. The patterns are predictable.

## HATEOAS

Hypermedia as the Engine of Application State. This is the most ignored REST principle. It means your API responses include links to related actions:

```json
{
  "id": 42,
  "name": "Alice",
  "email": "alice@test.com",
  "_links": {
    "self": { "href": "/users/42" },
    "posts": { "href": "/users/42/posts" },
    "update": { "href": "/users/42", "method": "PUT" },
    "delete": { "href": "/users/42", "method": "DELETE" }
  }
}
```

I will be honest: I rarely implement full HATEOAS. Most APIs I build are "RESTish" rather than fully RESTful. But knowing about it helps me add helpful links where they matter.

## Other Principles

- **Client-server separation**: The client and server evolve independently.
- **Cacheable**: Responses should declare if they are cacheable.
- **Layered system**: A client cannot tell if it is connected directly to the server or through intermediaries.

These are mostly handled by HTTP itself. Your job is to set proper cache headers and not couple your client and server.

REST is a guideline, not a religion. I follow the parts that make my API better and skip the parts that add complexity without value. The important thing is consistency.
