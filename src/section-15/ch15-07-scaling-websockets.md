# Scaling WebSockets

Everything works great with one server. But when your app grows and you need multiple instances, WebSockets get tricky. The problem is that a user connected to Server A cannot receive messages emitted from Server B. I ran into this when my chat app went from one server to three.

## The Problem

```
User 1 <---> Server A
User 2 <---> Server B
```

If User 1 sends a message and Server A does `io.emit()`, only User 1 gets it. User 2 is connected to Server B and never sees the message.

## The Redis Adapter

The solution is a shared pub/sub system. When Server A emits a message, it publishes it to Redis. Server B subscribes and forwards it to its connected clients.

```bash
npm install @socket.io/redis-adapter redis
```

```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const app = express();
const server = http.createServer(app);

async function start() {
  const pubClient = createClient({ url: 'redis://localhost:6379' });
  const subClient = pubClient.duplicate();

  await pubClient.connect();
  await subClient.connect();

  const io = new Server(server, {
    adapter: createAdapter(pubClient, subClient),
  });

  io.on('connection', (socket) => {
    console.log('Connected:', socket.id);

    socket.on('chat:message', (data) => {
      // This now reaches ALL clients across ALL servers
      io.emit('chat:message', data);
    });
  });

  server.listen(3000);
}

start();
```

Now when Server A emits a message, Redis forwards it to Server B, and both servers deliver it to their clients.

## How the Redis Adapter Works

```
User 1 <---> Server A <---> Redis <---> Server B <---> User 2
```

1. User 1 sends a message to Server A
2. Server A publishes the message to Redis
3. Redis broadcasts it to all subscribers (Server B)
4. Server B receives it and forwards to User 2

Room operations also work across servers. `io.to('room-1').emit()` reaches all users in room-1, regardless of which server they are connected to.

## Sticky Sessions

When you have a load balancer in front of multiple Socket.io servers, you need sticky sessions. A client must always be routed to the same server. Otherwise, the HTTP handshake and WebSocket upgrade might go to different servers.

With Nginx:

```nginx
upstream socket_servers {
  ip_hash;  # Sticky sessions based on IP
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
  server 127.0.0.1:3003;
}

server {
  listen 80;

  location / {
    proxy_pass http://socket_servers;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

The `ip_hash` directive ensures the same IP always routes to the same server.

## Horizontal Scaling Checklist

Before scaling your WebSocket servers, make sure:

- **Redis adapter is configured** for cross-server messaging
- **Sticky sessions are enabled** on your load balancer
- **Session state is shared** (use Redis for session storage, not memory)
- **CORS is configured** for all server origins
- **Health checks are in place** so the load balancer detects failed instances

## Other Adapter Options

Redis is the most popular, but there are other adapters:

- **@socket.io/mongo-adapter**: Uses MongoDB for pub/sub
- **@socket.io/rabbitmq-adapter**: Uses RabbitMQ
- **@socket.io/postgres-adapter**: Uses PostgreSQL LISTEN/NOTIFY

I stick with Redis because it is fast and I am usually already using it for caching and sessions. Adding pub/sub is one more feature from the same tool.

Scaling WebSockets felt intimidating at first, but it really comes down to two things: a shared pub/sub system and sticky sessions. Get those right, and everything else just works.
