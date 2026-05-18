# AWS Deployment

AWS is the big one. More services than anyone can memorize. But for deploying an Express app, I only need a handful of them. Let me break it down.

## The Architecture

Here is what I am building on AWS:

- **EC2** or **Elastic Beanstalk**: Runs the Express app
- **RDS** or **DocumentDB**: Managed database (MongoDB alternative)
- **S3**: Stores uploaded images
- **CloudFront**: CDN for fast delivery of static assets

## Option 1: EC2 (Manual but Full Control)

Launch an EC2 instance and set everything up yourself.

### Launch an Instance

1. Go to the EC2 console and click "Launch Instance"
2. Choose "Ubuntu 22.04" as the AMI
3. Select `t3.small` or `t3.medium` for a Node.js app
4. Create a security group allowing ports 22 (SSH) and 80 (HTTP) and 443 (HTTPS)
5. Create or select an SSH key pair

### Connect and Deploy

```bash
# SSH into the instance
ssh -i your-key.pem ubuntu@your-instance-ip

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Clone your repo
git clone https://github.com/your-username/ecommerce-api.git
cd ecommerce-api
npm ci --only=production

# Set up environment variables
cp .env.production.example .env.production
nano .env.production  # Fill in your values

# Run with PM2
sudo npm install -g pm2
pm2 start src/server.js --name ecommerce
pm2 save
pm2 startup
```

PM2 keeps the app running. If it crashes, PM2 restarts it. The `pm2 startup` command creates a system service so PM2 starts on boot.

## Option 2: Elastic Beanstalk (Managed)

Elastic Beanstalk handles the infrastructure for you. You just upload code.

### Prepare the App

Create a `Procfile` in the project root:

```text
web: node src/server.js
```

### Deploy

1. Go to Elastic Beanstalk console
2. Create a new application
3. Choose "Node.js" platform
4. Upload your code as a ZIP file
5. Set environment variables in the configuration

Elastic Beanstalk handles load balancing, auto-scaling, and health monitoring. It costs more than a bare EC2 instance but saves a lot of operational effort.

## S3 for File Uploads

Instead of storing images on the server, I upload them to S3. This works better with multiple servers and is more reliable.

```js
const AWS = require('aws-sdk');
const s3 = new AWS.S3({
  accessKeyId: process.env.AWS_ACCESS_KEY,
  secretAccessKey: process.env.AWS_SECRET_KEY,
});

exports.uploadToS3 = async (file) => {
  const params = {
    Bucket: process.env.AWS_S3_BUCKET,
    Key: `products/${Date.now()}-${file.originalname}`,
    Body: file.buffer,
    ContentType: file.mimetype,
  };

  const result = await s3.upload(params).promise();
  return result.Location;
};
```

## CloudFront CDN

Put CloudFront in front of S3 for faster image delivery worldwide:

1. Create a CloudFront distribution
2. Set the origin to your S3 bucket
3. Configure a custom domain and SSL certificate
4. Update image URLs in your app to use the CloudFront domain

## RDS for Databases

If you want a managed database on AWS, use RDS for PostgreSQL or DocumentDB for MongoDB compatibility:

1. Launch a database instance in RDS
2. Set the security group to allow access from your EC2 instance
3. Update `MONGODB_URI` or `DATABASE_URL` in your environment variables

AWS gives you the most control and the most responsibility. Pick the services you need and ignore the rest.
