# File Downloads and Streaming

Sending files from your server to the client comes up a lot. Whether it is letting users download reports, serving images, or streaming large videos, Express has several methods to handle it.

## res.download()

This tells the browser to download a file instead of displaying it:

```js
app.get('/report', (req, res) => {
  res.download('/files/report.pdf', 'monthly-report.pdf', (err) => {
    if (err) {
      console.error('Download failed:', err);
      res.status(500).send('Download failed');
    }
  });
});
```

The second argument is the filename the browser suggests. The third is a callback that runs when the download completes or fails.

## res.sendFile()

Sends a file as the response body. The browser displays it if it can, otherwise it downloads:

```js
const path = require('path');

app.get('/image', (req, res) => {
  const filePath = path.join(__dirname, 'public', 'photo.jpg');
  res.sendFile(filePath, (err) => {
    if (err) {
      console.error(err);
      res.status(500).send('File not found');
    }
  });
});
```

`res.sendFile()` needs an absolute path. Using `path.join()` with `__dirname` is the safest way to build one.

You can also set headers like this:

```js
res.sendFile(filePath, {
  headers: {
    'Content-Disposition': 'inline',
    'Cache-Control': 'public, max-age=86400',
  },
});
```

## Streaming Responses

For large files, reading the entire file into memory before sending is wasteful. Streaming sends data in chunks:

```js
const fs = require('fs');

app.get('/video', (req, res) => {
  const filePath = path.join(__dirname, 'videos', 'demo.mp4');
  const stat = fs.statSync(filePath);

  res.writeHead(200, {
    'Content-Type': 'video/mp4',
    'Content-Length': stat.size,
  });

  const stream = fs.createReadStream(filePath);
  stream.pipe(res);
});
```

`pipe()` takes the readable stream and pipes it directly to the response. The file never loads entirely into memory.

## Range Requests

Video players and download managers request specific byte ranges. Supporting this lets users seek in videos:

```js
app.get('/video', (req, res) => {
  const filePath = path.join(__dirname, 'videos', 'demo.mp4');
  const stat = fs.statSync(filePath);
  const range = req.headers.range;

  if (!range) {
    res.writeHead(200, {
      'Content-Length': stat.size,
      'Content-Type': 'video/mp4',
    });
    fs.createReadStream(filePath).pipe(res);
    return;
  }

  const parts = range.replace(/bytes=/, '').split('-');
  const start = parseInt(parts[0], 10);
  const end = parts[1] ? parseInt(parts[1], 10) : stat.size - 1;
  const chunkSize = end - start + 1;

  res.writeHead(206, {
    'Content-Range': `bytes ${start}-${end}/${stat.size}`,
    'Accept-Ranges': 'bytes',
    'Content-Length': chunkSize,
    'Content-Type': 'video/mp4',
  });

  fs.createReadStream(filePath, { start, end }).pipe(res);
});
```

The 206 Partial Content status tells the browser this is just a portion of the file. This is how video players work when you skip ahead.

## Quick Reference

| Method | Use Case |
|---|---|
| `res.download()` | Force browser to download |
| `res.sendFile()` | Send a file for display or download |
| `stream.pipe(res)` | Stream large files efficiently |
| Range requests | Support video seeking and resume |

Streaming was intimidating at first, but it is really just reading and writing in chunks. Start with `res.download()` and `res.sendFile()`, then move to streaming when you need to handle large files.
