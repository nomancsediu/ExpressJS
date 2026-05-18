# Environment Configuration

Every app runs in multiple environments: development on your laptop, maybe a staging server, and production. Each one needs different settings. Hardcoding values is a recipe for disaster. Let me set up proper configuration management.

## The .env File Approach

I use `.env` files to store environment-specific values. The `dotenv` package loads them into `process.env`.

```bash
# .env (development, never commit this)
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/ecommerce-dev
JWT_SECRET=dev-secret-key
JWT_REFRESH_SECRET=dev-refresh-secret
STRIPE_SECRET_KEY=sk_test_xxx
```

```bash
# .env.production (for production)
PORT=3000
NODE_ENV=production
MONGODB_URI=mongodb+srv://prod-cluster/ecommerce
JWT_SECRET=a-very-long-random-string
JWT_REFRESH_SECRET=another-very-long-random-string
STRIPE_SECRET_KEY=sk_live_xxx
```

Add `.env` and `.env.production` to `.gitignore`. Never commit secrets to git.

## A Config Module

Instead of accessing `process.env` directly everywhere, I create a config module that validates and centralizes all settings.

```js
// src/config/index.js
const dotenv = require('dotenv');

// Load the right .env file
const env = process.env.NODE_ENV || 'development';
dotenv.config({ path: `.env${env !== 'development' ? '.' + env : ''}` });

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT, 10) || 3000,
  db: {
    uri: process.env.MONGODB_URI || 'mongodb://localhost:27017/ecommerce-dev',
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    refreshSecret: process.env.JWT_REFRESH_SECRET,
    accessExpiry: process.env.JWT_ACCESS_EXPIRY || '15m',
    refreshExpiry: process.env.JWT_REFRESH_EXPIRY || '7d',
  },
  cors: {
    origins: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  },
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
  },
};

// Validate required values
const requiredKeys = ['jwt.secret', 'jwt.refreshSecret'];
requiredKeys.forEach((key) => {
  const value = key.split('.').reduce((obj, k) => obj?.[k], config);
  if (!value) {
    throw new Error(`Missing required config: ${key}`);
  });
});

module.exports = config;
```

Now I use `config.jwt.secret` instead of `process.env.JWT_SECRET`. If a required value is missing, the app fails immediately at startup instead of crashing later at runtime.

## Using the Config

```js
// src/server.js
const config = require('./config');
const { connectDB } = require('./config/db');

connectDB(config.db.uri).then(() => {
  app.listen(config.port, () => {
    console.log(`Server running in ${config.env} on port ${config.port}`);
  });
});
```

```js
// In auth controller
const config = require('../../config');

const signToken = (id) =>
  jwt.sign({ id }, config.jwt.secret, { expiresIn: config.jwt.accessExpiry });
```

## Managing Secrets

For production, avoid putting secrets in files at all. Use your hosting platform's secret manager:

- **Railway/Render**: Set env vars in the dashboard
- **AWS**: Use AWS Secrets Manager or Parameter Store
- **Docker**: Pass env vars at runtime with `--env-file` or `-e`

## Validation at Startup

The config module validates everything when the app starts. If something is wrong, you find out immediately:

```js
// Quick sanity check
if (config.env === 'production' && config.jwt.secret.length < 32) {
  throw new Error('JWT_SECRET must be at least 32 characters in production');
}
```

This pattern saves me from the classic "works on my machine" problem. The config is explicit, validated, and consistent across environments.
