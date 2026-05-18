# Event-Driven Architecture

When one part of your app does something, other parts might need to react. A user registers, so you send a welcome email. An order is placed, so you update inventory and notify the warehouse. Wiring these together directly creates tangled code. Events decouple them.

## Node.js EventEmitter

Node.js has a built-in EventEmitter. It is simple and works great for in-process events.

```js
const EventEmitter = require('events');

class AppEvents extends EventEmitter {}
const emitter = new AppEvents();

// Listen for an event
emitter.on('user.registered', (user) => {
  console.log(`Welcome email sent to ${user.email}`);
});

// Emit an event
emitter.emit('user.registered', { id: 1, name: 'Alice', email: 'alice@example.com' });
```

The emitter does not know or care who is listening. The listener does not know or care who emitted the event. They are completely decoupled.

## Building an Event System for Express

I create a central event bus that any module can use:

```js
// src/events/index.js
const EventEmitter = require('events');

const emitter = new EventEmitter();

// Increase max listeners to avoid warnings with many listeners
emitter.setMaxListeners(20);

module.exports = emitter;
```

## Emitting Events from Controllers

In the user controller, I emit an event after registration instead of doing everything inline:

```js
const emitter = require('../../events');

exports.register = asyncHandler(async (req, res) => {
  const user = await User.create(req.body);

  // Emit event instead of doing everything here
  emitter.emit('user.registered', user);

  const accessToken = signToken(user._id);
  res.status(201).json({ success: true, data: { user, accessToken } });
});
```

## Listening for Events in Other Modules

Each module registers its own listeners:

```js
// src/modules/email/email.listeners.js
const emitter = require('../../events');
const { sendWelcomeEmail } = require('./email.service');

emitter.on('user.registered', async (user) => {
  try {
    await sendWelcomeEmail(user.email, user.name);
  } catch (err) {
    console.error('Failed to send welcome email:', err.message);
  }
});
```

```js
// src/modules/analytics/analytics.listeners.js
const emitter = require('../../events');

emitter.on('user.registered', (user) => {
  // Track signups in analytics
  console.log(`New signup tracked: ${user.id}`);
});

emitter.on('order.created', (order) => {
  // Track revenue
  console.log(`Revenue tracked: $${order.totalAmount}`);
});
```

## Naming Events

Good event names make the system understandable. I follow a `entity.action` pattern:

```js
emitter.emit('user.registered', user);
emitter.emit('user.password_reset', user);
emitter.emit('order.created', order);
emitter.emit('order.shipped', order);
emitter.emit('order.cancelled', order);
emitter.emit('product.low_stock', product);
emitter.emit('payment.completed', payment);
emitter.emit('payment.failed', payment);
```

Past tense for completed actions. This makes it clear that the event happened, not that it is about to happen.

## Error Handling in Listeners

If a listener throws an error, it can crash the process. I wrap listeners with error handling:

```js
const safeListener = (fn) => async (...args) => {
  try {
    await fn(...args);
  } catch (err) {
    console.error('Event listener error:', err);
  }
};

emitter.on('user.registered', safeListener(async (user) => {
  await sendWelcomeEmail(user.email, user.name);
}));
```

This prevents one failing listener from affecting others.

## When to Use Events

Use events when:

- Multiple things need to happen after one action
- You want to add new behaviors without modifying existing code
- Modules should not know about each other

Do not use events for:

- Request/response flows (use function calls)
- Critical operations where the result matters immediately
- Operations that need to block the current request

Events are a tool for decoupling. They keep your code clean as it grows. But they add indirection, so use them where the benefit outweighs the cost.
