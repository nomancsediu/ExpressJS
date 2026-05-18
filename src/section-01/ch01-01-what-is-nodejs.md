# What is Node.js

Before I can explain Express, I need to talk about what Node.js actually is. I used to think Node.js was a framework, similar to Django or Rails. It is not. Node.js is a runtime. It lets you run JavaScript outside of a browser.

## The Short Version

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Ryan Dahl created it in 2009 because he was frustrated with how web servers handled concurrent connections. He wanted something that could handle thousands of connections without creating thousands of threads.

## Why Node.js Was Created

Back in 2009, the common way to handle web requests was to use a thread per connection. Apache, for example, would spin up a new thread for each incoming request. That works fine for a few dozen users. It falls apart when you have thousands of users connecting at the same time.

Ryan Dahl saw this problem and had a different idea. What if a single thread could handle many connections by waiting efficiently? Instead of blocking on each request, the server could start handling a request, pause when it needs to wait for something like a database query, and then resume when the data comes back.

That idea became Node.js.

## What You Can Build With Node.js

Here are some things I have built or seen built with Node.js:

- **REST APIs and web servers.** This is what Express is for, and it is the most common use case.
- **Real-time applications.** Chat apps, collaborative editors, live dashboards. Node.js excels at these because of its event-driven nature.
- **CLI tools.** Many popular command line tools are built with Node.js, like npm itself.
- **Desktop applications.** Electron uses Node.js under the hood. VS Code is built with it.
- **Scripts and automation.** Any task you would write a bash or Python script for, you can write in Node.js.
- **Microservices.** Node.js is lightweight and starts fast, making it a good fit for microservice architectures.

## What Node.js Is NOT

This is important, so let me be clear:

- Node.js is **not a framework**. It does not give you routing, middleware, or templates out of the box.
- Node.js is **not a language**. It runs JavaScript. The language is JavaScript.
- Node.js is **not a web server** by itself. It provides the building blocks to create one, but you have to assemble them.
- Node.js is **not the right tool for everything**. CPU-intensive tasks like video encoding or heavy math are not its strength.

## A Quick Example

Here is what it looks like to create a simple HTTP server with just Node.js, no Express:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from Node.js!');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

This works. But imagine adding routing, query string parsing, POST body handling, cookies, sessions, and error pages. That is where Express comes in. Express makes all of that easier by providing a clean API on top of Node.js.

Understanding that Express is built on Node.js is the first step to mastering both. When something goes wrong in your Express app, knowing how Node.js works underneath will help you figure out why.
