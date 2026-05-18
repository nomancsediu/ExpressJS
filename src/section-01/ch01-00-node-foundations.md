# Why Node.js Foundations Matter Before Express

When I first heard about Express, I wanted to jump straight in and build things. I thought, "Why waste time on Node.js basics when Express is the fun part?" That was a mistake. Let me save you from the same trap.

Express sits on top of Node.js. It does not replace Node.js. It does not hide Node.js. It wraps Node.js in a friendlier API. If you do not understand what is happening underneath, you will eventually hit a wall where nothing makes sense.

## What Happens When You Skip the Basics

I skipped the basics at first, and here is what happened to me:

- I could not debug slow endpoints because I did not understand the event loop.
- I wrote code that crashed under load because I blocked the single thread.
- I could not read error stack traces because I did not know how Node.js modules work.
- I was confused by asynchronous code patterns that Express uses everywhere.

Every single one of those problems came back to the same root cause: I did not understand Node.js.

## What You Will Learn in This Section

This section covers the Node.js foundations that actually matter for building Express applications. I am not going to teach you everything about Node.js. That would take an entire book on its own. Instead, I am focusing on the concepts that come up again and again when you work with Express.

Here is what we will go through:

- **What Node.js is** and why it exists. Understanding its purpose helps you use it correctly.
- **The architecture and event loop.** This is probably the most important topic. Express is non-blocking by design, and you need to know why that matters.
- **Installing and setting up Node.js.** Getting your environment right from the start saves hours of frustration later.
- **Core modules.** Express relies on many of these under the hood, especially `http`, `fs`, and `path`.
- **npm and package management.** Your entire project depends on npm. Knowing how it works is not optional.
- **CommonJS vs ESM.** Express projects use module systems, and you will encounter both patterns.
- **Async JavaScript.** Almost everything in Express is asynchronous. Callbacks, Promises, and async/await are daily tools.
- **The file system module.** Serving static files, reading configs, and writing logs all use `fs`.
- **The HTTP module.** Express is built on top of Node's `http` module. Seeing the raw version helps you appreciate Express.
- **Streams and buffers.** File uploads, large responses, and data processing all depend on streams.

## How to Use This Section

Read through it in order the first time. Each chapter builds on the previous one. After that, you can come back to individual chapters as reference material. I still revisit the event loop chapter myself when debugging tricky performance issues.

Let us get started.
