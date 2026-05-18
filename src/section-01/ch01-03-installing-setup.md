# Installing and Setting Up Node.js

Getting Node.js installed seems simple enough, but there are choices to make. I have installed Node.js probably fifty times at this point, and I have learned a few things along the way that will save you headaches.

## Option 1: NVM (Recommended)

NVM stands for Node Version Manager. It lets you install and switch between multiple versions of Node.js on the same machine. I strongly recommend this approach because different projects often need different Node.js versions.

**On Mac or Linux:**

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Restart your terminal, then install the latest LTS version of Node.js
nvm install --lts

# Use it
nvm use --lts

# Check the version
node --version
```

**On Windows:**

Use nvm-windows instead. Download it from the GitHub releases page at `https://github.com/coreybutler/nvm-windows`.

```bash
nvm install lts
nvm use lts
node --version
```

Why NVM? Because someday you will clone a project that requires Node.js 18 while your current project uses Node.js 20. With NVM, switching is one command. Without it, you are in for a frustrating time.

## Option 2: Direct Install

You can download Node.js directly from the official website at `https://nodejs.org`. Always choose the LTS version for production projects. The "Current" version has the latest features but may have bugs.

On Mac, you can also use Homebrew:

```bash
brew install node
```

This works fine, but you are stuck with one version unless you use a version manager.

## Your First Script

Once Node.js is installed, let us make sure it works. Create a file called `hello.js`:

```javascript
// hello.js
const greeting = 'Hello, Node.js!';
console.log(greeting);

const add = (a, b) => a + b;
console.log(`2 + 3 = ${add(2, 3)}`);
```

Run it from your terminal:

```bash
node hello.js
```

You should see:

```
Hello, Node.js!
2 + 3 = 5
```

## The REPL

Node.js comes with an interactive shell called REPL (Read-Eval-Print Loop). It is great for quick experiments. Just type `node` in your terminal:

```bash
node
```

Now you can type JavaScript directly:

```
> const name = 'Express Learner'
undefined
> name
'Express Learner'
> 10 * 5
50
> .exit
```

Type `.exit` or press Ctrl+C twice to leave the REPL.

## Setting Up package.json

Every Node.js project needs a `package.json` file. It tracks your dependencies, scripts, and project metadata. You can create one quickly:

```bash
mkdir my-express-project
cd my-express-project
npm init -y
```

The `-y` flag uses defaults. Here is what you get:

```json
{
  "name": "my-express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

I always customize a few fields right away:

```json
{
  "name": "my-express-project",
  "version": "1.0.0",
  "description": "My first Express project",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "node --watch app.js"
  }
}
```

The `--watch` flag restarts the server when files change, which saves you from manually restarting during development.

## Check Your Setup

Run these commands to verify everything is working:

```bash
node --version    # Should print v18.x or v20.x or higher
npm --version     # Should print 9.x or 10.x or higher
nvm --version     # If using NVM, should print 0.39.x or higher
```

If all three work, you are ready to move on to the next chapter where we explore the core modules that Node.js provides out of the box.
