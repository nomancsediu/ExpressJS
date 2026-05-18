# File System Operations

Working with files is something every Express app needs to do. Whether it is reading configuration files, writing logs, serving static assets, or handling uploads, the `fs` module is your tool. Let me show you the operations that come up most often.

## Reading Files

There are several ways to read files in Node.js. Let me show you the ones I use.

**Callback style (older but still common):**

```javascript
const fs = require('fs');

fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Read error:', err);
    return;
  }
  console.log(data);
});
```

**Promise style (my preferred way):**

```javascript
const fs = require('fs').promises;

async function readData() {
  try {
    const data = await fs.readFile('data.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Read error:', err);
  }
}

readData();
```

**Synchronous style (avoid in servers, but handy for startup code):**

```javascript
const fs = require('fs');

try {
  const data = fs.readFileSync('config.json', 'utf8');
  const config = JSON.parse(data);
  console.log('Config loaded:', config);
} catch (err) {
  console.error('Failed to load config:', err);
  process.exit(1);
}
```

I sometimes use `readFileSync` once at startup to load configuration. After that, I switch to async reads for everything else.

## Writing Files

```javascript
const fs = require('fs').promises;

async function writeData() {
  try {
    // Overwrite or create a file
    await fs.writeFile('output.txt', 'Hello, World!');
    console.log('File written');

    // Append to an existing file (great for logs)
    await fs.appendFile('log.txt', `${new Date().toISOString()} - Request received\n`);
    console.log('Log entry added');
  } catch (err) {
    console.error('Write error:', err);
  }
}

writeData();
```

## Working with Directories

```javascript
const fs = require('fs').promises;

async function workWithDirectories() {
  try {
    // Create a directory
    await fs.mkdir('uploads', { recursive: true });
    console.log('Directory created');

    // Read directory contents
    const files = await fs.readdir('./src');
    console.log('Files in src:', files);

    // Read with file types
    const entries = await fs.readdir('./src', { withFileTypes: true });
    entries.forEach(entry => {
      const type = entry.isDirectory() ? 'DIR' : 'FILE';
      console.log(`[${type}] ${entry.name}`);
    });

    // Remove an empty directory
    await fs.rmdir('empty-folder');

    // Remove a directory and everything inside
    await fs.rm('old-project', { recursive: true, force: true });
  } catch (err) {
    console.error('Directory error:', err);
  }
}

workWithDirectories();
```

## Getting File Information

```javascript
const fs = require('fs').promises;

async function getFileInfo() {
  try {
    const stats = await fs.stat('package.json');

    console.log('Is file?', stats.isFile());        // true
    console.log('Is directory?', stats.isDirectory()); // false
    console.log('Size:', stats.size, 'bytes');
    console.log('Created:', stats.birthtime);
    console.log('Modified:', stats.mtime);
    console.log('Permissions:', stats.mode.toString(8));

    // Check if something exists
    try {
      await fs.access('config.json');
      console.log('config.json exists and is readable');
    } catch {
      console.log('config.json does not exist or is not readable');
    }
  } catch (err) {
    console.error('Stat error:', err);
  }
}

getFileInfo();
```

## Watching Files

The `fs.watch` function lets you react to file changes. This is how tools like nodemon work under the hood.

```javascript
const fs = require('fs');

const watcher = fs.watch('data.txt', (eventType, filename) => {
  console.log(`Event: ${eventType} on ${filename}`);
});

// Stop watching after 30 seconds
setTimeout(() => {
  watcher.close();
  console.log('Stopped watching');
}, 30000);
```

For more reliable file watching in Express projects, I recommend using the `chokidar` package instead of the built-in `fs.watch`. The built-in version has known issues on some platforms:

```bash
npm install chokidar
```

```javascript
const chokidar = require('chokidar');

chokidar.watch('./src').on('all', (event, path) => {
  console.log(event, path);
});
```

## A Practical Example: Simple Logger

Here is a small utility I use in projects to write logs to a file:

```javascript
const fs = require('fs').promises;
const path = require('path');

class FileLogger {
  constructor(filename = 'app.log') {
    this.logPath = path.join(__dirname, '..', 'logs', filename);
  }

  async write(level, message) {
    const timestamp = new Date().toISOString();
    const entry = `[${timestamp}] ${level.toUpperCase()}: ${message}\n`;
    try {
      await fs.appendFile(this.logPath, entry);
    } catch (err) {
      console.error('Failed to write log:', err);
    }
  }

  async info(message) { await this.write('info', message); }
  async error(message) { await this.write('error', message); }
  async warn(message) { await this.write('warn', message); }
}

module.exports = new FileLogger();
```

The `fs` module is straightforward once you get the hang of it. The key rule is: always use the async versions in your Express route handlers. Use sync versions only at startup or in scripts.
