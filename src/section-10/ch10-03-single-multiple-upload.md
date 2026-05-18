# Single and Multiple File Uploads

Multer gives you different methods depending on how many files you want to accept. Let me walk through each one.

## Single File Upload

The simplest case. One file, one field name:

```js
const upload = multer({ dest: 'uploads/' });

app.post('/profile', upload.single('avatar'), (req, res) => {
  // req.file contains the file info
  console.log(req.file.fieldname);    // 'avatar'
  console.log(req.file.originalname); // Original file name
  console.log(req.file.size);         // Size in bytes
  console.log(req.file.mimetype);     // MIME type like 'image/png'

  // req.body contains text fields
  console.log(req.body.username);
  res.send('Upload complete');
});
```

The uploaded file is available at `req.file` (singular).

## Multiple Files, Same Field

When users can select multiple files from one input:

```js
app.post('/photos', upload.array('photos', 5), (req, res) => {
  // req.files is an array of file objects
  console.log(req.files.length); // Number of uploaded files

  req.files.forEach(file => {
    console.log(file.originalname);
  });

  res.send(`${req.files.length} files uploaded`);
});
```

The second argument `5` is the maximum count. If someone sends 6 files, Multer throws an error.

## Multiple Files, Different Fields

When your form has multiple file inputs with different names:

```js
const cpUpload = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]);

app.post('/profile', cpUpload, (req, res) => {
  // req.files is an object keyed by field name
  console.log(req.files['avatar'][0]);    // The avatar file
  console.log(req.files['gallery']);       // Array of gallery files

  res.send('Upload complete');
});
```

Notice that even for `avatar` with `maxCount: 1`, the result is still an array. You access it with `req.files['avatar'][0]`.

## No File Upload

If you just need to parse multipart form data without files:

```js
app.post('/text-only', upload.none(), (req, res) => {
  console.log(req.body); // Only text fields
  res.send('No files, just text');
});
```

## HTML Form Setup

Your HTML form must use `multipart/form-data`:

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="text" name="username" />
  <input type="file" name="avatar" />
  <button type="submit">Upload</button>
</form>
```

For multiple files, add the `multiple` attribute:

```html
<input type="file" name="photos" multiple />
```

I used to forget the `enctype` attribute all the time. Without it, the browser sends URL-encoded data and Multer sees nothing. Always double-check your form encoding.

## Quick Reference

| Method | Request Property | Use Case |
|--------|-----------------|----------|
| `single()` | `req.file` | One file |
| `array()` | `req.files` | Multiple files, same field |
| `fields()` | `req.files` (object) | Multiple files, different fields |
| `none()` | `req.body` only | Multipart text without files |
