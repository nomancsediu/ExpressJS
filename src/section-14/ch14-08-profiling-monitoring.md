# Profiling and Monitoring

Profiling helps you understand exactly where your app spends its time and memory. I used to guess at bottlenecks. Now I profile first, then optimize. It saves a lot of wasted effort.

## Chrome DevTools

Node.js integrates with Chrome DevTools for profiling:

```bash
# Start your app with the inspect flag
node --inspect src/index.js
```

Then open Chrome and go to `chrome://inspect`. Click "Inspect" on your Node process.

You get:

- **CPU Profiler**: Record and analyze which functions consume the most CPU time
- **Memory Profiler**: Take heap snapshots and compare them to find memory leaks
- **Performance Tab**: Timeline view of function calls

To capture a CPU profile, click "Record", make some requests to your app, then stop recording. You will see a flame chart showing which functions took the most time.

## Clinic.js

Clinic.js is a specialized Node.js profiling tool with three modes:

```bash
npm install -g clinic
```

**Clinic Doctor**: Diagnoses performance issues

```bash
clinic doctor -- node src/index.js
```

Make some requests, then stop the server. Clinic generates an HTML report with recommendations like "Event loop is blocked" or "Possible garbage collection issue."

**Clinic Bubbleprof**: Visualizes async operations

```bash
clinic bubbleprof -- node src/index.js
```

This shows you the flow of async operations and where time is spent waiting.

**Clinic Flame**: Generates flame graphs

```bash
clinic flame -- node src/index.js
```

Flame graphs are incredibly useful. They show you at a glance which code paths are hot. Wide bars at the top are functions consuming the most CPU.

## Application Performance Monitoring (APM)

For production monitoring, I use APM tools that run continuously:

**New Relic:**

```bash
npm install newrelic
```

```javascript
// Add as the very first line in your entry file
require('newrelic');
```

**Datadog:**

```bash
npm install dd-trace
```

```javascript
const tracer = require('dd-trace').init();
```

These tools give you real-time dashboards showing response times, error rates, throughput, and slow queries. They alert you when something goes wrong.

## Custom Metrics

I also add custom metrics for business-specific monitoring:

```javascript
const responseTime = require('response-time');

app.use(responseTime((req, res, time) => {
  // Track response time per route
  const route = `${req.method} ${req.route?.path || req.path}`;

  metrics.histogram('http_response_time', time, {
    route,
    status: res.statusCode,
  });

  // Track error rate
  if (res.statusCode >= 500) {
    metrics.increment('http_errors', { route });
  }
}));
```

Custom metrics let you monitor what matters for your specific app, not just generic server health. Combined with APM dashboards and alerts, I always know how my app is performing in production.
