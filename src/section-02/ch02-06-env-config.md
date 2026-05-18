# Environment and Configuration

Hardcoding values in your code is one of those things that works fine until it does not. The day you accidentally push your database password to a public GitHub repository is the day you understand why environment variables matter. Let me show you how I handle configuration in Express projects.

## The Problem

Imagine this scenario. Your app connects to a database. In development, you use a local database. In production, you use a hosted one. If the database URL is hardcoded, you have to change the code every time you deploy. That is error-prone and dangerous.

Environment variables solve this by letting you configure your app from the outside. The same code works in every environment. Only the variables change.

## Using dotenv

The `dotenv` package loads variables from a `.env` file into `process.env`. This is the most common way to manage environment variables in Node.js.

Install it:

```bash
npm install dotenv
```

Create a `.env` file:

```env
PORT=3000
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
JWT_SECRET=my-super-secret-key
JWT_EXPIRES_IN=7d
CORS_ORIGIN=http://localhost:3001
LOG_LEVEL=debug
```

Load it early in your application:

```javascript
// server.js
const dotenv = require('dotenv');

// Load environment variables before anything else
dotenv.config();

const app = require('./app');
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

The `dotenv.config()` call reads the `.env` file and adds each variable to `process.env`. After that, you access them with `process.env.VARIABLE_NAME`.

## Never Commit .env to Git

I cannot stress this enough. Your `.env` file contains secrets. Database passwords, API keys, JWT secrets. If these end up in git, anyone with access to the repository can see them.

Add `.env` to your `.gitignore`:

```gitignore
.env
.env.local
.env.*.local
```

Instead, commit a `.env.example` file that shows the required variables without the actual values:

```env
PORT=
NODE_ENV=
DATABASE_URL=
JWT_SECRET=
JWT_EXPIRES_IN=
CORS_ORIGIN=
LOG_LEVEL=
```

Other developers copy `.env.example` to `.env` and fill in their own values.

## Validating Environment Variables

One of the worst things that can happen is your app starts with a missing environment variable and crashes in a confusing way later. I validate all required variables at startup.

```javascript
// src/config/index.js
const requiredVars = [
  'DATABASE_URL',
  'JWT_SECRET',
  'NODE_ENV'
];

const missingVars = requiredVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  throw new Error(
    `Missing required environment variables: ${missingVars.join(', ')}. ` +
    `Check your .env file.`
  );
}

module.exports = {
  port: parseInt(process.env.PORT, 10) || 3000,
  env: process.env.NODE_ENV || 'development',
  isDev: process.env.NODE_ENV === 'development',
  isProd: process.env.NODE_ENV === 'production',
  isTest: process.env.NODE_ENV === 'test',
  database: {
    url: process.env.DATABASE_URL,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
  },
  cors: {
    origin: process.env.CORS_ORIGIN || '*',
  },
  log: {
    level: process.env.LOG_LEVEL || 'info',
  }
};
```

Using a config module like this has several benefits:

- All configuration lives in one place. You do not have `process.env.SOMETHING` scattered across your codebase.
- Values are parsed and typed once. Notice `parseInt` for the port number.
- Helper flags like `isDev` and `isProd` make conditional logic cleaner.
- Validation happens at startup. Your app fails fast with a clear error message.

## Using the Config Module

```javascript
// app.js
const config = require('./src/config');

app.listen(config.port, () => {
  console.log(`Server running in ${config.env} mode on port ${config.port}`);
});

// In a route
if (config.isDev) {
  console.log('Debug info:', debugData);
}
```

## Different Environments

You might need different settings for development, testing, and production. Here is how I handle it:

```javascript
// src/config/index.js
const env = process.env.NODE_ENV || 'development';

const configs = {
  development: {
    port: 3000,
    database: { url: process.env.DATABASE_URL },
    jwt: { expiresIn: '7d' },
    log: { level: 'debug' },
  },
  test: {
    port: 3001,
    database: { url: process.env.DATABASE_URL_TEST },
    jwt: { expiresIn: '1h' },
    log: { level: 'error' },
  },
  production: {
    port: process.env.PORT || 8080,
    database: { url: process.env.DATABASE_URL },
    jwt: { expiresIn: '1d' },
    log: { level: 'warn' },
  }
};

module.exports = { ...configs[env], env };
```

This way, you get sensible defaults for each environment, and you can still override them with environment variables.

## Security Tips

Here are the configuration security practices I follow:

- **Never commit secrets to git.** I said it before, I will say it again.
- **Use strong secrets in production.** A JWT secret should be a long, random string. Not "secret123".
- **Rotate secrets regularly.** If a secret leaks, change it immediately.
- **Limit who has access to production secrets.** Use a secrets manager like AWS Secrets Manager or HashiCorp Vault for production.
- **Do not log secrets.** Be careful with logging the full config object.
- **Use different secrets per environment.** Your development JWT secret should not work in production.

Configuration is one of those topics that seems boring until something goes wrong. Take the time to set it up right, and you will thank yourself later.
