# What Is Middleware?

Middleware is a function that runs between receiving a request and sending a response. That is the simplest definition I can give. Every middleware function has access to the request object (`req`), the response object (`res`), and a function called `next`.

Here is the basic signature:

```js
function myMiddleware(req, res, next) {
  // do something with req or res
  next(); // pass control to the next middleware
}
```

When you call `next()`, Express moves to the next function in the stack. If you do not call `next()`, the request just hangs. The client will wait forever until it times out. I learned this the hard way more than once.

## The Middleware Stack

Express maintains an ordered list of middleware functions. When a request comes in, it goes through them one by one:

```
Request → middleware1 → middleware2 → middleware3 → route handler → Response
```

Think of it like an assembly line. Each station can do something with the request and then pass it along, or it can stop the line entirely by sending a response.

```js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  console.log('First middleware');
  next();
});

app.use((req, res, next) => {
  console.log('Second middleware');
  next();
});

app.get('/', (req, res) => {
  res.send('Hello!');
});
```

When you visit `/`, the console prints:

```
First middleware
Second middleware
```

Then the route handler sends the response.

## Request Lifecycle

The full lifecycle looks like this:

1. Express receives an HTTP request
2. The request enters the middleware stack in order
3. Each middleware can read or modify `req` and `res`
4. A middleware calls `next()` to continue, or sends a response to stop
5. If a response is sent, remaining middleware is skipped
6. The response goes back to the client

Here is a simple diagram:

```
Client Request
     ↓
[Middleware A] → next()
     ↓
[Middleware B] → next()
     ↓
[Route Handler] → res.send()
     ↓
Client Response
```

The key insight for me was that route handlers are just middleware that send a response instead of calling `next()`. They are part of the same stack. Once I saw it that way, everything else about Express started making more sense.
