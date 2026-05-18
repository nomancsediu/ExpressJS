# npm and Package Management

npm is the package manager that comes with Node.js. It is how you install libraries, manage dependencies, and run scripts. If you are building an Express app, you will use npm every single day. Let me walk you through what you need to know.

## Basic npm Commands

Here are the commands I use most often:

```bash
# Initialize a new project
npm init -y

# Install a package (adds to node_modules and package.json)
npm install express

# Install a specific version
npm install express@4.18.2

# Install as a dev dependency (only needed during development)
npm install --save-dev nodemon

# Install globally (available anywhere on your system)
npm install -g npm-check-updates

# Uninstall a package
npm uninstall express

# Update all packages to their latest allowed versions
npm update

# List installed packages
npm list

# List only top-level packages
npm list --depth=0

# View info about a package
npm view express
```

## Understanding package.json

The `package.json` file is the heart of your project. Here is a realistic example from an Express project:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "A learning project for Express",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "lint": "eslint ."
  },
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "eslint": "^8.50.0"
  }
}
```

The key sections:

- **dependencies** are packages your app needs to run in production.
- **devDependencies** are packages only needed during development, like testing tools and linters.
- **scripts** are shortcuts for common commands.

## Semantic Versioning (semver)

Version numbers in npm follow semver: `MAJOR.MINOR.PATCH`. For example, `4.18.2` means major version 4, minor version 18, patch version 2.

- **PATCH** (4.18.2 to 4.18.3): Bug fixes. Should not break anything.
- **MINOR** (4.18.2 to 4.19.0): New features. Backward compatible.
- **MAJOR** (4.18.2 to 5.0.0): Breaking changes. Not backward compatible.

The symbols in front of versions matter:

```
^4.18.2   Allows MINOR and PATCH updates (4.18.2 up to 4.x.x, but not 5.0.0)
~4.18.2   Allows PATCH updates only (4.18.2 up to 4.18.x, but not 4.19.0)
4.18.2    Exact version only
*         Any version (dangerous, never use this)
```

When you run `npm install express`, npm adds `^4.18.2` by default. This means you get patch and minor updates automatically when you run `npm update`, but never a major version that could break your code.

## package-lock.json

When you install packages, npm creates a `package-lock.json` file. This file locks down the exact versions of every package, including sub-dependencies. Always commit this file to git. It ensures that every developer on your team and every deployment environment gets the same versions.

## npm Scripts

Scripts in `package.json` are incredibly useful. You run them with `npm run <name>` or `npm <name>` for built-in scripts like `start` and `test`.

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "dev:debug": "DEBUG=app:* nodemon app.js",
    "lint": "eslint src/",
    "clean": "rm -rf node_modules && npm install"
  }
}
```

```bash
npm start          # Runs: node app.js
npm run dev        # Runs: nodemon app.js
npm run dev:debug  # Runs: DEBUG=app:* nodemon app.js
npm run lint       # Runs: eslint src/
```

## npx

npx comes with npm and lets you run packages without installing them globally. This is handy for one-off commands:

```bash
# Create a new Express app using the generator
npx express-generator my-app

# Run a local package binary without installing globally
npx eslint --init

# Check package versions
npx npm-check-updates
```

## node_modules

The `node_modules` folder is where npm puts all installed packages. Never edit files in here manually, and never commit it to git. Always add it to your `.gitignore`:

```
node_modules/
```

When someone clones your project, they run `npm install` to recreate `node_modules` from `package.json` and `package-lock.json`.

Understanding npm is not glamorous, but it is foundational. Every Express project starts with `npm init` and `npm install express`, and everything builds from there.
