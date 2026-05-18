# MongoDB Connection

Before I can query anything, I need to connect my Express app to MongoDB. Here is how I set up a reliable connection with proper error handling.

## mongoose.connect

The basic connection is simple:

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/myapp');
```

But I never use the basic version in a real project. I always include the database name and proper options.

## Connection Options

Mongoose has changed its default options over time. Here is what I use:

```js
mongoose.connect('mongodb://localhost:27017/myapp', {
  // These are mostly default now in Mongoose 7+, but I include them for clarity
})
.then(() => console.log('Connected to MongoDB'))
.catch((err) => console.error('MongoDB connection error:', err));
```

For a production MongoDB Atlas connection, the URI looks different:

```js
const MONGO_URI = 'mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/myapp?retryWrites=true&w=majority';

mongoose.connect(MONGO_URI)
  .then(() => console.log('Connected to MongoDB Atlas'))
  .catch((err) => console.error('Connection failed:', err));
```

Never hardcode credentials. I always use environment variables:

```js
const MONGO_URI = process.env.MONGO_URI;
```

## Error Handling

Connection errors can happen at startup or later if the database goes down. I handle both:

```js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
    console.log(`MongoDB connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`MongoDB connection error: ${error.message}`);
    process.exit(1); // Crash and let the process manager restart
  }
};

module.exports = connectDB;
```

I also listen for connection events after the initial connection:

```js
mongoose.connection.on('disconnected', () => {
  console.warn('MongoDB disconnected');
});

mongoose.connection.on('error', (err) => {
  console.error('MongoDB error:', err);
});

mongoose.connection.on('reconnected', () => {
  console.log('MongoDB reconnected');
});
```

## Connection Pooling

Mongoose manages a connection pool under the hood. By default, it creates a pool of 100 connections. This means multiple queries can run at the same time without opening new connections each time.

```js
mongoose.connect(MONGO_URI, {
  serverSelectionTimeoutMS: 5000, // Give up after 5 seconds
  socketTimeoutMS: 45000,         // Close idle sockets after 45 seconds
});
```

For a small app, the defaults are fine. For high-traffic apps, I might tune the pool size based on my server resources.

## Where to Connect

I call `connectDB` at the top of my app entry file, before starting the server:

```js
const express = require('express');
const connectDB = require('./config/db');

const app = express();

// Connect to database first
connectDB().then(() => {
  app.listen(process.env.PORT || 3000, () => {
    console.log('Server running');
  });
});
```

This ensures the database is connected before any requests come in. No database, no server.
