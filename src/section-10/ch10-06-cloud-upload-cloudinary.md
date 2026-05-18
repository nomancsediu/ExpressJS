# Cloud Upload with Cloudinary

AWS S3 is great for raw file storage. But if you are working with images, Cloudinary is even better. It handles uploads, transformations, optimization, and delivery all in one service.

## Why Cloudinary?

Cloudinary does things S3 cannot do easily. It resizes images on the fly. It converts formats automatically. It serves images through a CDN. It generates thumbnails, crops faces, and applies filters. If your app deals with images, Cloudinary saves you a ton of work.

## Setup

```bash
npm install multer multer-storage-cloudinary cloudinary
```

## Configuring Cloudinary

```js
const cloudinary = require('cloudinary').v2;
const { CloudinaryStorage } = require('multer-storage-cloudinary');

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET
});
```

## Uploading with Multer

```js
const storage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: 'my-app',
    allowed_formats: ['jpg', 'png', 'webp'],
    transformation: [{ width: 1000, crop: 'limit' }],  // Max 1000px wide
    public_id: (req, file) => {
      return Date.now() + '-' + Math.round(Math.random() * 1E9);
    }
  }
});

const upload = multer({ storage });

app.post('/upload', upload.single('photo'), (req, res) => {
  console.log(req.file.path);       // Cloudinary URL
  console.log(req.file.filename);   // Public ID
  res.json({ url: req.file.path });
});
```

The `transformation` option is powerful. In this example, any uploaded image wider than 1000 pixels gets scaled down automatically.

## On-the-Fly Transformations

You do not need to create separate images for every size. Cloudinary generates them from the URL:

```js
// Get a 300px wide thumbnail
const thumbnail = cloudinary.url('my-app/sample-image', {
  width: 300,
  height: 300,
  crop: 'fill',
  gravity: 'face'    // Center on the face
});

// Convert to WebP for better compression
const webpUrl = cloudinary.url('my-app/sample-image', {
  width: 500,
  crop: 'limit',
  fetch_format: 'auto',
  quality: 'auto'
});
```

## Common Transformations

```js
// Round corners
cloudinary.url('my-app/img', { radius: 'max' });

// Add text overlay
cloudinary.url('my-app/img', {
  overlay: { text: 'Hello World', font_family: 'Arial', font_size: 40 }
});

// Blur effect
cloudinary.url('my-app/img', { effect: 'blur:300' });
```

## Deleting Files

```js
app.delete('/image/:publicId', async (req, res) => {
  try {
    const result = await cloudinary.uploader.destroy(req.params.publicId);
    res.json({ result });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

I switched from S3 to Cloudinary for an image-heavy project and it cut my codebase in half. No more separate image processing pipeline. Cloudinary handles it all.
