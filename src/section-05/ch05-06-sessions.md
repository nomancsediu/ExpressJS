# Sessions

Sessions let you store data on the server for a given user. Unlike cookies, the data stays on your server. The browser only holds a session ID, and that ID points to the stored data. This is more secure because sensitive information never leaves the server.

## express-session

The standard session middleware for Express:

```bash
npm install express-session
```

```js
const session = require('express-session');

app.use(session({
  secret: 'my-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 3600000, // 1 hour
  },
}));

app.get('/login', (req, res) => {
  req.session.userId = 42;
  req.session.role = 'admin';
  res.send('Logged in');
});

app.get('/profile', (req, res) => {
  if (req.session.userId) {
    res.json({
      userId: req.session.userId,
      role: req.session.role,
    });
  } else {
    res.status(401).send('Not logged in');
  }
});

app.get('/logout', (req, res) => {
  req.session.destroy(() => {
    res.send('Logged out');
  });
});
```

## Session Options

The important options I learned about:

- **secret**: Used to sign the session ID cookie. Use a long, random string. In production, use an environment variable.
- **resave**: Forces the session to be saved back to the store even if it was not modified. Set to `false` unless your store requires it.
- **saveUninitialized**: Saves a new session that has not been modified. Set to `false` to avoid creating empty sessions for every visitor.
- **cookie**: Same options as regular cookies. Always use `httpOnly: true`.

## Storage Options

By default, `express-session` stores sessions in memory. This is fine for development but terrible for production. Memory storage leaks, does not scale, and resets when the server restarts.

For production, use a session store:

```bash
npm install connect-redis
```

```js
const RedisStore = require('connect-redis');
const { createClient } = require('redis');

const redisClient = createClient({ url: 'redis://localhost:6379' });
redisClient.connect();

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
}));
```

Other popular stores include `connect-mongo` for MongoDB and `connect-pg-simple` for PostgreSQL.

## Sessions vs Cookies

| Feature | Cookies | Sessions |
|---|---|---|
| Where data lives | Browser | Server |
| Size limit | ~4KB | Unlimited (server-side) |
| Security | Data is visible to user | User only sees session ID |
| Server state | Stateless | Stateful |
| Scaling | Easy | Needs shared store |

I use sessions for authentication and cookies for small, non-sensitive preferences.

## Production Tips

- Always use a persistent session store, not memory
- Set `secure: true` on the cookie when using HTTPS
- Rotate your secret periodically
- Set a reasonable expiration time
- Use `sameSite: 'strict'` to prevent CSRF

Sessions are powerful but add complexity. If you are building a stateless API, JWT tokens might be a better fit. But for traditional web apps with server-rendered pages, sessions are still my go-to.
