# Installing and Setting Up Express

Setting up a project properly from the start saves you hours of headaches later. I learned this the hard way by jumping in without any structure and then having to reorganize everything. Let me show you the setup process I use now.

## Creating the Project

```bash
# Create a project directory
mkdir my-express-app
cd my-express-app

# Initialize package.json
npm init -y

# Install Express
npm install express

# Install development dependencies
npm install --save-dev nodemon
```

That gives you the basics. Now let me customize the `package.json`:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "My Express learning project",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

I always set up the `dev` script with nodemon. It watches your files and restarts the server automatically when you save changes. Without it, you have to stop and start the server manually after every edit, which gets old fast.

## Creating a .gitignore

If you are using git, and you should be, create a `.gitignore` file right away:

```gitignore
# Dependencies
node_modules/

# Environment variables
.env
.env.local
.env.*.local

# Logs
logs/
*.log

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Build output
dist/
build/
```

The most critical entry is `node_modules/`. You never commit this folder. It can contain tens of thousands of files and is recreated with `npm install`. The `.env` entry is equally important because environment files often contain secrets like database passwords and API keys.

## Setting Up Environment Variables

Environment variables let you configure your app without hardcoding values. This is how you handle different settings for development, testing, and production.

Install the `dotenv` package:

```bash
npm install dotenv
```

Create a `.env` file in your project root:

```env
PORT=3000
NODE_ENV=development
APP_NAME=MyExpressApp
```

Create a `.env.example` file that you DO commit to git. This tells other developers what variables they need:

```env
PORT=
NODE_ENV=
APP_NAME=
```

The actual `.env` file stays out of git. The `.env.example` file goes into git as documentation.

## Basic Project Structure

Here is the folder structure I start with:

```
my-express-app/
  app.js              # App setup and configuration
  server.js           # Server startup (separated from app for testing)
  .env                # Environment variables (not in git)
  .env.example        # Template for environment variables
  .gitignore          # Git ignore rules
  package.json        # Project metadata and dependencies
  package-lock.json   # Locked dependency versions
  src/
    routes/           # Route definitions
    middleware/       # Custom middleware
    controllers/      # Route handler logic
    models/           # Data models
    config/           # Configuration files
    utils/            # Utility functions
  public/             # Static files (CSS, JS, images)
```

You do not need all of these folders on day one. Create them as you need them. But having the structure planned out helps you stay organized as the project grows.

## The Minimal Starting Files

Let me create the absolute minimum files to get a running Express app.

**server.js:**

```javascript
const app = require('./app');
const dotenv = require('dotenv');

dotenv.config();

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

**app.js:**

```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Server is running!' });
});

module.exports = app;
```

I separate `app.js` and `server.js` for a reason. By exporting `app` from `app.js`, I can import it in test files without starting the server. This makes testing much easier.

## Running the Project

```bash
# Development mode (auto-restart on changes)
npm run dev

# Production mode
npm start
```

Open your browser or use curl to test:

```bash
curl http://localhost:3000
```

You should see:

```json
{ "message": "Server is running!" }
```

## Additional Setup Packages

As the project grows, I usually add these packages early on:

```bash
# Security
npm install helmet cors

# Logging
npm install morgan

# Cookie and session support
npm install cookie-parser

# Utility
npm install http-errors
```

Do not install everything at once. Add packages when you actually need them. Every dependency is something you have to maintain and update.

Setting up a project correctly from the start is like laying a good foundation for a house. It is not glamorous work, but it makes everything you build on top of it more stable and easier to maintain.
