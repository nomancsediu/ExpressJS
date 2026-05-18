# Core Modules

Node.js comes with a set of built-in modules that do not require any installation. You just `require` them and start using them. These are the building blocks that Express relies on, so knowing them well pays off. Let me walk you through the ones that matter most.

## The `fs` Module (File System)

The `fs` module lets you read, write, and manipulate files and directories. Express uses it for serving static files, reading templates, and writing logs.

```javascript
const fs = require('fs');

// Read a file asynchronously
fs.readFile('config.json', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  console.log(data);
});

// Write a file
fs.writeFile('output.txt', 'Hello, file system!', (err) => {
  if (err) throw err;
  console.log('File written successfully');
});

// Check if a file exists (async version)
fs.promises.access('config.json')
  .then(() => console.log('File exists'))
  .catch(() => console.log('File does not exist'));
```

## The `path` Module

The `path` module handles file path operations in a cross-platform way. This is essential because Windows uses backslashes and Unix uses forward slashes.

```javascript
const path = require('path');

// Join path segments
const fullPath = path.join(__dirname, 'public', 'images', 'logo.png');
console.log(fullPath);
// On Mac/Linux: /home/user/project/public/images/logo.png
// On Windows:   C:\Users\project\public\images\logo.png

// Get file extension
console.log(path.extname('app.js'));       // .js
console.log(path.extname('archive.tar.gz')); // .gz

// Get the base name
console.log(path.basename('/users/docs/file.txt')); // file.txt
console.log(path.basename('/users/docs/file.txt', '.txt')); // file

// Parse a path into its components
const parsed = path.parse('/home/user/project/app.js');
console.log(parsed);
// { root: '/', dir: '/home/user/project', base: 'app.js', ext: '.js', name: 'app' }
```

## The `os` Module

The `os` module gives you information about the operating system. Useful for system-level configuration and monitoring.

```javascript
const os = require('os');

console.log('Platform:', os.platform());       // linux, darwin, win32
console.log('CPU cores:', os.cpus().length);   // Number of CPU cores
console.log('Free memory:', os.freemem());     // Bytes of free memory
console.log('Total memory:', os.totalmem());   // Bytes of total memory
console.log('Home directory:', os.homedir());  // User's home directory
console.log('Uptime:', os.uptime(), 'seconds');
```

## The `http` Module

This is the module Express is built on. It creates web servers and handles HTTP requests and responses. You will see it in detail in a later chapter, but here is a preview:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from the http module');
});

server.listen(3000, () => {
  console.log('Server listening on port 3000');
});
```

## The `url` Module

The `url` module parses and formats URLs. Express handles a lot of this for you, but understanding it helps with debugging.

```javascript
const url = require('url');

const myUrl = new URL('https://example.com/api/users?page=2&sort=name');

console.log('Hostname:', myUrl.hostname);   // example.com
console.log('Pathname:', myUrl.pathname);   // /api/users
console.log('Search params:', myUrl.searchParams.get('page')); // 2
console.log('Full search:', myUrl.search);  // ?page=2&sort=name
```

## The `events` Module

Node.js is event-driven, and the `events` module provides the EventEmitter class. Many Node.js objects are event emitters.

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const emitter = new MyEmitter();

// Listen for an event
emitter.on('greet', (name) => {
  console.log(`Hello, ${name}!`);
});

// Emit the event
emitter.emit('greet', 'World');
emitter.emit('greet', 'Express Learner');
```

## The `util` Module

The `util` module provides utility functions. The most useful one I have found is `promisify`, which converts callback-based functions into Promise-based ones.

```javascript
const util = require('util');
const fs = require('fs');

// Convert callback-style fs.readFile to Promise-style
const readFile = util.promisify(fs.readFile);

readFile('config.json', 'utf8')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Or with async/await
async function readConfig() {
  try {
    const data = await readFile('config.json', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Failed to read config:', err);
  }
}
```

Nowadays, `fs.promises` provides this natively, but `util.promisify` works with any callback-style function, not just `fs`.

These core modules are the foundation everything else is built on. When you use Express, you are using these modules under the hood whether you realize it or not. Knowing them gives you a deeper understanding of what your Express app is actually doing.
