# Setting Up Socket.io with Express

Socket.io is the library I use for real-time features in Express. It sits on top of WebSockets and adds reconnection, fallbacks, rooms, and a nice event-based API. It makes real-time code much simpler to write.

## Installing Socket.io

```bash
npm install socket.io
```

## Basic Setup with Express

The key is to attach Socket.io to the HTTP server that Express creates under the hood:

```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

io.on('connection', (socket) => {
  console.log('A user connected:', socket.id);

  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

Notice I use `server.listen()` instead of `app.listen()`. This is because Socket.io needs access to the raw HTTP server.

## Client-Side Setup

On the client, include the Socket.io client library:

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();

  socket.on('connect', () => {
    console.log('Connected to server:', socket.id);
  });

  socket.on('disconnect', () => {
    console.log('Disconnected from server');
  });
</script>
```

Or with a bundler:

```bash
npm install socket.io-client
```

```javascript
import { io } from 'socket.io-client';

const socket = io();
```

## Connection Events

Socket.io provides built-in events for connection lifecycle:

```javascript
io.on('connection', (socket) => {
  console.log('Connected:', socket.id);

  socket.on('disconnect', (reason) => {
    console.log('Disconnected:', socket.id, 'Reason:', reason);
  });

  socket.on('connect_error', (error) => {
    console.error('Connection error:', error.message);
  });
});
```

The `reason` parameter in `disconnect` tells you why the connection closed. Common reasons are `client namespace disconnect` (client called `disconnect()`) or `transport close` (network issue).

## Namespaces

Namespaces let you separate different channels of communication on the same server. Think of them as separate pipelines:

```javascript
// Main namespace (default)
io.on('connection', (socket) => {
  console.log('Main namespace connection');
});

// Custom namespace
const chat = io.of('/chat');
chat.on('connection', (socket) => {
  console.log('Chat namespace connection');

  socket.on('message', (data) => {
    chat.emit('message', data);
  });
});

// Another namespace
const notifications = io.of('/notifications');
notifications.on('connection', (socket) => {
  console.log('Notifications namespace connection');
});
```

Client connects to a specific namespace:

```javascript
const chatSocket = io('/chat');
const notifSocket = io('/notifications');
```

Namespaces are useful for separating concerns. I use `/chat` for messaging and `/notifications` for alerts. They do not interfere with each other, and each can have its own authentication and event handlers.
