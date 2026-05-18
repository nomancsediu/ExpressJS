# File Validation

Never trust user uploads. I learned this the hard way when someone uploaded a PHP script disguised as a JPEG. File validation is not optional, it is essential.

## MIME Type Checking

The `fileFilter` option in Multer lets you reject files before they are saved:

```js
const upload = multer({
  dest: 'uploads/',
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];

    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error(`Invalid file type: ${file.mimetype}. Only images allowed.`), false);
    }
  }
});
```

A word of caution: the MIME type comes from the browser, which bases it on the file extension. A malicious user could rename `.php` to `.jpg` and the browser sends `image/jpeg`. For critical validation, check the actual file contents too.

## Size Limits

Multer's `limits` option prevents oversized uploads:

```js
const upload = multer({
  limits: {
    fileSize: 2 * 1024 * 1024,   // 2MB per file
    files: 5,                      // Max 5 files
    fields: 10,                    // Max 10 text fields
    fieldSize: 1024 * 1024,       // Max 1MB per text field
    parts: 15                      // Max total parts (fields + files)
  }
});
```

When a limit is exceeded, Multer throws a `MulterError`. You can catch it in your error handler.

## Custom Validation Function

For more complex validation, create a reusable function:

```js
function createFileFilter(allowedTypes) {
  return (req, file, cb) => {
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error(`Type ${file.mimetype} not allowed`), false);
    }
  };
}

const imageUpload = multer({
  dest: 'uploads/',
  fileFilter: createFileFilter(['image/jpeg', 'image/png']),
  limits: { fileSize: 5 * 1024 * 1024 }
});

const docUpload = multer({
  dest: 'uploads/',
  fileFilter: createFileFilter(['application/pdf']),
  limits: { fileSize: 10 * 1024 * 1024 }
});
```

## Error Handling

Always handle Multer errors separately from other errors:

```js
app.post('/upload', imageUpload.single('photo'), (req, res) => {
  res.send('Upload successful');
});

app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).send('File is too large');
    }
    if (err.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).send('Too many files');
    }
    return res.status(400).send(err.message);
  }

  if (err.message.includes('not allowed')) {
    return res.status(400).send(err.message);
  }

  res.status(500).send('Something went wrong');
});
```

## Validating File Extensions

As an extra layer, check the file extension too:

```js
const allowedExts = ['.jpg', '.jpeg', '.png', '.webp'];
const ext = path.extname(file.originalname).toLowerCase();

if (!allowedExts.includes(ext)) {
  return cb(new Error('Extension not allowed'), false);
}
```

Combine extension checks with MIME type checks for defense in depth. Neither method alone is foolproof, but together they catch most problems.
