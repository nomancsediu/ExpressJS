# Session-Based Authentication

JWT is popular, but sessions are still a solid choice for many apps. Let me explain how session auth works and when I prefer it over JWT.

## How Sessions Work

With sessions, the server stores the user's state and gives the client a session ID. The client sends this ID with every request, and the server looks up the session data.

```
Client                           Server
  |                                 |
  |--- POST /login (credentials) -->|
  |                                 | Create session
  |                                 | Store: { userId: 123, role: 'admin' }
  |<-- Set-Cookie: sid=abc123 -----|
  |                                 |
  |--- GET /profile (with cookie) ->|
  |                                 | Look up session abc123
  |<-- user data -------------------|
```

The key difference from JWT: the server stores the session data. The client only holds a session ID. This means sessions are stateful, while JWT is stateless.

## express-session Setup

```bash
npm install express-session
```

```js
const session = require('express-session');

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 24 * 60 * 60 * 1000, // 1 day
  },
}));
```

Let me explain the options:

- **secret**: Used to sign the session ID cookie. Keep this safe and long.
- **resave**: Whether to save the session back to the store even if it was not modified. I set it to false.
- **saveUninitialized**: Whether to save a new session that has not been modified. I set it to false to prevent empty sessions.
- **cookie.httpOnly**: Prevents JavaScript from accessing the cookie. Protects against XSS.
- **cookie.secure**: Only sends the cookie over HTTPS. Essential for production.
- **cookie.maxAge**: How long the session lasts.

## Login and Logout

```js
router.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;

    const user = await User.findOne({ email }).select('+passwordHash');
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.passwordHash);
    if (!isMatch) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    // Store user info in session
    req.session.userId = user.id;
    req.session.userRole = user.role;

    res.json({ id: user.id, name: user.name, email: user.email });
  } catch (err) {
    next(err);
  }
});

router.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).json({ message: 'Could not log out' });
    res.clearCookie('connect.sid');
    res.json({ message: 'Logged out' });
  });
});
```

## Session Store

By default, express-session stores sessions in memory. This is fine for development but terrible for production. Memory leaks, no persistence across restarts, and does not work with multiple server instances.

I use Redis for production session storage:

```bash
npm install connect-redis
```

```js
const RedisStore = require('connect-redis').default;
const redisClient = require('./redis');

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 86400000,
  },
}));
```

## Sessions vs JWT

I choose based on the use case:

| Factor | Sessions | JWT |
|--------|---------|-----|
| Storage | Server-side | Client-side |
| State | Stateful | Stateless |
| Revocation | Easy (delete from store) | Hard (need blacklist) |
| Scaling | Needs shared store | Works across servers |
| Mobile/API | Cookie handling is tricky | Natural fit |
| Logout | Instant | Wait for expiry |

I use sessions for traditional web apps where the client is a browser. I use JWT for APIs and mobile apps where the client needs to manage tokens directly.
