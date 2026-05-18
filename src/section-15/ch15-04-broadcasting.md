# Broadcasting

Broadcasting means sending a message to multiple clients at once. Socket.io gives you several ways to do this, and choosing the right one matters.

## io.emit: To Everyone

The broadest option. Sends to every connected client:

```javascript
// Server sends to ALL connected clients
io.emit('announcement', { text: 'Server is restarting in 5 minutes' });
```

I use this sparingly. Not every message is relevant to every user.

## broadcast: To Everyone Except the Sender

The `broadcast` flag sends to all connected clients except the one that triggered the event:

```javascript
io.on('connection', (socket) => {
  socket.on('chat:message', (data) => {
    // Everyone gets this EXCEPT the sender
    socket.broadcast.emit('chat:message', {
      from: socket.id,
      text: data.text,
    });

    // Acknowledge to sender that their message went through
    socket.emit('chat:sent', { text: data.text });
  });
});
```

This is the most common pattern I use. When someone sends a chat message, they already see their own message locally. Broadcasting to others avoids duplicates.

## Room Broadcasting

Combining rooms with broadcasting is where things get powerful:

```javascript
// To everyone in the room
io.to('room-1').emit('room:update', data);

// To everyone in the room EXCEPT the sender
socket.to('room-1').emit('room:message', data);

// To all rooms the sender is in (except the sender)
socket.broadcast.emit('room:message', data);
```

A practical example for a chat room:

```javascript
socket.on('chat:send', (data) => {
  const { room, text } = data;

  // Sender sees their own message immediately
  socket.emit('chat:sent', { id: Date.now(), text, from: 'you' });

  // Everyone else in the room gets the message
  socket.to(room).emit('chat:received', {
    id: Date.now(),
    text,
    from: socket.id,
    timestamp: Date.now(),
  });
});
```

## Excluding the Sender

There are multiple ways to exclude the sender:

```javascript
// Method 1: socket.broadcast
socket.broadcast.emit('event', data);

// Method 2: socket.to() for rooms
socket.to('room-1').emit('event', data);

// Method 3: io.except() for fine-grained control
io.except(socket.id).emit('event', data);
```

Method 3 is useful when you want to exclude specific sockets, not just the sender.

## Volatile Events

Sometimes you want to send data that is okay to lose. Think real-time cursors or stock prices. If a client is slow, dropping old updates is better than queuing them:

```javascript
// Volatile: if the client is not ready, the message is dropped
io.volatile.emit('cursor:position', { x: 150, y: 200 });

// Regular: messages are queued if the client is slow
io.emit('chat:message', { text: 'Hello' });
```

I use volatile events for:

- Mouse cursor positions
- Live score updates
- Real-time sensor data
- Any data where only the latest value matters

For chat messages or notifications, I always use regular events. Losing those would be a bug.

## A Complete Broadcasting Example

```javascript
io.on('connection', (socket) => {
  // User joins a room
  socket.on('join', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('user:joined', { userId: socket.id });
  });

  // User sends a message
  socket.on('message', ({ room, text }) => {
    socket.to(room).emit('message', { from: socket.id, text });
  });

  // User is typing (volatile, okay to miss)
  socket.on('typing', (room) => {
    socket.volatile.to(room).emit('user:typing', { userId: socket.id });
  });

  // User leaves
  socket.on('leave', (roomId) => {
    socket.leave(roomId);
    socket.to(roomId).emit('user:left', { userId: socket.id });
  });
});
```

This pattern covers most real-time scenarios I build.
