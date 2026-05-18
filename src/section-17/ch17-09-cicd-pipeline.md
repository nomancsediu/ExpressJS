# CI/CD Pipeline

Manually deploying code gets old fast. SSH into the server, pull the latest code, restart the app. Every single time. CI/CD automates this so you can focus on writing code instead of deploying it.

## What CI/CD Means

- **CI (Continuous Integration)**: Automatically test your code when you push changes
- **CD (Continuous Deployment)**: Automatically deploy your code after tests pass

The flow is: push code, tests run, code deploys. No manual steps.

## GitHub Actions

GitHub Actions is built into GitHub. No extra services needed. I define workflows in YAML files inside the repo.

### Basic Test Workflow

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:7
        ports:
          - 27017:27017

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test
        env:
          NODE_ENV: test
          MONGODB_URI: mongodb://localhost:27017/ecommerce-test
          JWT_SECRET: test-secret
          JWT_REFRESH_SECRET: test-refresh-secret
```

This workflow runs on every push to `main` or `develop` and on every pull request. It starts a MongoDB service container, installs dependencies, lints the code, and runs tests.

### Deploy Workflow

I add a separate workflow for deploying to production. It only runs on pushes to `main` after tests pass.

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test
        env:
          NODE_ENV: test
          MONGODB_URI: mongodb://localhost:27017/ecommerce-test
          JWT_SECRET: test-secret
          JWT_REFRESH_SECRET: test-refresh-secret

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Railway
        uses: railway-app/railway-deploy@v1
        with:
          railway-token: ${{ secrets.RAILWAY_TOKEN }}
```

The `needs: test` line means the deploy job only runs if the test job succeeds. If tests fail, nothing deploys.

### Storing Secrets

Never put API keys or tokens in workflow files. Use GitHub Secrets:

1. Go to your repo settings
2. Click "Secrets and variables" then "Actions"
3. Add each secret (like `RAILWAY_TOKEN`, `AWS_ACCESS_KEY`)

Reference them in workflows with `${{ secrets.SECRET_NAME }}`.

## Deploying to EC2 with SSH

For servers I manage myself, I use SSH in the workflow:

```yaml
- name: Deploy to EC2
  uses: appleboy/ssh-action@v1
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ubuntu
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /home/ubuntu/ecommerce-api
      git pull origin main
      npm ci --only=production
      pm2 restart ecommerce
```

This SSHs into the server, pulls the latest code, installs dependencies, and restarts PM2. Simple and effective.

## Branch Protection

In GitHub repo settings, I enable branch protection for `main`:

- Require pull request reviews before merging
- Require status checks to pass (the test workflow)
- Require branches to be up to date before merging

This ensures no untested code reaches production. Ever.

CI/CD is not just about convenience. It is about safety. If something breaks, you find out before it reaches users, not after.
