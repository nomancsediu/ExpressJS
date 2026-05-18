# Microservices

Monoliths get a bad reputation. They are actually fine for most projects. But when your app grows large enough, splitting it into microservices can make sense. Let me talk about when and how to do it.

## What Are Microservices?

Instead of one big application that does everything, you build several small applications. Each one handles a specific business capability:

- **User Service**: Handles auth and profiles
- **Product Service**: Manages products and inventory
- **Order Service**: Processes orders
- **Payment Service**: Handles payments

Each service has its own database, its own code, and its own deployment cycle. They communicate over the network.

## Service Communication

Services need to talk to each other. There are two main patterns:

### Synchronous (HTTP/REST)

Service A makes an HTTP request to Service B and waits for a response.

```js
// Order service calling Product service
const axios = require('axios');

const checkProductStock = async (productId) => {
  const response = await axios.get(
    `${process.env.PRODUCT_SERVICE_URL}/api/products/${productId}`
  );
  return response.data.data.stock > 0;
};
```

This is simple but creates coupling. If the Product service is down, the Order service fails too.

### Asynchronous (Messages)

Service A publishes a message. Service B picks it up when ready.

```js
// Using RabbitMQ or Redis pub/sub
const publish = require('./message-broker');

// Order service publishes an event
await publish('order.created', { orderId: order._id, items: order.items });

// Product service subscribes to the event
subscribe('order.created', async (data) => {
  for (const item of data.items) {
    await Product.findByIdAndUpdate(item.product, {
      $inc: { stock: -item.quantity },
    });
  }
});
```

Asynchronous communication is more resilient. If the Product service is down, messages queue up and get processed when it comes back online.

## API Gateway

An API gateway sits in front of all services and routes requests to the right one:

```js
// Simplified API gateway using Express
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

app.use('/api/users', createProxyMiddleware({
  target: process.env.USER_SERVICE_URL,
  changeOrigin: true,
}));

app.use('/api/products', createProxyMiddleware({
  target: process.env.PRODUCT_SERVICE_URL,
  changeOrigin: true,
}));

app.use('/api/orders', createProxyMiddleware({
  target: process.env.ORDER_SERVICE_URL,
  changeOrigin: true,
}));

app.listen(3000);
```

Clients talk to the gateway. The gateway forwards to the right service. This hides the internal architecture and lets you change services without breaking clients.

## Service Discovery

In a dynamic environment (containers, Kubernetes), service IPs change frequently. Service discovery helps services find each other.

Simple approach with environment variables:

```bash
USER_SERVICE_URL=http://user-service:3001
PRODUCT_SERVICE_URL=http://product-service:3002
ORDER_SERVICE_URL=http://order-service:3003
```

In Kubernetes, DNS-based discovery works automatically. `http://user-service:3001` resolves to the right pod.

## When to Use Microservices

Do not start with microservices. Start with a monolith. Split services out when:

- A specific module needs to scale independently
- Different teams need to deploy at different speeds
- A module has different reliability requirements

Microservices add complexity: network calls, deployment coordination, data consistency. The payoff is flexibility and independent scaling. Make sure you need that before committing to the complexity.
