# Multer Middleware

Express cannot parse file uploads on its own. The body parsers only handle JSON and URL-encoded data. For multipart form data, which is what file uploads use, you need Multer.

Multer is the go-to middleware for file uploads in Express. It adds a `body` object and a `file` or `files` object to the request.

## Installation

```bash
npm install multer
```

## Basic Setup

```js
const express = require('express');
const multer = require('multer');
const app = express();

// Default: save to os.tmpdir() with random filenames
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('avatar'), (req, res) => {
  console.log(req.file);
  res.send('File uploaded!');
});
```

The `dest` option tells Multer where to save files. If you skip it, Multer stores files in memory only, which disappears when the request ends.

## Disk Storage

For more control over filenames and destinations, use `multer.diskStorage`:

```js
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({ storage });
```

This lets you keep the original file extension and add a unique suffix to prevent name collisions. I always use this approach because random filenames without extensions are annoying to work with.

## Memory Storage

Sometimes you do not want to save files to disk at all. Like when you are forwarding them directly to cloud storage:

```js
const upload = multer({ storage: multer.memoryStorage() });
```

With memory storage, the file is stored in `req.file.buffer` as a Buffer. This is faster for small files since there is no disk I/O. But be careful with large files because they eat up RAM.

## Configuration Options

```js
const upload = multer({
  dest: 'uploads/',
  limits: {
    fileSize: 5 * 1024 * 1024,  // 5MB max
    files: 5,                    // Max 5 files per request
    fields: 10                   // Max 10 non-file fields
  },
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Only images allowed'), false);
    }
  }
});
```

The `limits` option helps protect your server from abuse. The `fileFilter` option lets you reject files before they are even saved. We will dive deeper into validation in a later chapter.
