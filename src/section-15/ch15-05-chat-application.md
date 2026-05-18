# Building a Chat Application

Now let me put everything together and build a real-time chat application. This is the project that made everything click for me. It uses events, rooms, broadcasting, and presence tracking.

## Server-Side Setup

```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' },
});

// Track connected users
const users = new Map();

io.on('connection', (socket) => {
  console.log('Connected:', socket.id);

  // User joins with a username
  socket.on('user:join', (username) => {
    users.set(socket.id, { username, room: null });
    socket.emit('user:welcome', { message: `Welcome, ${username}!` });
    io.emit('users:online', getUsernames());
  });

  // Join a chat room
  socket.on('room:join', (room) => {
    const user = users.get(socket.id);
    if (!user) return;

    // Leave previous room
    if (user.room) {
      socket.leave(user.room);
      socket.to(user.room).emit('user:left', { username: user.username });
    }

    // Join new room
    socket.join(room);
    user.room = room;
    socket.to(room).emit('user:joined', { username: user.username });
    socket.emit('room:joined', { room });
  });

  // Send a message
  socket.on('message:send', (text) => {
    const user = users.get(socket.id);
    if (!user || !user.room) return;

    const message = {
      id: Date.now(),
      username: user.username,
      text,
      timestamp: new Date().toISOString(),
    };

    io.to(user.room).emit('message:new', message);
  });

  // Typing indicator
  socket.on('typing:start', () => {
    const user = users.get(socket.id);
    if (!user || !user.room) return;

    socket.volatile.to(user.room).emit('user:typing', {
      username: user.username,
    });
  });

  // Disconnect
  socket.on('disconnect', () => {
    const user = users.get(socket.id);
    if (user) {
      if (user.room) {
        socket.to(user.room).emit('user:left', { username: user.username });
      }
      users.delete(socket.id);
      io.emit('users:online', getUsernames());
    }
    console.log('Disconnected:', socket.id);
  });
});

function getUsernames() {
  return [...users.values()].map(u => u.username);
}

server.listen(3000);
```

## Client-Side Code

```javascript
import { io } from 'socket.io-client';

const socket = io();

// Join with username
socket.emit('user:join', 'Alice');

// Join a room
socket.emit('room:join', 'general');

// Send a message
function sendMessage(text) {
  socket.emit('message:send', text);
}

// Listen for new messages
socket.on('message:new', (msg) => {
  addMessageToUI(msg);
});

// Typing indicator
socket.on('user:typing', ({ username }) => {
  showTypingIndicator(username);
});

// Listen for user joining/leaving
socket.on('user:joined', ({ username }) => {
  addSystemMessage(`${username} joined the room`);
});

socket.on('user:left', ({ username }) => {
  addSystemMessage(`${username} left the room`);
});

// Typing detection on input
let typingTimeout;
messageInput.addEventListener('input', () => {
  socket.emit('typing:start');
  clearTimeout(typingTimeout);
  typingTimeout = setTimeout(() => {}, 2000);
});
```

## Key Features Implemented

- **Rooms**: Users join specific chat rooms and only see messages for that room
- **Presence**: Online users list updates in real-time when people join or leave
- **Typing indicators**: Volatile events so missed indicators are fine
- **Message broadcasting**: `io.to(room)` sends to everyone including the sender for consistency

This chat app is the foundation I build on for more complex real-time features. Adding private messages, file sharing, or read receipts all follow the same event-based pattern.
