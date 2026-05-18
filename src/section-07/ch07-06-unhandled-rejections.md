# Unhandled Rejections and Uncaught Exceptions

Even with the best error middleware, some errors slip past Express entirely. Promises get rejected with no catch. Synchronous code throws where nobody is listening. These are the errors that crash your process at 3 AM and leave you staring at a blank screen.

## What Are Unhandled Rejections?

When a Promise is rejected and there is no `.catch()` handler or try/catch around it, that is an unhandled rejection. In modern Node.js, unhandled rejections terminate the process by default.

```js
// This promise rejection has no handler
fetch('https://api.example.com/data')
  .then(res => res.json())
  .then(data => console.log(data));
  // No .catch()! If the fetch fails, it is an unhandled rejection
```

In Express, this happens when you forget to catch async errors in route handlers.

## What Are Uncaught Exceptions?

When a synchronous error is thrown and nothing catches it, that is an uncaught exception. These are usually programming bugs.

```js
// This variable is not defined anywhere
const result = performCalculation(undefinedVar);
// ReferenceError: undefinedVar is not defined
```

## Setting Up Handlers

I add these handlers at the very top of my app entry file:

```js
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  // Log the error, maybe send an alert
  // Then decide: crash or continue?
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  // Log the error
  // Then crash gracefully
  process.exit(1);
});
```

## When to Restart

Here is the important lesson I learned. After an uncaught exception, your app is in an unpredictable state. Memory might be leaked. Connections might be broken. State might be corrupted. The safest thing is to crash and let your process manager restart the app.

```js
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);

  // Give time for logs to flush, then exit
  process.exit(1);
});
```

For unhandled rejections, the debate is similar. Some people let the app continue. I prefer to crash and restart because the app might be in a bad state.

## Using a Process Manager

I never run production apps without a process manager. PM2 is my go-to:

```bash
# Start with PM2
pm2 start app.js --name "my-api"

# PM2 automatically restarts when the process crashes
pm2 logs my-api
```

With PM2 (or Docker, or Kubernetes), when my app crashes due to an unhandled error, it comes right back up within seconds. The combination of proper handlers and a process manager gives me confidence that my app stays alive even when the unexpected happens.
