# File Download and Streaming

Serving files back to users is just as important as receiving them. Express gives you several ways to do this, from simple downloads to efficient streaming.

## Simple File Download

The easiest way to send a file is `res.download()`:

```js
app.get('/download/:filename', (req, res) => {
  const filePath = path.join(__dirname, 'files', req.params.filename);

  res.download(filePath, 'report.pdf', (err) => {
    if (err) {
      if (!res.headersSent) {
        res.status(404).send('File not found');
      }
    }
  });
});
```

`res.download()` sets the `Content-Disposition` header to `attachment`, which tells the browser to download the file instead of displaying it. The second argument is the filename the user sees.

## Sending Files for Display

If you want the browser to display the file inline (like showing a PDF or image), use `res.sendFile()`:

```js
app.get('/images/:name', (req, res) => {
  const filePath = path.join(__dirname, 'uploads', req.params.name);

  res.sendFile(filePath, (err) => {
    if (err) {
      res.status(404).send('File not found');
    }
  });
});
```

Always use absolute paths with `res.sendFile()`. Relative paths can cause issues depending on where Node is running from.

## Streaming Large Files

For large files like videos, loading the entire file into memory is a bad idea. Streaming sends the file in chunks:

```js
app.get('/video/:name', (req, res) => {
  const filePath = path.join(__dirname, 'videos', req.params.name);
  const stat = fs.statSync(filePath);
  const fileSize = stat.size;

  res.writeHead(200, {
    'Content-Length': fileSize,
    'Content-Type': 'video/mp4'
  });

  const readStream = fs.createReadStream(filePath);
  readStream.pipe(res);
});
```

The `pipe()` method connects the read stream directly to the response stream. Data flows from disk to the network without loading everything into RAM.

## Range Requests

Browsers request video segments using range headers. This is how video seeking works. You need to handle `Range` headers:

```js
app.get('/video/:name', (req, res) => {
  const filePath = path.join(__dirname, 'videos', req.params.name);
  const stat = fs.statSync(filePath);
  const fileSize = stat.size;
  const range = req.headers.range;

  if (range) {
    const parts = range.replace(/bytes=/, '').split('-');
    const start = parseInt(parts[0], 10);
    const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
    const chunkSize = end - start + 1;

    res.writeHead(206, {
      'Content-Range': `bytes ${start}-${end}/${fileSize}`,
      'Accept-Ranges': 'bytes',
      'Content-Length': chunkSize,
      'Content-Type': 'video/mp4'
    });

    fs.createReadStream(filePath, { start, end }).pipe(res);
  } else {
    res.writeHead(200, {
      'Content-Length': fileSize,
      'Content-Type': 'video/mp4'
    });
    fs.createReadStream(filePath).pipe(res);
  }
});
```

The `206 Partial Content` response tells the browser it is getting just a piece of the file. Without this, video players cannot seek to different positions.

## Security Note

Never pass user input directly to file paths without validation. Always sanitize the filename to prevent directory traversal attacks:

```js
const safeName = path.basename(req.params.filename);
const filePath = path.join(__dirname, 'files', safeName);
```

`path.basename()` strips any directory components, so `../../etc/passwd` becomes just `passwd`.
