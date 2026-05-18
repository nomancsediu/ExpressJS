# Async JavaScript

If there is one topic that confuses beginners more than anything else in Node.js, it is asynchronous programming. I struggled with it for a long time. The good news is that once it clicks, everything else falls into place. Let me walk you through the evolution of async patterns in JavaScript.

## Why Async Matters in Node.js

Node.js is single-threaded and non-blocking. That means when your code needs to wait for something like a database query, a file read, or an HTTP request, it does not freeze. Instead, it registers a callback and moves on. When the operation completes, the callback runs.

Almost everything you do in Express is asynchronous: reading request bodies, querying databases, writing responses, sending emails. You need to be comfortable with async patterns.

## Callbacks: The Original Way

A callback is a function passed as an argument that gets called when an async operation finishes.

```javascript
const fs = require('fs');

// Read a file using a callback
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Something went wrong:', err);
    return;
  }
  console.log('File contents:', data);
});

console.log('This runs first, before the file is read');
```

The convention in Node.js is "error-first callbacks." The first argument is always an error (or null if everything went fine), and the second argument is the result.

## Callback Hell

Callbacks work fine for simple cases. The problem comes when you need to do multiple async operations in sequence:

```javascript
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handleError(err);
    getOrderItems(orders[0].id, (err, items) => {
      if (err) return handleError(err);
      getItemDetails(items[0].id, (err, details) => {
        if (err) return handleError(err);
        // We are four levels deep and it keeps getting worse
        console.log(details);
      });
    });
  });
});
```

This pyramid shape is called "callback hell" or "the pyramid of doom." It is hard to read, hard to debug, and hard to add error handling to. I have seen code ten levels deep like this. It is not fun.

## Promises: A Better Way

Promises represent a value that will be available in the future. They can be in three states: pending, fulfilled, or rejected.

```javascript
// Creating a Promise
function readFilePromise(filepath) {
  return new Promise((resolve, reject) => {
    fs.readFile(filepath, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// Using a Promise
readFilePromise('data.txt')
  .then(data => {
    console.log('File contents:', data);
    return readFilePromise('another-file.txt');
  })
  .then(data => {
    console.log('Another file:', data);
  })
  .catch(err => {
    console.error('Error:', err);
  });
```

Chaining `.then()` calls is much flatter than nested callbacks. A single `.catch()` at the end handles errors from any step in the chain.

## async/await: The Cleanest Way

async/await is syntactic sugar over Promises. It makes async code look and behave like synchronous code, which is much easier to read and write.

```javascript
async function readMultipleFiles() {
  try {
    const data1 = await readFilePromise('data.txt');
    console.log('First file:', data1);

    const data2 = await readFilePromise('another-file.txt');
    console.log('Second file:', data2);
  } catch (err) {
    console.error('Something went wrong:', err);
  }
}

readMultipleFiles();
```

Look how clean that is. No nesting, no chaining. Just linear code that reads top to bottom. The `try/catch` block handles errors naturally.

With `fs.promises`, you do not even need to wrap callbacks yourself:

```javascript
const fs = require('fs').promises;

async function readConfig() {
  try {
    const data = await fs.readFile('config.json', 'utf8');
    const config = JSON.parse(data);
    console.log('Config loaded:', config);
  } catch (err) {
    console.error('Failed to load config:', err);
  }
}
```

## Promise.all: Running Things in Parallel

Sometimes you need to run multiple async operations that do not depend on each other. Instead of waiting for each one sequentially with `await`, you can run them all at once with `Promise.all`:

```javascript
async function loadAllData() {
  try {
    // These run in parallel
    const [users, posts, comments] = await Promise.all([
      db.query('SELECT * FROM users'),
      db.query('SELECT * FROM posts'),
      db.query('SELECT * FROM comments')
    ]);

    console.log(`Loaded ${users.length} users`);
    console.log(`Loaded ${posts.length} posts`);
    console.log(`Loaded ${comments.length} comments`);
  } catch (err) {
    console.error('Failed to load data:', err);
  }
}
```

`Promise.all` takes an array of Promises and resolves when all of them resolve. If any one rejects, the whole thing rejects. This can make your Express handlers significantly faster when you have independent database queries.

There is also `Promise.allSettled` which waits for all Promises regardless of whether they resolve or reject, giving you the result of each one.

In Express, you will use async/await almost exclusively. It is the standard pattern. But understanding callbacks and raw Promises is important because many older libraries still use them, and sometimes you need to convert between the patterns.
