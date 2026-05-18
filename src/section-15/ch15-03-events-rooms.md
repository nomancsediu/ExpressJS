# Events and Rooms

Socket.io uses events for communication, and rooms for grouping connections. These two concepts are the building blocks of every real-time feature I build.

## Custom Events

Events are how you send data between client and server. You emit events with a name and optional data:

**Server emitting to client:**

```javascript
io.on('connection', (socket) => {
  // Send a welcome event when someone connects
  socket.emit('welcome', { message: 'Hello! You are connected.' });

  // Listen for a chat message from the client
  socket.on('chat:message', (data) => {
    console.log('Message received:', data);
  });
});
```

**Client emitting to server:**

```javascript
const socket = io();

socket.on('welcome', (data) => {
  console.log(data.message);
});

socket.emit('chat:message', {
  text: 'Hello everyone!',
  timestamp: Date.now(),
});
```

I name events with colons like `chat:message` or `user:typing` to keep them organized. It is a convention, not a requirement, but it helps when you have many events.

## Acknowledgements

You can request confirmation that an event was received:

```javascript
// Server
socket.on('chat:message', (data, callback) => {
  console.log('Message:', data);
  callback({ status: 'received', id: Date.now() });
});

// Client
socket.emit('chat:message', { text: 'Hello' }, (response) => {
  console.log('Server acknowledged:', response);
});
```

This is like delivery confirmation for events. I use it when I need to guarantee the server processed the message.

## Rooms

Rooms are groups of sockets. You can join a room, leave a room, and send messages to everyone in a room:

```javascript
io.on('connection', (socket) => {
  // Join a room
  socket.on('room:join', (roomName) => {
    socket.join(roomName);
    socket.emit('room:joined', { room: roomName });
  });

  // Leave a room
  socket.on('room:leave', (roomName) => {
    socket.leave(roomName);
    socket.emit('room:left', { room: roomName });
  });

  // Send a message to a specific room
  socket.on('room:message', (data) => {
    io.to(data.room).emit('room:message', {
      from: socket.id,
      text: data.text,
      timestamp: Date.now(),
    });
  });
});
```

## Joining and Leaving

A socket can be in multiple rooms at the same time:

```javascript
socket.join('general');    // Now in 'general'
socket.join('random');     // Now in both 'general' and 'random'

// Check which rooms a socket is in
const rooms = socket.rooms; // Set { socket.id, 'general', 'random' }

// Leave one room
socket.leave('random');    // Still in 'general'
```

When a socket disconnects, it automatically leaves all rooms. You do not need to clean up manually.

## Room Messaging

There are several ways to send messages within rooms:

```javascript
// To everyone in the room including sender
io.to('general').emit('message', data);

// To everyone in the room EXCEPT the sender
socket.to('general').emit('message', data);

// To multiple rooms at once
io.to('general').to('random').emit('announcement', data);

// To everyone in the room except certain sockets
io.in('general').except(socket.id).emit('message', data);
```

Rooms are the key to building features like group chats, game lobbies, and team channels. Each group gets its own room, and messages only go to members of that room.
