# Memory Leaks

Memory leaks are sneaky. Your app works fine for hours or even days, then it gets slower and slower until it crashes with an out-of-memory error. I have dealt with several memory leaks in production, and each time the root cause surprised me.

## What Is a Memory Leak?

A memory leak happens when your app allocates memory but never releases it. Over time, the used memory grows until the process runs out. In Node.js, this means garbage collection cannot free objects because something still holds a reference to them.

```javascript
// Classic leak: global array that never stops growing
const requestLog = [];

app.use((req, res, next) => {
  requestLog.push({ url: req.url, time: Date.now() });
  next();
});
```

Every request adds an entry. Nothing ever removes entries. After a million requests, this array holds a million objects in memory.

## Common Leak Patterns

**1. Uncleared intervals and timeouts:**

```javascript
// Leak: interval keeps running and referencing the user object
function trackActivity(user) {
  setInterval(() => {
    console.log(user.name, 'last active:', Date.now());
  }, 10000);
}

// Fix: store the reference and clear it
function trackActivity(user) {
  const id = setInterval(() => {
    console.log(user.name, 'last active:', Date.now());
  }, 10000);

  return () => clearInterval(id);
}
```

**2. Closures capturing large objects:**

```javascript
// Leak: closure keeps reference to entire largeData
function processData(largeData) {
  const summary = largeData.items.length;

  return function getResult() {
    // Only needs summary, but largeData stays in memory
    return summary;
  };
}

// Fix: only capture what you need
function processData(largeData) {
  const summary = largeData.items.length;
  // largeData can now be garbage collected

  return function getResult() {
    return summary;
  };
}
```

**3. Event listeners that are never removed:**

```javascript
// Leak: listener added every time but never removed
app.get('/subscribe', (req, res) => {
  emitter.on('update', (data) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  });
  res.writeHead(200, { 'Content-Type': 'text/event-stream' });
});
```

## Detecting Leaks

I use three approaches to find leaks:

**1. Monitor memory usage over time:**

```javascript
setInterval(() => {
  const used = process.memoryUsage();
  console.log(`RSS: ${(used.rss / 1024 / 1024).toFixed(1)}MB, ` +
    `Heap: ${(used.heapUsed / 1024 / 1024).toFixed(1)}MB / ` +
    `${(used.heapTotal / 1024 / 1024).toFixed(1)}MB`);
}, 60000);
```

If heap usage keeps growing and never drops, you have a leak.

**2. Heap snapshots:**

```javascript
const v8 = require('v8');
const fs = require('fs');

// Take snapshot at intervals
app.get('/debug/heap-snapshot', (req, res) => {
  const snapshot = v8.writeHeapSnapshot();
  res.json({ file: snapshot });
});
```

Load two snapshots in Chrome DevTools and use the "Comparison" view to see which objects were created between them.

**3. Automated leak detection with node-memwatch:**

```bash
npm install @airbnb/node-memwatch
```

```javascript
const memwatch = require('@airbnb/node-memwatch');

memwatch.on('leak', (info) => {
  console.error('Memory leak detected:', info);
});
```

Finding and fixing memory leaks takes patience, but the process is always the same: reproduce the leak, take heap snapshots, compare them, and look for objects that should have been collected but were not.
