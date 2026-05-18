# Message Queues

Some tasks are slow and do not need to happen right away. Sending emails, processing images, generating reports. If you do these in a request handler, the user waits. Message queues let you offload slow work to background processes.

## The Problem

Imagine a user places an order. The handler needs to:

1. Create the order record (fast)
2. Send a confirmation email (slow, 2-5 seconds)
3. Update inventory (fast)
4. Notify the warehouse (slow, external API)
5. Generate an invoice PDF (slow)

If I do all of this synchronously, the user waits 5-10 seconds for a response. That is a terrible experience. The user only needs to know the order was created. Everything else can happen in the background.

## BullMQ with Redis

BullMQ is a popular queue library for Node.js. It uses Redis as the message broker.

```bash
npm install bullmq ioredis
```

### Setting Up the Queue

```js
// src/queues/orderQueue.js
const { Queue, Worker } = require('bullmq');
const redis = require('../config/redis');

const connection = { host: '127.0.0.1', port: 6379 };

// Create a queue
const orderQueue = new Queue('orders', { connection });

module.exports = orderQueue;
```

### Adding Jobs to the Queue

In the order controller, I add background jobs instead of doing everything inline:

```js
const orderQueue = require('../../queues/orderQueue');

exports.createOrder = asyncHandler(async (req, res) => {
  // Create the order (fast, synchronous)
  const order = await Order.create({ ... });

  // Add background jobs (fast, returns immediately)
  await orderQueue.add('send-confirmation-email', {
    orderId: order._id,
    userEmail: req.user.email,
    userName: req.user.name,
  });

  await orderQueue.add('update-inventory', {
    items: order.items,
  });

  await orderQueue.add('notify-warehouse', {
    orderId: order._id,
    shippingAddress: order.shippingAddress,
  });

  // Respond immediately
  res.status(201).json({ success: true, data: order });
});
```

The user gets a response in milliseconds. The background jobs run independently.

### Processing Jobs

Workers pick up jobs from the queue and process them:

```js
// src/queues/workers/emailWorker.js
const { Worker } = require('bullmq');
const { sendOrderConfirmation } = require('../../modules/email/email.service');

const worker = new Worker('orders', async (job) => {
  if (job.name === 'send-confirmation-email') {
    await sendOrderConfirmation(job.data.userEmail, job.data.orderId);
  }

  if (job.name === 'notify-warehouse') {
    await notifyWarehouse(job.data);
  }
}, {
  connection: { host: '127.0.0.1', port: 6379 },
  concurrency: 5,
});

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err.message);
});
```

The `concurrency` option lets the worker process up to 5 jobs at the same time.

## Retry and Error Handling

BullMQ retries failed jobs automatically:

```js
await orderQueue.add('send-confirmation-email', data, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 1000,
  },
});
```

If the email service is down, BullMQ retries up to 3 times with exponential backoff: 1 second, 2 seconds, 4 seconds.

## RabbitMQ for More Complex Scenarios

RabbitMQ is a full message broker that supports more complex routing patterns:

```js
// Publishing a message
const amqp = require('amqplib');

const publishEvent = async (exchange, routingKey, message) => {
  const connection = await amqp.connect(process.env.RABBITMQ_URL);
  const channel = await connection.createChannel();

  await channel.assertExchange(exchange, 'topic', { durable: true });
  channel.publish(exchange, routingKey, Buffer.from(JSON.stringify(message)));

  await channel.close();
  await connection.close();
};

// Consuming messages
const consumeEvents = async (queue, exchange, pattern, handler) => {
  const connection = await amqp.connect(process.env.RABBITMQ_URL);
  const channel = await connection.createChannel();

  await channel.assertQueue(queue, { durable: true });
  await channel.bindQueue(queue, exchange, pattern);

  channel.consume(queue, async (msg) => {
    try {
      await handler(JSON.parse(msg.content.toString()));
      channel.ack(msg);
    } catch (err) {
      channel.nack(msg, false, true); // Requeue on failure
    }
  });
};
```

RabbitMQ supports topic routing, dead letter queues, and message persistence. It is more powerful but also more complex than Redis-based queues.

Start with BullMQ for simplicity. Move to RabbitMQ when you need its advanced features.
