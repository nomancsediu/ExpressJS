# Image Processing with Sharp

Sometimes you need to process images on your server before saving or sending them. Sharp is the fastest and easiest Node.js library for this. It handles resizing, format conversion, cropping, and much more.

## Why Sharp?

I tried other image libraries like Jimp and ImageMagick wrappers. Sharp is in a different league. It is fast, uses minimal memory, and supports all common formats. It can process a 20MB JPEG in milliseconds.

## Installation

```bash
npm install sharp
```

## Resizing Images

```js
const sharp = require('sharp');

// Simple resize
await sharp('input.jpg')
  .resize(800, 600)
  .toFile('output.jpg');

// Resize with options
await sharp('input.jpg')
  .resize(300, 300, {
    fit: 'cover',       // Crop to fill the area
    position: 'center', // Crop from center
    withoutEnlargement: true // Don't upscale small images
  })
  .toFile('thumbnail.jpg');
```

The `fit` option controls how the image fits the target size:
- `cover` crops the image to fill the area
- `contain` fits the entire image within the area (may have padding)
- `fill` stretches to fill (may distort)
- `inside` fits within the area without cropping or stretching
- `outside` fills the area, keeping aspect ratio

## Format Conversion

```js
// Convert JPEG to WebP (better compression)
await sharp('input.jpg')
  .webp({ quality: 80 })
  .toFile('output.webp');

// Convert to PNG (lossless)
await sharp('input.jpg')
  .png()
  .toFile('output.png');

// Convert to AVIF (next-gen format)
await sharp('input.jpg')
  .avif({ quality: 65 })
  .toFile('output.avif');
```

I always convert uploaded images to WebP now. It cuts file sizes by 30-50% with barely visible quality loss.

## Processing Uploads on the Fly

Combine Sharp with Multer for in-memory processing:

```js
const upload = multer({ storage: multer.memoryStorage() });

app.post('/upload', upload.single('photo'), async (req, res) => {
  try {
    const processed = await sharp(req.file.buffer)
      .resize(800, null, { withoutEnlargement: true })
      .webp({ quality: 80 })
      .toBuffer();

    // Save processed buffer to disk or cloud
    fs.writeFileSync('uploads/processed.webp', processed);

    res.send('Image processed and saved');
  } catch (err) {
    res.status(500).send('Processing failed');
  }
});
```

## Generating Thumbnails

A common pattern is creating multiple sizes from one upload:

```js
async function createSizes(inputBuffer) {
  const sizes = {
    thumbnail: await sharp(inputBuffer).resize(150, 150, { fit: 'cover' }).webp().toBuffer(),
    medium: await sharp(inputBuffer).resize(400, null, { withoutEnlargement: true }).webp().toBuffer(),
    large: await sharp(inputBuffer).resize(1200, null, { withoutEnlargement: true }).webp().toBuffer()
  };

  return sizes;
}
```

## Other Useful Operations

```js
// Rotate based on EXIF orientation
await sharp('photo.jpg').rotate().toFile('rotated.jpg');

// Add a blur
await sharp('photo.jpg').blur(5).toFile('blurred.jpg');

// Grayscale
await sharp('photo.jpg').grayscale().toFile('bw.jpg');

// Get image metadata
const metadata = await sharp('photo.jpg').metadata();
console.log(metadata.width, metadata.height, metadata.format);
```

Sharp processes images in a stream, so it uses very little memory even for large files. It is my go-to for any server-side image work.
