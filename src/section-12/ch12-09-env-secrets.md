# Environment Secrets

Never hardcode secrets in your code. API keys, database passwords, JWT secrets, and other sensitive values must come from the environment. I learned this the hard way when I accidentally pushed my database password to a public GitHub repository.

## The Problem

```js
// NEVER do this
const dbPassword = 'my_super_secret_password';
const apiKey = 'sk-abc123def456';
const jwtSecret = 'keyboard_cat';

app.use(session({ secret: 'keyboard_cat' }));
```

If this code gets pushed to GitHub, anyone can see your secrets. Even in a private repository, it is bad practice. Secrets should never live in source code.

## Using dotenv

The `dotenv` package loads secrets from a `.env` file:

```bash
npm install dotenv
```

Create a `.env` file in your project root:

```
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
JWT_SECRET=a_long_random_string_here
AWS_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

Load it in your app:

```js
require('dotenv').config();

const dbUrl = process.env.DATABASE_URL;
const jwtSecret = process.env.JWT_SECRET;
```

## Protecting the .env File

Add `.env` to your `.gitignore` immediately:

```
# .gitignore
.env
.env.local
.env.production
```

Never commit `.env` files to version control. Instead, create a `.env.example` file with placeholder values:

```
DATABASE_URL=your_database_url_here
JWT_SECRET=your_jwt_secret_here
AWS_ACCESS_KEY=your_aws_access_key_here
```

Commit the example file so other developers know what variables they need.

## Generating Strong Secrets

```bash
# Generate a random secret
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

Use this for JWT secrets, session secrets, and any other tokens. Never use dictionary words or simple strings.

## Different Environments

Use different `.env` files for different environments:

```
.env              # Default
.env.local        # Local overrides (gitignored)
.env.development  # Development defaults
.env.production   # Production values
```

```js
const env = process.env.NODE_ENV || 'development';
require('dotenv').config({ path: `.env.${env}` });
require('dotenv').config({ path: '.env.local' });  // Local overrides
```

## Required Variables Validation

Fail fast if required variables are missing:

```js
function requireEnv(name) {
  const value = process.env[name];
  if (!value) {
    throw new Error(`Missing required environment variable: ${name}`);
  }
  return value;
}

const jwtSecret = requireEnv('JWT_SECRET');
const dbUrl = requireEnv('DATABASE_URL');
```

This is much better than your app failing mysteriously halfway through a request because a variable is undefined.

## Secret Management for Production

For production, consider dedicated secret management:

- **AWS Secrets Manager** for AWS deployments
- **HashiCorp Vault** for self-hosted or multi-cloud
- **Docker secrets** for containerized deployments

These tools handle rotation, access control, and auditing. They are overkill for small projects but essential for production systems with teams.

## Rotating Secrets

Change your secrets regularly. If a secret leaks, rotate it immediately. Design your app to support rotation without downtime:

```js
// Support multiple JWT secrets during rotation
const jwtSecrets = [
  process.env.JWT_SECRET_CURRENT,
  process.env.JWT_SECRET_PREVIOUS
].filter(Boolean);
```

Treat secrets like toothbrushes. Do not share them, replace them regularly, and if they fall into the wrong hands, get new ones immediately.
