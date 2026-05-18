# Vercel Deployment

Vercel is built for frontend frameworks, but you can run Express on it too. It works by converting your Express app into serverless functions. This is great for small APIs but has some important limitations.

## How It Works

Vercel takes each route handler and deploys it as a separate serverless function. When a request comes in, Vercel spins up the function, handles the request, and shuts it down. You pay per invocation, not for a running server.

## Setup

### Project Structure

Vercel expects a specific structure. I need an `api/` directory at the project root:

```text
my-project/
  api/
    index.js
  package.json
  vercel.json
```

### The Entry Point

```js
// api/index.js
const app = require('../src/app');

// Vercel needs a default export
module.exports = app;
```

### vercel.json Configuration

```json
{
  "version": 2,
  "builds": [
    {
      "src": "api/index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/index.js"
    }
  ]
}
```

This tells Vercel to route all `/api/*` requests to the Express app.

## Deploying

### Option 1: Vercel CLI

```bash
npm install -g vercel
vercel
```

Follow the prompts. Vercel reads the config and deploys.

### Option 2: GitHub Integration

1. Push your code to GitHub
2. Go to [vercel.com](https://vercel.com) and import the repo
3. Vercel detects the `vercel.json` and deploys automatically
4. Every push to `main` triggers a new deploy

Set environment variables in the Vercel dashboard under "Settings" then "Environment Variables".

## Limitations

Serverless Express is not the same as running a dedicated server. Here are the gotchas:

### No Persistent Connections

Each request might hit a different function instance. WebSocket connections do not work well. Long-running processes get killed after 10 seconds on the free tier, 60 seconds on Pro.

### Cold Starts

If a function has not been called recently, Vercel needs to spin it up. This adds latency. The first request can take 1 to 3 seconds. Subsequent requests are fast until the function spins down again.

### No File System Writes

Serverless functions have a read-only file system. You cannot use `multer.diskStorage` for uploads. Use S3 or another cloud storage service instead:

```js
// Use memory storage instead of disk storage
const multer = require('multer');
const upload = multer({ storage: multer.memoryStorage() });
```

### Body Size Limits

Vercel limits request bodies to 4.5 MB on the free tier. For larger payloads (like file uploads), you need to use a direct upload to S3 from the client.

### Database Connections

Each serverless function invocation creates a new database connection unless you use connection pooling. With MongoDB, enable connection pooling:

```js
mongoose.connect(uri, {
  maxPoolSize: 10,
  minPoolSize: 2,
});
```

Or use a serverless-friendly database driver like Prisma with connection pooling.

## When to Use Vercel

Vercel is great for:

- Small APIs with low traffic
- Personal projects and portfolios
- APIs that serve frontend apps also on Vercel

It is not great for:

- Long-running requests
- WebSocket-heavy apps
- Apps that need file system access
- High-traffic production APIs

Pick the right tool for the job. Vercel is easy and fast to deploy, but know the tradeoffs before you commit.
