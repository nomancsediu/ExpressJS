# API Key Authentication

Not every client is a human with a browser. Sometimes I need to authenticate other servers, scripts, or services. For these machine-to-machine scenarios, API keys are a simple and effective solution.

## What API Keys Are

An API key is a unique string that identifies a client. The client sends the key with each request, and the server looks it up to verify access.

```
Client                           Server
  |                                 |
  |--- GET /api/data               -->|
  |    Header: X-API-Key: abc123     |
  |                                 | Look up key abc123
  |<-- data -------------------------|
```

API keys are simpler than JWT or OAuth. There are no tokens to refresh, no sessions to manage. Just a string that says "I am client X."

## Generating API Keys

I generate keys using Node's crypto module:

```js
const crypto = require('crypto');

const generateApiKey = () => {
  return `sk_${crypto.randomBytes(32).toString('hex')}`;
};

// Example output: sk_a3f8b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0
```

I prefix the key with `sk_` (for "secret key") so I can easily identify it in logs and configuration files. The prefix also helps prevent accidental commits since I can scan for `sk_` patterns.

## Storing API Keys

I store hashed keys in the database, just like passwords:

```js
const apiKeySchema = new mongoose.Schema({
  name: { type: String, required: true },       // "Production Server"
  keyHash: { type: String, required: true },     // Hashed key
  keyPrefix: { type: String, required: true },   // "sk_a3f8..." for identification
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  permissions: [String],                         // ['read:users', 'write:posts']
  lastUsed: Date,
  isActive: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now },
});
```

Notice I store `keyPrefix` (the first 8 characters) so I can show users a hint about which key they created, without revealing the full key.

```js
const createApiKey = async (userId, name) => {
  const rawKey = generateApiKey();
  const keyHash = crypto.createHash('sha256').update(rawKey).digest('hex');
  const keyPrefix = rawKey.substring(0, 8);

  await ApiKey.create({ name, keyHash, keyPrefix, userId });

  // Return the raw key only once
  return rawKey;
};
```

## Validation Middleware

```js
const AppError = require('../utils/AppError');

const apiKeyAuth = async (req, res, next) => {
  try {
    const key = req.headers['x-api-key'];

    if (!key) {
      return next(new AppError('API key required', 401));
    }

    const keyHash = crypto.createHash('sha256').update(key).digest('hex');
    const apiKey = await ApiKey.findOne({ keyHash, isActive: true });

    if (!apiKey) {
      return next(new AppError('Invalid API key', 401));
    }

    // Update last used timestamp
    apiKey.lastUsed = new Date();
    await apiKey.save();

    req.apiKey = apiKey;
    next();
  } catch (err) {
    next(err);
  }
};

module.exports = apiKeyAuth;
```

## When to Use API Keys

I use API keys for:

- **Server-to-server communication**: My backend calling another service
- **Third-party integrations**: External developers accessing my API
- **Webhooks**: Verifying that incoming webhooks are from who they claim
- **CLI tools**: Scripts that interact with my API

I do NOT use API keys for:

- **User-facing applications**: JWT or sessions are better for this
- **High-security operations**: API keys are long-lived and hard to scope precisely
- **Public web apps**: Keys in browser code are easily stolen

## Key Rotation

API keys should be rotated periodically. I let users create a new key, update their services, and then deactivate the old one:

```js
router.post('/api-keys/:id/deactivate', protect, async (req, res) => {
  const apiKey = await ApiKey.findOne({
    _id: req.params.id,
    userId: req.user.id,
  });

  if (!apiKey) {
    return res.status(404).json({ message: 'API key not found' });
  }

  apiKey.isActive = false;
  await apiKey.save();

  res.json({ message: 'API key deactivated' });
});
```

API keys are not fancy, but they solve a real problem. For machine-to-machine auth, simple is good.
