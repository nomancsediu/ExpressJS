# The WebSocket Protocol

Before I started using WebSockets, I only knew HTTP. Understanding the difference between HTTP and WebSocket helped me decide when to use each one.

## HTTP vs WebSocket

HTTP is a request-response protocol. The client sends a request, the server sends a response, and the connection closes. Every message requires a new connection (or at least a new request on a keep-alive connection).

WebSocket is different. It starts as an HTTP request, then upgrades to a persistent connection. After the upgrade, both sides can send messages freely. The connection stays open until one side closes it.

```
HTTP:        Client --> Request --> Server
             Client <-- Response <-- Server
             (Connection done)

WebSocket:   Client --> Upgrade Request --> Server
             Client <-- Upgrade Response <-- Server
             (Connection stays open)
             Client <-- Message <-- Server
             Client --> Message --> Server
             Client <-- Message <-- Server
             ... (as long as needed)
```

## The Handshake

A WebSocket connection starts with an HTTP upgrade request:

```http
GET /socket HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

The server responds:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

After this handshake, the protocol switches from HTTP to WebSocket. You normally do not handle this yourself. Libraries like `ws` or Socket.io do it for you.

## Persistent Connections

The biggest advantage of WebSocket is persistence. Once established, the connection stays open. This means:

- **No request overhead**: Messages do not carry HTTP headers
- **Instant delivery**: The server pushes data immediately
- **Low latency**: No reconnecting or re-handshaking
- **Efficient**: Small message frames instead of full HTTP requests

## When to Use WebSockets

I use WebSockets when:

- The server needs to push updates to clients (chat, notifications, live data)
- Messages are frequent and small
- Low latency matters
- I need true bidirectional communication

I stick with HTTP when:

- The client requests data occasionally
- Responses are large (file downloads, reports)
- Caching matters (HTTP has built-in caching, WebSocket does not)
- REST semantics are important (idempotent, cacheable endpoints)

## A Simple WebSocket Example

Using the `ws` library directly:

```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  console.log('New client connected');

  ws.on('message', (message) => {
    console.log('Received:', message.toString());
    ws.send('Echo: ' + message.toString());
  });

  ws.on('close', () => {
    console.log('Client disconnected');
  });
});
```

This works, but in practice I almost always use Socket.io instead. It handles reconnection, fallbacks, rooms, and much more. We will set it up in the next chapter.
