# Clustering

Node.js runs on a single thread by default. That means on a 4-core machine, your Express app only uses one core. Clustering lets you create multiple processes that share the same port, utilizing all CPU cores.

## The Problem

```javascript
// This only uses one CPU core
const express = require('express');
const app = express();

app.get('/heavy', (req, res) => {
  // CPU-intensive work blocks the single thread
  let result = 0;
  for (let i = 0; i < 1e7; i++) result += Math.sqrt(i);
  res.json({ result });
});

app.listen(3000);
```

With clustering, I can run one process per core and multiply throughput.

## Using the Cluster Module

Node.js has a built-in `cluster` module:

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Primary process running. Forking ${numCPUs} workers.`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  const express = require('express');
  const app = express();

  app.get('/heavy', (req, res) => {
    let result = 0;
    for (let i = 0; i < 1e7; i++) result += Math.sqrt(i);
    res.json({ result, pid: process.pid });
  });

  app.listen(3000);
  console.log(`Worker ${process.pid} started`);
}
```

The primary process forks workers. Each worker runs the Express app. All workers share port 3000 because the primary process distributes incoming connections.

## Master and Worker Roles

- **Master/Primary**: Manages workers, forks new ones, handles restarts
- **Workers**: Run the actual Express app, handle requests

The master does not handle HTTP traffic. It just coordinates workers.

## Graceful Restart

When you need to deploy new code without downtime, you restart workers one at a time:

```javascript
// In the primary process
function gracefulRestart() {
  const workers = Object.values(cluster.workers);
  workers.forEach((worker, index) => {
    setTimeout(() => {
      console.log(`Restarting worker ${worker.process.pid}`);
      worker.disconnect(); // Let it finish current requests
      setTimeout(() => {
        if (worker.isConnected()) worker.kill();
        cluster.fork();
      }, 5000);
    }, index * 2000); // Stagger restarts
  });
}
```

This ensures some workers are always running while others restart.

## Important Considerations

Clustering is powerful but has trade-offs:

- **Memory**: Each worker has its own memory. 4 workers means 4x memory usage.
- **State**: Workers do not share state. In-memory caches and sessions need to move to Redis.
- **Sticky sessions**: If you use WebSockets, you need sticky sessions so a client reconnects to the same worker.

I use clustering as a baseline for production Express apps. It is a simple way to multiply performance without changing any application code. For more advanced management, I reach for PM2, which we will cover next.
