# Cloud Upload with AWS S3

Saving files to your server disk works for small projects. But eventually you will want cloud storage. AWS S3 is the most popular choice for storing files in the cloud.

## Why S3?

Your server disk fills up. You lose files if the server crashes. Scaling becomes hard when files live on one machine. S3 solves all of this. Files live in the cloud, accessible from anywhere, and they are durable.

## Setup

Install the packages:

```bash
npm install multer multer-s3 @aws-sdk/client-s3
```

## Uploading to S3

```js
const express = require('express');
const multer = require('multer');
const multerS3 = require('multer-s3');
const { S3Client } = require('@aws-sdk/client-s3');

const s3 = new S3Client({
  region: 'us-east-1',
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY
  }
});

const upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: 'my-app-uploads',
    key: (req, file, cb) => {
      const uniqueName = Date.now() + '-' + file.originalname;
      cb(null, `uploads/${uniqueName}`);
    },
    contentType: multerS3.AUTO_CONTENT_TYPE
  })
});

app.post('/upload', upload.single('photo'), (req, res) => {
  console.log(req.file.location); // S3 URL
  res.json({ url: req.file.location });
});
```

The `key` function determines the file path inside the S3 bucket. I like organizing files in folders by date or type.

## Presigned URLs

Sometimes you want users to upload directly to S3 without going through your server. Presigned URLs make this possible:

```js
const { GetObjectCommand, PutObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

// Generate an upload URL
app.get('/upload-url', async (req, res) => {
  const command = new PutObjectCommand({
    Bucket: 'my-app-uploads',
    Key: `uploads/${Date.now()}.jpg`,
    ContentType: 'image/jpeg'
  });

  const url = await getSignedUrl(s3, command, { expiresIn: 300 });
  res.json({ uploadUrl: url });
});
```

The client can PUT directly to this URL. It expires after 5 minutes. This saves your server from handling the file data at all.

## Getting Presigned Download URLs

```js
app.get('/download/:key', async (req, res) => {
  const command = new GetObjectCommand({
    Bucket: 'my-app-uploads',
    Key: req.params.key
  });

  const url = await getSignedUrl(s3, command, { expiresIn: 60 });
  res.redirect(url);
});
```

This keeps your S3 bucket private while still letting authorized users download files. Never make your bucket public if you can avoid it.

Always store AWS credentials in environment variables. Never commit them to your repository. I made that mistake once and got a surprising AWS bill the next day.
