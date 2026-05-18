# Authenticating WebSocket Connections

By default, anyone can connect to your Socket.io server. In production, you need to verify who is connecting. I learned this the hard way when I found unauthorized users connecting to my app and reading private messages.

## Authenticating with Tokens

The most common approach is passing a JWT token when the client connects:

```javascript
// Client sends token on connection
const socket = io({
  auth: {
    token: localStorage.getItem('authToken'),
  },
});
```

On the server, Socket.io middleware checks the token:

```javascript
io.use(async (socket, next) => {
  const token = socket.handshake.auth.token;

  if (!token) {
    return next(new Error('Authentication error: no token provided'));
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.user = decoded; // Attach user data to the socket
    next();
  } catch (err) {
    next(new Error('Authentication error: invalid token'));
  }
});
```

If `next()` is called with an error, the connection is rejected. The client receives a `connect_error` event:

```javascript
socket.on('connect_error', (err) => {
  if (err.message === 'Authentication error: no token provided') {
    // Redirect to login
    window.location.href = '/login';
  }
});
```

## Socket.io Middleware

Socket.io middleware runs for every new connection. It works like Express middleware but for WebSockets:

```javascript
// Multiple middleware can be chained
io.use(async (socket, next) => {
  // First: verify the token
  const token = socket.handshake.auth.token;
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.user = decoded;
    next();
  } catch (err) {
    next(new Error('Unauthorized'));
  }
});

io.use(async (socket, next) => {
  // Second: check if user still exists and is active
  const user = await User.findById(socket.user.id);
  if (!user || user.banned) {
    return next(new Error('Account suspended'));
  }
  next();
});
```

Each middleware calls `next()` to pass control to the next one, just like Express.

## Query-Based Authentication

Some people pass tokens in the query string instead:

```javascript
// Client
const socket = io({ query: { token: myToken } });

// Server
io.use((socket, next) => {
  const token = socket.handshake.query.token;
  // Verify token...
});
```

This works but be careful. Query strings can appear in server logs, which is a security risk for tokens. I prefer the `auth` option.

## Reconnecting with Fresh Tokens

JWT tokens expire. When a long-lived WebSocket connection tries to reconnect with an expired token, it fails. I handle this by refreshing the token before reconnecting:

```javascript
socket.on('connect_error', async (err) => {
  if (err.message.includes('expired')) {
    // Get a fresh token
    const newToken = await refreshToken();
    localStorage.setItem('authToken', newToken);

    // Update the auth option and reconnect
    socket.auth.token = newToken;
    socket.connect();
  }
});
```

## Per-Namespace Authentication

Different namespaces can have different authentication:

```javascript
const chat = io.of('/chat');
chat.use((socket, next) => {
  // Only authenticated users can use chat
  verifyToken(socket, next);
});

const public = io.of('/public');
// No middleware, anyone can connect
```

This is useful when some features need authentication and others do not. I use it to let unauthenticated users see public notifications while requiring login for chat.

Authentication is not optional for production apps. Without it, anyone can connect, join rooms, and read or send messages. Adding it early saves a lot of trouble later.
