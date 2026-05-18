# Architecture and the Event Loop

This is the chapter I wish someone had written when I started learning Node.js. The event loop is the heart of Node.js, and understanding it will change how you write code forever.

## Single-Threaded Does Not Mean Simple

Node.js runs your JavaScript code in a single thread. That sounds limiting at first. How can a single thread handle thousands of requests? The answer is the event loop.

Think of the event loop like a restaurant waiter. The waiter takes an order from table one, gives it to the kitchen, and while the food is cooking, the waiter takes an order from table two. The waiter does not stand around waiting for table one's food to be ready. That is exactly how Node.js works. It takes a request, starts an I/O operation, and moves on to the next request. When the I/O finishes, it comes back and handles the result.

## The Event Loop Phases

The event loop runs in a cycle, going through several phases on each iteration. Here are the main ones:

1. **Timers.** Executes callbacks from `setTimeout` and `setInterval`.
2. **Pending callbacks.** Executes I/O callbacks that were deferred to the next loop iteration.
3. **Idle, prepare.** Used internally by Node.js. You rarely interact with these.
4. **Poll.** Retrieves new I/O events and executes their callbacks. This is where most of your code runs.
5. **Check.** Executes callbacks from `setImmediate`.
6. **Close callbacks.** Executes close events, like when a socket closes.

Here is a simple example that shows the order:

```javascript
console.log('1: Synchronous');

setTimeout(() => {
  console.log('2: Timeout');
}, 0);

setImmediate(() => {
  console.log('3: Immediate');
});

Promise.resolve().then(() => {
  console.log('4: Promise');
});

process.nextTick(() => {
  console.log('5: Next tick');
});

console.log('6: Synchronous');
```

The output will be:

```
1: Synchronous
6: Synchronous
5: Next tick
4: Promise
2: Timeout
3: Immediate
```

Notice that `process.nextTick` and Promises run before timers and immediate callbacks. This is because microtasks like nextTick and Promises are processed after the current operation completes, before moving to the next event loop phase.

## Blocking vs Non-Blocking

This is where most beginners get into trouble. Blocking code stops the event loop. Non-blocking code lets the event loop continue.

**Blocking example:**

```javascript
// This blocks the event loop for the entire duration
const fs = require('fs');
const data = fs.readFileSync('/big-file.txt', 'utf8');
console.log(data);
console.log('This waits until the file is read');
```

**Non-blocking example:**

```javascript
// This lets the event loop continue
const fs = require('fs');
fs.readFile('/big-file.txt', 'utf8', (err, data) => {
  console.log(data);
});
console.log('This runs immediately, before the file is read');
```

In a server handling hundreds of requests, using blocking calls means every other request has to wait. That is why Node.js APIs provide asynchronous versions of almost everything.

## When You Must Block

Sometimes you need to do CPU-intensive work, like processing images or crunching numbers. In those cases, you should move that work off the main thread. You can use:

- **Worker threads** for CPU-heavy tasks
- **Child processes** to spawn separate processes
- **Breaking work into chunks** using `setImmediate` to yield back to the event loop

```javascript
const { Worker } = require('worker_threads');

function heavyComputation(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavy-task.js', {
      workerData: data
    });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
```

Understanding the event loop is not just academic. When your Express app slows down under load, the first thing I check is whether something is blocking the event loop. Nine times out of ten, that is the culprit.
