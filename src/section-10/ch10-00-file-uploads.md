# Handling Files in Express

Files are everywhere in web apps. Profile pictures, PDF reports, video uploads, document attachments. If you are building anything real, you will deal with files sooner or later.

When I first started with Express, I thought handling files would be simple. Just grab the file from the request and save it, right? Not quite. The default Express body parsers, `express.json()` and `express.urlencoded()`, do not understand multipart form data. That is the format browsers use when uploading files. So we need extra tools.

This section covers the full journey of file handling in Express:

- **Serving static files** like images, CSS, and JavaScript using `express.static`
- **Accepting file uploads** from users with Multer
- **Validating files** to make sure people upload what you expect
- **Uploading to the cloud** with AWS S3 and Cloudinary
- **Processing images** with Sharp for resizing and format conversion
- **Streaming file downloads** efficiently to users

There is a big difference between saving files to your server disk and sending them to cloud storage. I learned this the hard way when my server ran out of disk space after a few months. Cloud storage scales better and keeps your server clean.

File handling also comes with security risks. A malicious upload can crash your app or worse. We will talk about validating file types, limiting sizes, and filtering out dangerous content.

By the end of this section, you will know how to accept uploads safely, store them locally or in the cloud, process them, and serve them back to users. Let us get started.
