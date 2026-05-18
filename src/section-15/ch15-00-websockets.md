# Real-Time Features in Express

When I first built web apps, everything was request and response. The user clicks, the server responds. That model works for most things, but some features feel wrong without real-time updates. Chat messages, live notifications, collaborative editing, stock tickers. These need data to flow to the user without them asking for it.

That is where real-time features come in.

## Why Real-Time Matters

Think about a chat app. Without real-time, you would have to refresh the page to see new messages. That feels broken. Users expect new messages to appear instantly.

Or a dashboard showing live sales data. Without real-time, the numbers are stale the moment they load. Real-time updates keep the data fresh.

## The Old Way: Polling

Before I learned about WebSockets, I used polling. The client asks the server for updates at regular intervals:

```javascript
// Client-side polling (not great)
setInterval(async () => {
  const response = await fetch('/api/messages');
  const messages = await response.json();
  updateUI(messages);
}, 3000);
```

This works but has big problems:

- **Wasted requests**: Most polls return nothing new
- **Latency**: Updates are delayed by up to the polling interval
- **Server load**: Every connected client hammers your server

Polling is like repeatedly asking "Are we there yet?" on a road trip.

## The Better Way: WebSockets

WebSockets give you a persistent, two-way connection between client and server. Either side can send messages at any time. No polling needed.

```
Client <-----> Server
   (persistent, bidirectional connection)
```

The server can push data to the client instantly. The client can send data to the server anytime. It is like a phone call instead of sending letters back and forth.

## What We Will Build

In this section, I will walk through everything I learned about adding real-time features to Express apps:

- How the WebSocket protocol works and when to use it
- Setting up Socket.io with Express
- Working with events, rooms, and broadcasting
- Building a real-time chat application
- Authenticating WebSocket connections
- Scaling WebSockets across multiple servers

Real-time features transformed my apps from feeling static to feeling alive. Once you see how straightforward it is with Socket.io, you will find excuses to add real-time to everything.
