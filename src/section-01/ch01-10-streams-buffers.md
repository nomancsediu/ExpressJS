# Streams and Buffers

Streams and buffers are two concepts that seemed intimidating when I first encountered them. But they are actually straightforward once you understand the basic idea. And they are important for Express because they power file uploads, large responses, and real-time data.

## What Are Buffers?

A buffer is a fixed-size chunk of memory that stores raw binary data. JavaScript does not have a native byte type, so Node.js provides the Buffer class.

```javascript
// Create a buffer from a string
const buf1 = Buffer.from('Hello, Buffer!');
console.log(buf1);            // <Buffer 48 65 6c 6c 6f 2c 20 42 75 66 66 65 72 21>
console.log(buf1.toString()); // Hello, Buffer!
console.log(buf1.length);     // 14 (bytes)

// Create a buffer of a specific size
const buf2 = Buffer.alloc(10); // 10 bytes, all zeros
console.log(buf2);            // <Buffer 00 00 00 00 00 00 00 00 00 00>

// Write to a buffer
buf2.write('Hi');
console.log(buf2.toString()); // Hi

// Create an uninitialized buffer (faster, but contains random data)
const buf3 = Buffer.allocUnsafe(10);
```

Buffers are useful when you need to work with binary data like images, videos, or network protocols. Most of the time in Express, you deal with buffers indirectly through streams.

## What Are Streams?

A stream is a way to handle data piece by piece instead of loading everything into memory at once. Think of it like watching a video online. You do not download the entire video before you start watching. The data flows to you in chunks. That is a stream.

There are four types of streams in Node.js:

- **Readable** - Data you can read from (like reading a file)
- **Writable** - Data you can write to (like writing to a file)
- **Duplex** - Both readable and writable (like a TCP socket)
- **Transform** - Modify data as it passes through (like compression)

## Readable Streams

```javascript
const fs = require('fs');

// Create a readable stream
const readStream = fs.createReadStream('big-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024 // 64KB chunks
});

readStream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes of data`);
});

readStream.on('end', () => {
  console.log('Finished reading the file');
});

readStream.on('error', (err) => {
  console.error('Stream error:', err);
});
```

Instead of reading a 1GB file all at once (which would use 1GB of RAM), streams read it in small chunks. Your memory usage stays low regardless of the file size.

## Writable Streams

```javascript
const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt');

writeStream.write('First line\n');
writeStream.write('Second line\n');
writeStream.end('Final line\n');

writeStream.on('finish', () => {
  console.log('All data written');
});

writeStream.on('error', (err) => {
  console.error('Write error:', err);
});
```

## Piping Streams

Piping connects a readable stream directly to a writable stream. Data flows from one to the other automatically. This is incredibly useful.

```javascript
const fs = require('fs');

// Copy a file using pipe
const readStream = fs.createReadStream('source.txt');
const writeStream = fs.createWriteStream('destination.txt');

readStream.pipe(writeStream);

writeStream.on('finish', () => {
  console.log('File copied successfully');
});
```

In Express, piping is how you send files to clients efficiently:

```javascript
app.get('/download', (req, res) => {
  const fileStream = fs.createReadStream('large-file.pdf');
  res.setHeader('Content-Type', 'application/pdf');
  fileStream.pipe(res);
});
```

The file streams directly from disk to the network. Your server never holds the entire file in memory.

## Transform Streams

Transform streams modify data as it passes through. A common example is compression:

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Compress a file on the fly
fs.createReadStream('access.log')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('access.log.gz'))
  .on('finish', () => {
    console.log('File compressed');
  });
```

You can also create custom transform streams:

```javascript
const { Transform } = require('stream');

const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString().toUpperCase());
  }
});

process.stdin.pipe(upperCase).pipe(process.stdout);
```

## Backpressure

Backpressure is what happens when a readable stream produces data faster than a writable stream can consume it. Without handling backpressure, data piles up in memory and your app can crash.

The good news is that `pipe()` handles backpressure automatically. If the writable stream cannot keep up, `pipe()` pauses the readable stream until the writable stream catches up.

If you are manually connecting streams without `pipe()`, you need to handle backpressure yourself using the `pause()`, `resume()`, and `drain` events:

```javascript
const readStream = fs.createReadStream('huge-file.txt');
const writeStream = fs.createWriteStream('copy.txt');

readStream.on('data', (chunk) => {
  const canContinue = writeStream.write(chunk);
  if (!canContinue) {
    readStream.pause();
  }
});

writeStream.on('drain', () => {
  readStream.resume();
});

readStream.on('end', () => {
  writeStream.end();
});
```

This is exactly what `pipe()` does for you, which is why I almost always use `pipe()` instead of handling it manually.

## Streams in Express

Here are common scenarios where streams matter in Express:

**Serving large files:**
```javascript
app.get('/video', (req, res) => {
  const stream = fs.createReadStream('video.mp4');
  stream.pipe(res);
});
```

**File uploads:**
```javascript
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({ file: req.file });
});
```

**Server-sent events:**
```javascript
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: new Date() })}\n\n`);
  }, 1000);

  req.on('close', () => clearInterval(interval));
});
```

Streams are one of Node.js's most powerful features. Once you understand them, you will start seeing opportunities to use them everywhere in your Express applications.
