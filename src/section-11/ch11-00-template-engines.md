# Server-Side Rendering with Express

Not every app needs a React or Vue frontend. Sometimes the simplest and fastest approach is to render HTML on the server and send it to the browser. That is what server-side rendering (SSR) is about.

When I started learning Express, I jumped straight into building APIs and assumed I would always pair them with a separate frontend. But there are many cases where server-side rendering just makes more sense. Blogs, documentation sites, admin dashboards, small internal tools. These do not need a heavy client-side framework.

Express supports template engines out of the box. You pick an engine, set it up, and Express handles the rest. You write templates with placeholders, pass data from your routes, and the engine generates HTML.

This section covers the most popular template engines for Express:

- **EJS** for developers who want templates that look like HTML
- **Pug** for developers who prefer minimal syntax with no angle brackets
- **Handlebars** for developers who want logic-less templates

We will also look at layouts and partials for keeping your templates DRY, passing data from routes to views, and view caching for better performance.

Each engine has its own style and philosophy. There is no "best" one. Pick whichever feels natural to you. I personally like EJS because it is closest to plain HTML and easy to learn. But Pug fans swear by its concise syntax, and Handlebars users love the separation of logic from presentation.

By the end of this section, you will be able to render dynamic HTML pages in Express with any of these engines. Let us start with the basics.
